# Database Access with EF Core

`[Mid]`

## What Is EF Core

Entity Framework Core is the official .NET ORM (Object-Relational Mapper). It maps C# classes to database tables and translates LINQ queries to SQL. You write C#. EF Core generates the database schema and queries.

## Code-First: Define Entities, Generate Schema

### Step 1: Install the Provider

```bash
dotnet add package Microsoft.EntityFrameworkCore.Npgsql
dotnet add package Microsoft.EntityFrameworkCore.Design
```

PostgreSQL (`Npgsql`) is the recommended provider for new projects. For SQL Server, use `Microsoft.EntityFrameworkCore.SqlServer`.

### Step 2: Define Entities

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Category { get; set; } = "";
    public decimal Price { get; set; }
    public int StockQuantity { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public List<OrderLine> OrderLines { get; set; } = [];
}

public class Order
{
    public int Id { get; set; }
    public string CustomerEmail { get; set; } = "";
    public decimal Total { get; set; }
    public OrderStatus Status { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public List<OrderLine> Lines { get; set; } = [];
}

public class OrderLine
{
    public int Id { get; set; }
    public int OrderId { get; set; }
    public Order Order { get; set; } = null!;
    public int ProductId { get; set; }
    public Product Product { get; set; } = null!;
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
}

public enum OrderStatus { Pending, Processing, Shipped, Delivered, Cancelled }
```

### Step 3: Define the DbContext

```csharp
public class AppDbContext : DbContext
{
    public DbSet<Product> Products => Set<Product>();
    public DbSet<Order> Orders => Set<Order>();
    public DbSet<OrderLine> OrderLines => Set<OrderLine>();

    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Product>(entity =>
        {
            entity.HasKey(p => p.Id);
            entity.HasIndex(p => p.Name);
            entity.Property(p => p.Price).HasPrecision(18, 2);
        });

        modelBuilder.Entity<Order>(entity =>
        {
            entity.HasKey(o => o.Id);
            entity.Property(o => o.Total).HasPrecision(18, 2);
            entity.HasMany(o => o.Lines)
                .WithOne(l => l.Order)
                .HasForeignKey(l => l.OrderId);
        });
    }
}
```

### Step 4: Register in DI

```csharp
builder.Services.AddDbContext<AppDbContext>(opt =>
    opt.UseNpgsql(builder.Configuration.GetConnectionString("Default")));
```

In `appsettings.json`:

```json
{
    "ConnectionStrings": {
        "Default": "Host=localhost;Port=5432;Database=productdb;Username=postgres;Password=secret"
    }
}
```

## Migrations

Migrations track schema changes alongside your code. Each migration is a C# file that describes the change.

```bash
# Create a migration after changing entities
dotnet ef migrations add InitialCreate

# Apply migrations to the database
dotnet ef database update

# Generate SQL for review (without applying)
dotnet ef migrations script --idempotent --output migration.sql
```

The migration creates a `Migrations/` folder with timestamped files. Apply them in CI/CD before deployment.

## Querying with LINQ

```csharp
// Simple filter
var electronics = await db.Products
    .Where(p => p.Category == "Electronics")
    .AsNoTracking()
    .ToListAsync();

// Projection (select specific columns)
var summaries = await db.Products
    .Where(p => p.StockQuantity > 0)
    .Select(p => new ProductSummary(p.Id, p.Name, p.Price))
    .AsNoTracking()
    .ToListAsync();

// Join via navigation property
var ordersWithProducts = await db.Orders
    .Include(o => o.Lines)
        .ThenInclude(l => l.Product)
    .Where(o => o.Status == OrderStatus.Pending)
    .AsNoTracking()
    .ToListAsync();

// Aggregation
var revenue = await db.Orders
    .Where(o => o.Status == OrderStatus.Delivered)
    .SumAsync(o => o.Total);

// Pagination
var page2 = await db.Products
    .OrderBy(p => p.Name)
    .Skip(20)
    .Take(20)
    .AsNoTracking()
    .ToListAsync();
```

## Inserting and Updating

```csharp
// Insert
var product = new Product
{
    Name = "Wireless Keyboard",
    Category = "Electronics",
    Price = 79.99m,
    StockQuantity = 150
};
db.Products.Add(product);
await db.SaveChangesAsync();
// product.Id is now set

// Update (read-modify-save)
var order = await db.Orders.FindAsync(orderId);
order.Status = OrderStatus.Shipped;
await db.SaveChangesAsync(); // Generates UPDATE SQL
```

`SaveChangesAsync` wraps all pending changes in a transaction automatically.

## Tracking vs No-Tracking

- **Tracked** (default) -- EF Core tracks entity changes for `SaveChanges`
- **AsNoTracking()** -- no tracking overhead. Use for all read-only queries.

```csharp
// Read-only -- always use AsNoTracking
var products = await db.Products.AsNoTracking().ToListAsync();

// Read-modify-save -- tracking needed
var order = await db.Orders.FindAsync(id);
order.Status = OrderStatus.Shipped;
await db.SaveChangesAsync();
```

Add `AsNoTracking()` to every query that does not update entities. It is a significant performance improvement.

## N+1 Query Problem

The most common EF Core performance mistake:

```csharp
// WRONG -- N+1 queries
var orders = await db.Orders.ToListAsync(); // 1 query
foreach (var order in orders)
{
    var lines = await db.OrderLines
        .Where(l => l.OrderId == order.Id)
        .ToListAsync(); // N queries!
}

// CORRECT -- eager loading
var orders = await db.Orders
    .Include(o => o.Lines)
    .AsSplitQuery() // Separate queries instead of Cartesian JOIN
    .ToListAsync();
```

Always check generated SQL during development. Add logging:

```csharp
builder.Services.AddDbContext<AppDbContext>(opt =>
    opt.UseNpgsql(connectionString)
       .EnableSensitiveDataLogging() // Dev only
       .LogTo(Console.WriteLine, LogLevel.Information));
```
