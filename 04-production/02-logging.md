# Logging

`[Mid]`

## Why Structured Logging Matters

In production, you do not read logs line by line. You search and filter them. Structured logging stores log messages as key-value pairs, not flat strings. This makes logs queryable in aggregation systems (Seq, Datadog, CloudWatch, ELK).

## Microsoft.Extensions.Logging

ASP.NET Core includes structured logging out of the box via `ILogger<T>`:

```csharp
public class OrderService
{
    private readonly ILogger<OrderService> _logger;

    public OrderService(ILogger<OrderService> logger) => _logger = logger;

    public async Task<int> CreateOrderAsync(string email, List<OrderLine> lines)
    {
        _logger.LogInformation("Creating order for {CustomerEmail} with {LineCount} lines",
            email, lines.Count);

        var order = new Order(email, lines);
        await _db.SaveChangesAsync();

        _logger.LogInformation("Order {OrderId} created for {CustomerEmail}",
            order.Id, email);

        return order.Id;
    }
}
```

### The Key Rule: Use Message Templates, Not String Interpolation

```csharp
// CORRECT -- structured. {OrderId} is a property you can filter by.
_logger.LogInformation("Order {OrderId} created for {CustomerEmail}", orderId, email);

// WRONG -- flat string. No structure. Cannot filter by OrderId.
_logger.LogInformation($"Order {orderId} created for {email}");
```

With structured logging, you can query: "show me all logs where OrderId = 42". With string interpolation, you can only do a text search.

## Log Levels

| Level | Use For | Example |
|-------|---------|---------|
| Trace | Detailed diagnostics | Entering/exiting methods |
| Debug | Development diagnostics | Variable values, query SQL |
| Information | General flow | "Order created", "Request processed" |
| Warning | Unexpected but handled | "Slow query detected", "Retry attempt 2" |
| Error | Failures that need attention | "Database connection failed", "Payment declined" |
| Critical | Application-breaking failures | "Out of memory", "Disk full" |

```csharp
_logger.LogTrace("Entering CreateOrderAsync");
_logger.LogDebug("Lines count: {Count}", lines.Count);
_logger.LogInformation("Order {OrderId} created", order.Id);
_logger.LogWarning("Order {OrderId} processing took {Ms}ms", order.Id, elapsed);
_logger.LogError(exception, "Payment failed for order {OrderId}", order.Id);
_logger.LogCritical("Database unreachable");
```

Always pass the exception as the first parameter to `LogError` and `LogCritical`. This includes the stack trace in the log output.

## Serilog

For production, Serilog is the most popular logging provider. It supports rich output to files, consoles, and external services.

### Setup

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.Seq
```

### Configuration in Program.cs

```csharp
using Serilog;

Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .MinimumLevel.Override("Microsoft", LogEventLevel.Warning)
    .MinimumLevel.Override("Microsoft.EntityFrameworkCore", LogEventLevel.Warning)
    .Enrich.FromLogContext()
    .Enrich.WithMachineName()
    .Enrich.WithCorrelationId()
    .WriteTo.Console(outputTemplate: "[{Timestamp:HH:mm:ss} {Level:u3}] {Message:lj}{NewLine}{Exception}")
    .WriteTo.Seq("http://localhost:5341")
    .CreateLogger();

var builder = WebApplication.CreateBuilder(args);
builder.Host.UseSerilog();

var app = builder.Build();
```

- `MinimumLevel.Override("Microsoft", Warning)` -- suppresses noisy framework logs
- `Enrich.FromLogContext()` -- adds properties from the current scope (request ID, user ID)
- `WriteTo.Console()` -- formatted console output for development
- `WriteTo.Seq()` -- sends structured logs to Seq (a log search tool)

### Scoped Logging

Add context to all logs within a block:

```csharp
using (_logger.BeginScope("Processing order {OrderId} for {CustomerEmail}", orderId, email))
{
    _logger.LogInformation("Validating order");
    _logger.LogInformation("Applying discount");
    _logger.LogInformation("Saving to database");
}
// All three logs include OrderId and CustomerEmail properties
```

## Request Logging Middleware

Log every request automatically:

```csharp
app.UseSerilogRequestLogging(opt =>
{
    opt.MessageTemplate = "HTTP {RequestMethod} {RequestPath} responded {StatusCode} in {Elapsed:0.000}ms";
});
```

This replaces custom timing middleware with a one-liner.

## Production Tips

1. Set minimum level to `Information` in production. `Debug` and `Trace` generate too much data.
2. Suppress `Microsoft` namespace logs to `Warning` unless debugging framework issues.
3. Use structured properties for anything you might search by (order ID, user ID, request ID).
4. Include correlation IDs for distributed tracing.
5. Do not log sensitive data (passwords, tokens, credit card numbers).
6. Use a log aggregation service (Seq, Datadog, CloudWatch Logs, ELK) for centralized search.
