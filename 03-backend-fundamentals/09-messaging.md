# Messaging with MassTransit

> **Concept:** See [Message Reliability](https://github.com/TP-Coder-Innovation-Hub/learning-content/blob/main/distributed-systems/06-message-reliability.md) for outbox pattern, idempotent consumers, and DLQ concepts. This page covers the .NET implementation with MassTransit and RabbitMQ.

## Why MassTransit

MassTransit is the de facto standard for .NET messaging. It abstracts RabbitMQ, Azure Service Bus, and Amazon SQS behind a consistent API. You write consumers once, swap transports later.

## Setup

```bash
dotnet add package MassTransit.RabbitMQ
```

```csharp
builder.Services.AddMassTransit(opt =>
{
    opt.AddConsumer<OrderPlacedConsumer>();

    opt.UsingRabbitMq((context, cfg) =>
    {
        cfg.Host(builder.Configuration.GetConnectionString("RabbitMQ"));

        // Automatic retry: 3 attempts with exponential backoff
        cfg.UseMessageRetry(r => r.Intervals(1000, 5000, 15000));

        // Dead-letter queue after max retries
        cfg.UseDelayedRedelivery(r => r.Intervals(3600, 21600, 43200));

        cfg.ConfigureEndpoints(context);
    });
});
```

## Define Events

Events are simple C# records. They represent something that happened:

```csharp
namespace MyApp.Contracts;

public record OrderPlaced(
    Guid OrderId,
    Guid CustomerId,
    decimal Total,
    DateTime PlacedAt);

public record OrderShipped(
    Guid OrderId,
    string TrackingNumber,
    DateTime ShippedAt);
```

**Convention:** Put contracts in a separate shared project (`MyApp.Contracts`). Both producers and consumers reference it. This prevents tight coupling between services.

## Publishing Events (Producer)

Inject `IPublishEndpoint` and publish:

```csharp
public class OrderService(AppDbContext db, IPublishEndpoint publish) : IOrderService
{
    public async Task<Guid> PlaceOrderAsync(int customerId, List<OrderItem> items)
    {
        var order = new Order
        {
            Id = Guid.NewGuid(),
            CustomerId = customerId,
            Total = items.Sum(i => i.Price * i.Quantity),
            PlacedAt = DateTime.UtcNow
        };

        db.Orders.Add(order);
        await db.SaveChangesAsync();

        // Publish the event
        await publish.Publish(new OrderPlaced(
            order.Id, customerId, order.Total, order.PlacedAt));

        return order.Id;
    }
}
```

## Consuming Events (Consumer)

Implement `IConsumer<T>`:

```csharp
public class OrderPlacedConsumer(
    AppDbContext db,
    ILogger<OrderPlacedConsumer> logger) : IConsumer<OrderPlaced>
{
    public async Task Consume(ConsumeContext<OrderPlaced> context)
    {
        var msg = context.Message;

        // Idempotency: check if already processed
        bool alreadyHandled = await db.ProcessedMessages
            .AnyAsync(m => m.Id == msg.MessageId && m.Consumer == "OrderPlacedConsumer");

        if (alreadyHandled)
        {
            logger.LogInformation("Duplicate OrderPlaced {OrderId}, skipping", msg.OrderId);
            return;
        }

        // Process the event
        var inventory = await db.Inventory
            .Where(i => msg.Items.Contains(i.ProductId))
            .ToListAsync();

        foreach (var item in inventory)
            item.ReservedQuantity++;

        await db.SaveChangesAsync();

        // Record as processed
        db.ProcessedMessages.Add(new ProcessedMessage
        {
            Id = msg.MessageId,
            Consumer = "OrderPlacedConsumer"
        });
        await db.SaveChangesAsync();
    }
}
```

## Transactional Outbox

The dual-write problem: you save to the database and publish to RabbitMQ as separate operations. If one fails, you lose consistency.

MassTransit provides a built-in outbox:

```csharp
opt.AddEntityFrameworkOutbox<AppDbContext>(o =>
{
    o.QueryDelay = TimeSpan.FromSeconds(1);
    o.UsePostgres();
    o.UseBusOutbox();
});

// In your handler — the publish is part of the DB transaction
public async Task PlaceOrderAsync(...)
{
    await using var transaction = await db.Database
        .BeginTransactionAsync();

    db.Orders.Add(order);
    await db.SaveChangesAsync();

    // This goes to the outbox table, NOT directly to RabbitMQ
    await publish.Publish(new OrderPlaced(...));

    await transaction.CommitAsync();
    // MassTransit relay process reads outbox table and delivers to RabbitMQ
}
```

If the transaction commits, the event WILL be published. No dual-write problem.

## Request/Response (RPC)

When you need a synchronous response over async messaging:

```csharp
// Request
public record GetCustomerDetails(Guid CustomerId);
public record CustomerDetailsResponse(string Name, string Email);

// Client
var response = await client.GetResponse<CustomerDetailsResponse>(
    new GetCustomerDetails(customerId));
Console.WriteLine(response.Message.Name);

// Server
public class GetCustomerDetailsConsumer(AppDbContext db) : IConsumer<GetCustomerDetails>
{
    public async Task Consume(ConsumeContext<GetCustomerDetails> context)
    {
        var customer = await db.Customers.FindAsync(context.Message.CustomerId);
        await context.RespondAsync(new CustomerDetailsResponse(
            customer.Name, customer.Email));
    }
}
```

Use sparingly — if you always need a response, consider a direct HTTP call instead.

## Common Mistakes

- **Non-idempotent consumers.** MassTransit guarantees at-least-once delivery. Redeliveries happen. Your consumer MUST handle duplicates.
- **Publishing outside the transaction.** If `SaveChangesAsync` succeeds but `Publish` fails, the event is lost. Use the outbox.
- **Sharing the DbContext instance.** Consumers are transient. Each consumer gets its own scope. Don't share `DbContext` across consumers.
- **Fat events.** Include only the data the consumer needs. Don't publish the entire entity if consumers only need an ID.
