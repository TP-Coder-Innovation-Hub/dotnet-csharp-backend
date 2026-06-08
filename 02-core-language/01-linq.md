# LINQ

`[Mid]`

## Why LINQ Matters

Language Integrated Query (LINQ) is the single feature that differentiates C# from most backend languages. It provides a uniform, composable, type-safe way to query and transform any data source -- in-memory collections, databases, XML, JSON, anything that implements `IEnumerable<T>` or `IQueryable<T>`.

You write the same query syntax whether you are filtering a list in memory or querying a PostgreSQL database through EF Core.

## LINQ to Objects (In-Memory)

```csharp
var products = new List<Product>
{
    new(1, "Laptop", "Electronics", 999.99m),
    new(2, "Mouse", "Electronics", 29.99m),
    new(3, "Desk", "Furniture", 249.00m),
    new(4, "Keyboard", "Electronics", 79.99m),
    new(5, "Chair", "Furniture", 399.00m)
};

record Product(int Id, string Name, string Category, decimal Price);
```

### Filtering (Where)

```csharp
var electronics = products
    .Where(p => p.Category == "Electronics")
    .ToList();
// Laptop, Mouse, Keyboard
```

### Projecting (Select)

```csharp
var names = products
    .Select(p => p.Name)
    .ToList();
// ["Laptop", "Mouse", "Desk", "Keyboard", "Chair"]
```

### Ordering

```csharp
var sorted = products
    .OrderByDescending(p => p.Price)
    .Select(p => new { p.Name, p.Price })
    .ToList();
// Laptop 999.99, Chair 399.00, Desk 249.00, ...
```

### Grouping

```csharp
var byCategory = products
    .GroupBy(p => p.Category)
    .Select(g => new { Category = g.Key, Count = g.Count(), Total = g.Sum(p => p.Price) })
    .ToList();
// Electronics: 3 items, $1109.97
// Furniture: 2 items, $648.00
```

### Aggregation

```csharp
var avgPrice = products.Average(p => p.Price);    // 351.594
var maxPrice = products.Max(p => p.Price);        // 999.99
var count    = products.Count(p => p.Price > 100); // 3
```

## LINQ to SQL (via EF Core)

The same LINQ syntax translates to SQL when querying through EF Core:

```csharp
var expensiveElectronics = await dbContext.Products
    .Where(p => p.Category == "Electronics" && p.Price > 50)
    .OrderByDescending(p => p.Price)
    .Select(p => new ProductSummary(p.Id, p.Name, p.Price))
    .AsNoTracking()
    .ToListAsync();
```

Generated SQL (approximately):

```sql
SELECT p."Id", p."Name", p."Price"
FROM "Products" AS p
WHERE p."Category" = 'Electronics' AND p."Price" > 50
ORDER BY p."Price" DESC
```

This is the key insight: **you write LINQ, EF Core generates SQL**. Type-safe. Refactorable. No raw SQL strings.

## Deferred Execution

LINQ queries do not execute when you define them. They execute when you enumerate the results.

```csharp
var query = products.Where(p => p.Price > 100);
// Nothing has been evaluated yet

products.Add(new Product(6, "Monitor", "Electronics", 599m));
// The new product IS included because evaluation is deferred

var results = query.ToList(); // Evaluation happens here
```

Terminal operators that force execution: `ToList()`, `ToArray()`, `First()`, `Count()`, `Any()`, `Sum()`, `ToListAsync()`.

For database queries, always use `ToListAsync()` or similar async terminal operators. Never use `ToList()` on an `IQueryable` in ASP.NET Core -- it blocks a thread.

## IEnumerable vs IQueryable

- `IEnumerable<T>` -- LINQ to Objects. Executes in memory. No query translation.
- `IQueryable<T>` -- LINQ to SQL (or other providers). Translates to target query language.

```csharp
// IEnumerable -- filter runs in C# after loading ALL rows
IEnumerable<Product> all = dbContext.Products;
var filtered = all.Where(p => p.Price > 100).ToList(); // Loads entire table

// IQueryable -- filter translates to SQL WHERE clause
IQueryable<Product> query = dbContext.Products;
var filtered = query.Where(p => p.Price > 100).ToList(); // SQL: WHERE Price > 100
```

Always keep queries as `IQueryable` until you apply the terminal operator. Do not accidentally convert to `IEnumerable` by calling methods like `AsEnumerable()` prematurely.

## Common Patterns

```csharp
// Pagination
var page = await dbContext.Products
    .OrderBy(p => p.Name)
    .Skip((pageNumber - 1) * pageSize)
    .Take(pageSize)
    .ToListAsync();

// Existence check
var exists = await dbContext.Users.AnyAsync(u => u.Email == email);

// Join
var ordersWithUsers = await dbContext.Orders
    .Where(o => o.CreatedAt >= DateTime.UtcNow.AddDays(-7))
    .Select(o => new { o.Id, o.Total, o.User.Email })
    .ToListAsync();

// Distinct
var categories = await dbContext.Products
    .Select(p => p.Category)
    .Distinct()
    .ToListAsync();
```

LINQ is not an optional feature in C#. It is the standard way to transform data. Master it early.
