# REST API Design



## What Is REST

REST (Representational State Transfer) is a set of conventions for building APIs over HTTP. A REST API exposes resources (nouns) through URLs, uses HTTP methods (verbs) for actions, and returns standard status codes.

## URL Structure

```text
GET    /api/products          -- List all products
GET    /api/products/42       -- Get product 42
POST   /api/products          -- Create a new product
PUT    /api/products/42       -- Replace product 42 entirely
PATCH  /api/products/42       -- Partially update product 42
DELETE /api/products/42       -- Delete product 42
```

Rules:

- Nouns, not verbs. `/api/products` not `/api/getProducts`
- Plural for collections. `/products` not `/product`
- Nested resources for relationships: `GET /api/orders/42/lines`
- Use query parameters for filtering, sorting, pagination: `GET /api/products?category=electronics&sort=price&page=2`

## Request and Response Bodies

Use JSON. Always.

```json
// POST /api/products
// Request body:
{
    "name": "Wireless Mouse",
    "category": "Electronics",
    "price": 29.99
}

// Response (201 Created):
{
    "id": 42,
    "name": "Wireless Mouse",
    "category": "Electronics",
    "price": 29.99,
    "createdAt": "2026-06-08T10:30:00Z"
}
```

## Status Codes

Return the right code:

```csharp
app.MapPost("/products", async (CreateProductCommand cmd, AppDbContext db) =>
{
    var product = new Product(cmd.Name, cmd.Category, cmd.Price);
    db.Products.Add(product);
    await db.SaveChangesAsync();
    return Results.Created($"/products/{product.Id}", product);
    // 201 Created with Location header
});

app.MapGet("/products/{id}", async (int id, AppDbContext db) =>
{
    var product = await db.Products.FindAsync(id);
    return product is not null
        ? Results.Ok(product)       // 200 OK
        : Results.NotFound();       // 404 Not Found
});

app.MapDelete("/products/{id}", async (int id, AppDbContext db) =>
{
    var product = await db.Products.FindAsync(id);
    if (product is null) return Results.NotFound();
    db.Products.Remove(product);
    await db.SaveChangesAsync();
    return Results.NoContent(); // 204 No Content
});
```

## Pagination

Never return unbounded lists. Always paginate:

```csharp
app.MapGet("/products", async (AppDbContext db, int page = 1, int size = 20) =>
{
    var total = await db.Products.CountAsync();
    var items = await db.Products
        .OrderBy(p => p.Name)
        .Skip((page - 1) * size)
        .Take(size)
        .Select(p => new ProductSummary(p.Id, p.Name, p.Price))
        .AsNoTracking()
        .ToListAsync();

    return Results.Ok(new
    {
        Items = items,
        Total = total,
        Page = page,
        Size = size,
        HasNext = page * size < total
    });
});
```

## Error Responses

Return consistent error objects:

```json
// 400 Bad Request
{
    "error": "Validation failed",
    "details": {
        "price": ["Must be greater than 0"],
        "name": ["Required"]
    }
}
```

```csharp
app.MapPost("/products", async (CreateProductCommand cmd, AppDbContext db) =>
{
    if (string.IsNullOrWhiteSpace(cmd.Name))
        return Results.ValidationProblem(new Dictionary<string, string[]>
        {
            ["name"] = ["Name is required"]
        });

    // ...
});
```

## Versioning

Plan for change. Version your API:

- **URL versioning:** `/api/v1/products` (simple, visible)
- **Header versioning:** `Accept: application/vnd.myapi.v1+json` (cleaner URLs)

URL versioning is more common and easier to implement:

```csharp
var v1 = app.MapGroup("/api/v1");
v1.MapGet("/products", GetProductsV1);

var v2 = app.MapGroup("/api/v2");
v2.MapGet("/products", GetProductsV2);
```

## REST Is a Convention, Not a Standard

REST has no governing body and no specification. It is a set of conventions. The value is consistency: any developer who knows REST can understand your API without reading your documentation first.

The conventions that matter most:

1. Resources are nouns (URLs), actions are verbs (HTTP methods)
2. Return correct status codes
3. Use JSON
4. Paginate lists
5. Return consistent error shapes
