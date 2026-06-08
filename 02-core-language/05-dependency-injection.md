# Dependency Injection

`[Mid]`

## What Is Dependency Injection

Dependency Injection (DI) is a technique where a class receives its dependencies from outside rather than creating them itself. This makes code testable, loosely coupled, and easier to change.

```csharp
// Without DI -- tightly coupled
public class OrderService
{
    private readonly AppDbContext _db = new(); // Creates its own dependency
    private readonly EmailSender _email = new(); // Cannot swap or test
}

// With DI -- dependencies provided from outside
public class OrderService
{
    private readonly AppDbContext _db;
    private readonly IEmailSender _email;

    public OrderService(AppDbContext db, IEmailSender email)
    {
        _db = db;
        _email = email;
    }
}
```

In the DI version, `OrderService` does not know or care which `IEmailSender` implementation it gets. In production, it gets the real one. In tests, it gets a fake.

## ASP.NET Core Has Built-In DI

You do not need Autofac, Ninject, or any third-party container. ASP.NET Core ships with a capable DI container.

### Registration (Program.cs)

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register services
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddScoped<IOrderRepository, OrderRepository>();
builder.Services.AddSingleton<ICacheService, RedisCacheService>();
builder.Services.AddTransient<IEmailSender, SmtpEmailSender>();

// EF Core is registered as scoped by default
builder.Services.AddDbContext<AppDbContext>(opt =>
    opt.UseNpgsql(builder.Configuration.GetConnectionString("Default")));
```

### Resolution

The DI container resolves dependencies automatically:

```csharp
app.MapPost("/orders", async (CreateOrderCommand cmd, IOrderService service) =>
{
    var orderId = await service.CreateOrderAsync(cmd);
    return Results.Created($"/orders/{orderId}", new { OrderId = orderId });
});
```

`IOrderService` is resolved from the container. The handler never creates it.

## Service Lifetimes

| Lifetime | Created | Use For | Example |
|----------|---------|---------|---------|
| **Transient** | Every time it is requested | Lightweight, stateless services | Email sender, validators |
| **Scoped** | Once per HTTP request | Database context, repositories | DbContext, unit of work |
| **Singleton** | Once for the application | Caches, configuration, expensive resources | Redis client, settings |

```csharp
builder.Services.AddTransient<IValidator<Order>, OrderValidator>();  // New instance each time
builder.Services.AddScoped<IOrderRepository, OrderRepository>();     // One per request
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();   // One forever
```

### The Critical Rule

Never inject a scoped service into a singleton. The singleton captures the scoped instance and holds it forever, causing concurrency issues and stale data.

```csharp
// WRONG -- Scoped DbContext injected into Singleton
public class BackgroundCache(INotificationService notifications, AppDbContext db)
{
    // db will be stale, shared across requests, not thread-safe
}

// CORRECT -- Use IServiceScopeFactory in singletons
public class BackgroundCache(INotificationService notifications, IServiceScopeFactory scopeFactory)
{
    public async Task RefreshAsync()
    {
        using var scope = scopeFactory.CreateScope();
        var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
        // db is fresh, scoped to this operation
    }
}
```

## Why DI: Testability

DI enables unit testing with mocks or fakes:

```csharp
public class OrderServiceTests
{
    [Fact]
    public async Task CreateOrder_ValidInput_ReturnsOrderId()
    {
        // Arrange
        var db = new InMemoryDbContext(); // Fake
        var email = new FakeEmailSender(); // Fake
        var service = new OrderService(db, email);

        // Act
        var id = await service.CreateOrderAsync("alice@test.com", testLines);

        // Assert
        Assert.NotEqual(0, id);
        Assert.Equal(1, email.SentEmails.Count); // Verify email was sent
    }
}
```

Without DI, you cannot inject fakes. You would need the real database and real email server to test.

## Why DI: Loose Coupling

```csharp
// Swap implementation without changing OrderService
builder.Services.AddScoped<IEmailSender, SendGridEmailSender>(); // Production
builder.Services.AddScoped<IEmailSender, FakeEmailSender>();     // Development
```

The service that depends on `IEmailSender` does not change. Only the registration changes.

## Constructor Injection vs Method Injection

Constructor injection is the standard. Use method injection (via endpoint parameters) in minimal APIs:

```csharp
// Constructor injection (services)
public class OrderService(AppDbContext db, ILogger<OrderService> logger) { }

// Method injection (minimal API endpoints)
app.MapGet("/orders/{id}", async (int id, AppDbContext db) =>
{
    return await db.Orders.FindAsync(id);
});
```

Both are resolved by the DI container.
