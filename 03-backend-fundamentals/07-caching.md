# Caching in ASP.NET Core

> **Concept:** See [Caching](https://github.com/TP-Coder-Innovation-Hub/learning-content/blob/main/production-readiness/06-caching.md) for cache-aside, write-through, TTL, and invalidation strategies. This page covers the .NET implementation.

## Three Caching Layers in ASP.NET Core

| Layer | API | Scope | Best For |
|---|---|---|---|
| **In-memory** | `IMemoryCache` | Per-instance | Small data, low-stakes, single instance |
| **Distributed** | `IDistributedCache` | Cross-instance | Multi-instance, persistent, shared state |
| **Response** | Output Caching Middleware | HTTP response | Caching full endpoint responses |

## IMemoryCache

Fast, in-process cache. Lost on restart. Each instance has its own cache.

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddMemoryCache();
var app = builder.Build();
```

```csharp
public class UserService(AppDbContext db, IMemoryCache cache)
{
    public async Task<User?> GetUserAsync(int id)
    {
        string key = $"user:{id}";

        // Cache-aside pattern
        if (cache.TryGetValue(key, out User? cached))
            return cached;

        var user = await db.Users.FindAsync(id);
        if (user is not null)
        {
            cache.Set(key, user, TimeSpan.FromMinutes(5));
        }
        return user;
    }

    public async Task UpdateUserAsync(User user)
    {
        db.Users.Update(user);
        await db.SaveChangesAsync();
        cache.Remove($"user:{user.Id}"); // Explicit invalidation
    }
}
```

## IDistributedCache with Redis

For multiple instances or persistence. Redis is the standard backing store.

### Setup

```bash
dotnet add package Microsoft.Extensions.Caching.StackExchangeRedis
```

```csharp
builder.Services.AddStackExchangeRedisCache(opt =>
{
    opt.Configuration = builder.Configuration.GetConnectionString("Redis");
    opt.InstanceName = "myapp_";
});
```

### Usage

```csharp
public class ProductService(IDistributedCache cache, AppDbContext db)
{
    public async Task<Product?> GetProductAsync(int id)
    {
        string key = $"product:{id}";
        byte[]? cached = await cache.GetAsync(key);

        if (cached is not null)
        {
            return JsonSerializer.Deserialize<Product>(cached);
        }

        var product = await db.Products.FindAsync(id);
        if (product is not null)
        {
            var json = JsonSerializer.SerializeToUtf8Bytes(product);
            var options = new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
            };
            await cache.SetAsync(key, json, options);
        }
        return product;
    }
}
```

**Note:** `IDistributedCache` works with `byte[]` and `string`. You serialize/deserialize yourself. For a more ergonomic API, consider caching helper extensions or third-party libraries like `FusionCache`.

## Output Caching (.NET 7+)

Cache entire HTTP responses. The middleware handles storing and returning cached responses.

```csharp
builder.Services.AddOutputCache(opt =>
{
    opt.AddPolicy("Expensive", policy =>
        policy.Expire(TimeSpan.FromMinutes(5))
              .Tag("expensive"));
});

var app = builder.Build();
app.UseOutputCache();

// Cache this endpoint for 5 minutes
app.MapGet("/api/reports/monthly",
    (ReportService svc) => svc.GenerateMonthlyReport())
   .CacheOutput("Expensive");
```

Benefits:
- No cache logic in your handler
- Vary by query string, headers, or route values
- Evict by tag (`app.OutputCacheStore.EvictByTagAsync("expensive")`)

## Cache Invalidation

The hardest part. Strategies in .NET:

### Explicit Removal

```csharp
public async Task DeleteProductAsync(int id)
{
    db.Products.Remove(new Product { Id = id });
    await db.SaveChangesAsync();
    await cache.RemoveAsync($"product:{id}");
}
```

### Tag-Based Eviction (Output Cache)

```csharp
// Cache with a tag
app.MapGet("/api/products/{id}", ...).CacheOutput(p => p.Tag("products"));

// Evict all products when any product changes
await app.OutputCacheStore.EvictByTagAsync("products", default);
```

## Common Mistakes

- **Caching null/error results.** If `FindAsync` returns null, don't cache it. Every subsequent request gets a cached null.
- **Forgetting to invalidate.** Update the database but forget to remove the cache entry. Users see stale data until TTL expires.
- **Using IMemoryCache in multi-instance.** Instance A has the cached data, instance B doesn't. Use `IDistributedCache` (Redis) when you scale horizontally.
- **No TTL.** `cache.Set(key, value)` without an expiration lives forever. Always pass a `TimeSpan`.
