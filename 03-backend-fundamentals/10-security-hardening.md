# Security Hardening in ASP.NET Core

> **Concept:** See [CORS and Security Headers](https://github.com/TP-Coder-Innovation-Hub/learning-content/blob/main/auth-security/05-cors-and-headers.md) for browser security concepts. This page covers the .NET implementation of CORS, rate limiting, and Data Protection.

## CORS

Allow your React/Vue/Angular frontend to call your API from a browser.

### Setup

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddCors(opt =>
{
    opt.AddDefaultPolicy(policy =>
    {
        policy.WithOrigins(
                "https://app.example.com",
                "https://admin.example.com"
              )
              .AllowAnyHeader()
              .AllowAnyMethod()
              .AllowCredentials(); // Required for cookies, Authorization header
    });
});

var app = builder.Build();
app.UseCors(); // MUST be before UseAuthentication and UseAuthorization
```

**Rule:** Never use `AllowAnyOrigin()` with `AllowCredentials()`. Browsers reject it. Be specific about which origins are allowed.

### Per-Endpoint CORS

```csharp
builder.Services.AddCors(opt =>
{
    opt.AddPolicy("Public", p => p.WithOrigins("https://app.example.com"));
    opt.AddPolicy("Admin", p => p.WithOrigins("https://admin.example.com"));
});

app.MapGet("/api/products", () => ...).RequireCors("Public");
app.MapGet("/api/admin/stats", () => ...).RequireCors("Admin");
```

## Rate Limiting (.NET 7+)

Built-in rate limiting middleware. No third-party packages needed.

### Setup

```csharp
builder.Services.AddRateLimiter(opt =>
{
    // Global fallback: 100 requests per minute per user
    opt.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(httpContext =>
        RateLimitPartition.GetFixedWindowLimiter(
            partitionKey: httpContext.User.Identity?.Name ?? httpContext.Connection.RemoteIpAddress?.ToString() ?? "anon",
            factory: _ => new FixedWindowRateLimiterOptions
            {
                PermitLimit = 100,
                Window = TimeSpan.FromMinutes(1)
            }));

    // Specific policy for expensive endpoints
    opt.AddPolicy("Expensive", httpContext =>
        RateLimitPartition.GetTokenBucketLimiter(
            partitionKey: httpContext.User.Identity?.Name ?? "anon",
            factory: _ => new TokenBucketRateLimiterOptions
            {
                TokenLimit = 10,
                TokensPerPeriod = 5,
                ReplenishmentPeriod = TimeSpan.FromMinutes(1)
            }));

    opt.OnRejected = async (context, ct) =>
    {
        context.HttpContext.Response.StatusCode = 429;
        context.HttpContext.Response.Headers["Retry-After"] = "60";
        await context.HttpContext.Response.WriteAsJsonAsync(new
        {
            error = "Rate limit exceeded. Try again in 60 seconds."
        }, ct);
    };
});

var app = builder.Build();
app.UseRateLimiter();

// Apply specific policy
app.MapPost("/api/reports/generate", ...).RequireRateLimiting("Expensive");
```

### Algorithm Choice

| Algorithm | Method | Best For |
|---|---|---|
| **Fixed Window** | `AddFixedWindowLimiter` | Simple per-minute limits |
| **Sliding Window** | `AddSlidingWindowLimiter` | Smoother limits at window boundaries |
| **Token Bucket** | `AddTokenBucketLimiter` | Allow bursts (good for APIs) |
| **Concurrency** | `AddConcurrencyLimiter` | Limit simultaneous requests (protect resources) |

## Data Protection API

ASP.NET Core's built-in encryption for protecting sensitive data (auth cookies, CSRF tokens, temp data). It uses symmetric encryption with automatic key rotation.

### Default Behavior

The Data Protection system auto-generates and manages encryption keys. In development, keys are stored locally. In production, you must configure key persistence:

```csharp
// Persist keys to Redis (survives container restarts)
builder.Services.AddDataProtection()
    .PersistKeysToStackExchangeRedis(redis, "DataProtection-Keys")
    .SetApplicationName("myapp"); // Must be same across all instances

// Protect data
var protector = DataProtectionProvider.Create("myapp")
    .CreateProtector("PurposeString");
string encrypted = protector.Protect("sensitive data");
string decrypted = protector.Unprotect(encrypted);
```

**Why it matters:** Without shared key storage, each instance generates its own keys. Auth cookies created on instance A are invalid on instance B. Users get randomly logged out.

## Security Headers Middleware

Add security headers to every response:

```csharp
app.Use(async (context, next) =>
{
    context.Response.Headers["X-Content-Type-Options"] = "nosniff";
    context.Response.Headers["X-Frame-Options"] = "DENY";
    context.Response.Headers["Referrer-Policy"] = "strict-origin-when-cross-origin";
    context.Response.Headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains";
    context.Response.Headers["Content-Security-Policy"] = "default-src 'self'";
    await next();
});
```

Or use the official package for a more robust solution:

```bash
dotnet add package NetEscapades.AspNetCore.SecurityHeaders
```

```csharp
var headerPolicy = new HeaderPolicyCollection()
    .AddDefaultSecurityHeaders()
    .AddStrictTransportSecurityMaxAge(maxAgeInSeconds: 31536000)
    .AddContentSecurityPolicy(csp => csp.AddDefaultSrc().Self());

app.UseSecurityHeaders(headerPolicy);
```

## Common Mistakes

- **CORS before auth middleware.** Pipeline order matters: `UseCors()` must come before `UseAuthentication()` and `UseAuthorization()`.
- **Rate limiting by IP only.** Users behind a NAT share one IP. Use authenticated user ID as the partition key.
- **No Data Protection key persistence.** In multi-instance deployments, each instance has different keys. Users get logged out randomly. Persist keys to Redis or a shared file store.
- **Missing security headers.** These are free protection. Add them to every response via middleware, not per-endpoint.
