# Classes and Records



## class

A class defines a reference type that bundles data and behavior.

```csharp
public class Order
{
    public int Id { get; set; }
    public string CustomerEmail { get; set; }
    public decimal Total { get; set; }
    public OrderStatus Status { get; set; }
    public DateTime CreatedAt { get; set; }

    public Order(string customerEmail)
    {
        CustomerEmail = customerEmail;
        Status = OrderStatus.Pending;
        CreatedAt = DateTime.UtcNow;
    }

    public void Cancel()
    {
        if (Status == OrderStatus.Shipped)
            throw new InvalidOperationException("Cannot cancel a shipped order");

        Status = OrderStatus.Cancelled;
    }
}

public enum OrderStatus { Pending, Processing, Shipped, Delivered, Cancelled }
```

Classes are reference types. Two variables can point to the same object:

```csharp
var order1 = new Order("alice@example.com");
var order2 = order1;
order2.Status = OrderStatus.Shipped;
Console.WriteLine(order1.Status); // Shipped
```

## Constructor and Primary Constructor (C# 12+)

```csharp
// Traditional constructor
public class ProductService
{
    private readonly IProductRepository _repo;

    public ProductService(IProductRepository repo)
    {
        _repo = repo;
    }
}

// Primary constructor (C# 12+) -- shorter, same behavior
public class ProductService(IProductRepository repo)
{
    public async Task<Product> GetAsync(int id) =>
        await repo.FindAsync(id);
}
```

## interface

Defines a contract. A class implements it.

```csharp
public interface IOrderService
{
    Task<int> CreateOrderAsync(string email, List<OrderLine> lines);
    Task<Order> GetOrderAsync(int id);
}

public class OrderService : IOrderService
{
    private readonly AppDbContext _db;

    public OrderService(AppDbContext db) => _db = db;

    public async Task<int> CreateOrderAsync(string email, List<OrderLine> lines)
    {
        var order = new Order(email) { Lines = lines };
        _db.Orders.Add(order);
        await _db.SaveChangesAsync();
        return order.Id;
    }

    public async Task<Order> GetOrderAsync(int id) =>
        await _db.Orders.FindAsync(id)
            ?? throw new NotFoundException($"Order {id} not found");
}
```

Use interfaces for dependency injection, testing (mock the interface), and decoupling.

## struct

A value type for small, simple data. Copied by value, not reference.

```csharp
public struct Money
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }

    public Money Add(Money other) =>
        Currency != other.Currency
            ? throw new InvalidOperationException("Currency mismatch")
            : new Money(Amount + other.Amount, Currency);
}
```

Use `struct` for small, immutable values (coordinates, money, dates). For anything larger than 16 bytes, use `class` to avoid expensive copying.

## record -- C#'s Modern Data Type

Records are reference types with value-based equality and immutability. They are the preferred way to define data carriers in modern C#.

```csharp
// Positional record -- concise, immutable
public record UserDto(int Id, string Email, string Role);

// Record with body
public record OrderResponse(int Id, decimal Total, OrderStatus Status)
{
    public string Summary => $"Order {Id}: {Total:C} ({Status})";
}
```

### Why Records Over Classes for Data

```csharp
// With records, equality works by value
var user1 = new UserDto(1, "alice@example.com", "Admin");
var user2 = new UserDto(1, "alice@example.com", "Admin");
Console.WriteLine(user1 == user2); // True -- value equality

// With classes, equality is by reference
var c1 = new UserDtoClass(1, "alice@example.com", "Admin");
var c2 = new UserDtoClass(1, "alice@example.com", "Admin");
Console.WriteLine(c1 == c2); // False -- reference equality
```

### Non-Destructive Mutation (with expression)

```csharp
var original = new UserDto(1, "alice@example.com", "Admin");
var updated = original with { Role = "SuperAdmin" };
// original is unchanged. updated is a new record.
```

### When to Use What

| Type | Use For | Equality | Mutability |
|------|---------|----------|------------|
| `class` | Services, entities with behavior | Reference | Mutable by default |
| `record` | DTOs, commands, events, responses | Value | Immutable by default |
| `struct` | Small values (Point, Money) | Value | Mutable by default |
| `interface` | Contracts, abstractions | N/A | N/A |

In ASP.NET Core APIs, most of your data types should be records. Most of your services should be classes implementing interfaces.
