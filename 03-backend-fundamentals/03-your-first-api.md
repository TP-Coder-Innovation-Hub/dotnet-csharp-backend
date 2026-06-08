# Your First API



Build a minimal REST API for managing products. Every line explained.

## Step 1: Create the Project

```bash
dotnet new webapi -minimal -n ProductApi
cd ProductApi
dotnet add package Microsoft.EntityFrameworkCore.InMemory
```

The InMemory package lets us use EF Core with an in-memory database. No PostgreSQL or SQL Server needed to start.

## Step 2: Define the Data Model

Add this to `Program.cs` above `var builder = ...`:

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Category { get; set; } = "";
    public decimal Price { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
}

public record CreateProductRequest(string Name, string Category, decimal Price);
public record ProductResponse(int Id, string Name, string Category, decimal Price);
```

- `Product` -- the entity class. Matches a database table row. Has a mutable `Id` that EF Core sets on insert.
- `CreateProductRequest` -- a record. Input from the client. Immutable.
- `ProductResponse` -- a record. Output to the client. Hides `CreatedAt` and other internal fields.

## Step 3: Register the Database

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register EF Core with in-memory database
builder.Services.AddDbContext<AppDbContext>(opt =>
    opt.UseInMemoryDatabase("ProductsDb"));

builder.Services.AddOpenApi();

var app = builder.Build();
```

- `AddDbContext<AppDbContext>` -- registers the DbContext in the DI container as scoped (one per request)
- `UseInMemoryDatabase("ProductsDb")` -- stores data in memory. Data is lost on restart.

## Step 4: Define the DbContext

```csharp
public class AppDbContext : DbContext
{
    public DbSet<Product> Products => Set<Product>();

    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
}
```

- `DbSet<Product>` -- represents the Products table. `Set<Product>()` returns the DbSet from the DbContext.
- The constructor accepts `DbContextOptions` which EF Core provides via DI with the InMemory configuration.

## Step 5: Create Endpoints

```csharp
// GET /products -- list all products
app.MapGet("/products", async (AppDbContext db) =>
{
    var products = await db.Products
        .Select(p => new ProductResponse(p.Id, p.Name, p.Category, p.Price))
        .AsNoTracking()
        .ToListAsync();

    return Results.Ok(products);
});

// GET /products/{id} -- get one product
app.MapGet("/products/{id:int}", async (int id, AppDbContext db) =>
{
    var product = await db.Products.FindAsync(id);
    return product is not null
        ? Results.Ok(new ProductResponse(product.Id, product.Name, product.Category, product.Price))
        : Results.NotFound(new { Error = $"Product {id} not found" });
});

// POST /products -- create a product
app.MapPost("/products", async (CreateProductRequest req, AppDbContext db) =>
{
    if (string.IsNullOrWhiteSpace(req.Name))
        return Results.BadRequest(new { Error = "Name is required" });

    var product = new Product
    {
        Name = req.Name,
        Category = req.Category,
        Price = req.Price
    };

    db.Products.Add(product);
    await db.SaveChangesAsync();

    return Results.Created($"/products/{product.Id}",
        new ProductResponse(product.Id, product.Name, product.Category, product.Price));
});

// DELETE /products/{id} -- delete a product
app.MapDelete("/products/{id:int}", async (int id, AppDbContext db) =>
{
    var product = await db.Products.FindAsync(id);
    if (product is null) return Results.NotFound();

    db.Products.Remove(product);
    await db.SaveChangesAsync();
    return Results.NoContent();
});

app.Run();
```

### Line-by-line for the POST endpoint:

1. `app.MapPost("/products", ...)` -- maps HTTP POST to `/products`
2. `async (CreateProductRequest req, AppDbContext db) =>` -- parameters resolved by DI. `req` comes from the request body, `db` from the container
3. `db.Products.Add(product)` -- starts tracking the entity (not yet saved)
4. `await db.SaveChangesAsync()` -- executes the INSERT and sets `product.Id`
5. `Results.Created(...)` -- returns HTTP 201 with Location header and response body

## Step 6: Run and Test

```bash
dotnet run
```

Test with curl:

```bash
# Create a product
curl -X POST http://localhost:5000/products \
  -H "Content-Type: application/json" \
  -d '{"name":"Laptop","category":"Electronics","price":999.99}'

# List products
curl http://localhost:5000/products

# Get product 1
curl http://localhost:5000/products/1

# Delete product 1
curl -X DELETE http://localhost:5000/products/1
```

## The Complete File

Your entire API is one `Program.cs` file, roughly 80 lines. No controller classes. No startup configuration files. One file, one API, running.

This is the ASP.NET Core minimal API approach. For small services, this is all you need. As the API grows, you extract services and move handlers into separate files. But the starting point is minimal.
