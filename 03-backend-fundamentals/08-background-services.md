# Background Services in ASP.NET Core

> **Concept:** See [Background Jobs](https://github.com/TP-Coder-Innovation-Hub/learning-content/blob/main/production-readiness/07-background-jobs.md) for the worker pattern, scheduling, and retry concepts. This page covers the .NET implementation.

## IHostedService vs BackgroundService

| Base Class | Use Case |
|---|---|
| `IHostedService` | Fine-grained control over start/stop logic |
| `BackgroundService` | Long-running async work (easier, most common choice) |

## BackgroundService: Long-Running Tasks

A simple base class. Override `ExecuteAsync` and run a loop.

```csharp
public class EmailQueueWorker(
    IServiceProvider services,
    ILogger<EmailQueueWorker> logger) : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            using var scope = services.CreateScope();
            var emailService = scope.ServiceProvider.GetRequiredService<IEmailService>();
            var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();

            var pending = await db.EmailQueue
                .Where(e => e.Status == "pending")
                .Take(50)
                .ToListAsync(stoppingToken);

            foreach (var email in pending)
            {
                try
                {
                    await emailService.SendAsync(email.To, email.Subject, email.Body);
                    email.Status = "sent";
                }
                catch (Exception ex)
                {
                    logger.LogError(ex, "Failed to send email {Id}", email.Id);
                    email.Status = "failed";
                    email.RetryCount++;
                }
            }

            await db.SaveChangesAsync(stoppingToken);
            await Task.Delay(TimeSpan.FromSeconds(10), stoppingToken);
        }
    }
}
```

**Critical:** Use `IServiceScopeFactory` to create a scope inside the loop. Background services are registered as **singletons**, but `DbContext` is **scoped**. Injecting scoped services into a singleton causes a captive dependency bug.

## Timed Background Service

For periodic tasks (every N seconds/minutes):

```csharp
public class CacheWarmerService(
    IServiceProvider services,
    ILogger<CacheWarmerService> logger) : BackgroundService
{
    private readonly TimeSpan _interval = TimeSpan.FromMinutes(5);

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        using var timer = new PeriodicTimer(_interval);

        do
        {
            try
            {
                using var scope = services.CreateScope();
                var cache = scope.ServiceProvider.GetRequiredService<IDistributedCache>();
                var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();

                // Warm the cache with popular products
                var products = await db.Products
                    .Where(p => p.IsFeatured)
                    .ToListAsync(stoppingToken);

                foreach (var product in products)
                {
                    await cache.SetStringAsync(
                        $"product:{product.Id}",
                        JsonSerializer.Serialize(product),
                        stoppingToken);
                }

                logger.LogInformation("Warmed {Count} products in cache", products.Count);
            }
            catch (Exception ex) when (ex is not OperationCanceledException)
            {
                logger.LogError(ex, "Cache warming failed");
            }
        } while (await timer.WaitForNextTickAsync(stoppingToken));
    }
}
```

`PeriodicTimer` (not `Task.Delay`) prevents timer drift and doesn't fire while the previous tick is still running.

## Scheduled Jobs with IHostedLifecycleService (.NET 9)

.NET 9 adds `IHostedLifecycleService` for more control over the hosted service lifecycle:

```csharp
public class MonthlyReportService(
    IServiceProvider services,
    ILogger<MonthlyReportService> logger) : IHostedLifecycleService
{
    private Timer? _timer;

    public Task StartAsync(CancellationToken ct)
    {
        // Calculate next run at 2 AM on the 1st of each month
        var nextRun = DateTime.Today.AddDays(1).AddHours(2);
        var delay = nextRun - DateTime.Now;
        _timer = new Timer(RunJob!, null, delay, TimeSpan.FromDays(30));
        return Task.CompletedTask;
    }

    private async void RunJob(object state)
    {
        using var scope = services.CreateScope();
        var reportService = scope.ServiceProvider
            .GetRequiredService<IReportService>();
        await reportService.GenerateMonthlyReportAsync();
    }

    public Task StopAsync(CancellationToken ct)
    {
        _timer?.Dispose();
        return Task.CompletedTask;
    }

    // Lifecycle hooks (optional, can be no-ops)
    public Task StartingAsync(CancellationToken ct) => Task.CompletedTask;
    public Task StartedAsync(CancellationToken ct) => Task.CompletedTask;
    public Task StoppingAsync(CancellationToken ct) => Task.CompletedTask;
    public Task StoppedAsync(CancellationToken ct) => Task.CompletedTask;
}
```

## Registration

```csharp
// Registers as singleton, starts automatically with the app
builder.Services.AddHostedService<EmailQueueWorker>();
builder.Services.AddHostedService<CacheWarmerService>();
builder.Services.AddHostedService<MonthlyReportService>();
```

## Graceful Shutdown

ASP.NET Core handles shutdown automatically:

1. Application receives SIGTERM (container stop, `Ctrl+C`)
2. Cancellation token is triggered
3. `ExecuteAsync` must check `stoppingToken` and exit
4. Host waits up to `ShutdownTimeout` (default 5 seconds) for services to finish

Configure the timeout:

```csharp
builder.Services.Configure<HostOptions>(opt =>
{
    opt.ShutdownTimeout = TimeSpan.FromSeconds(30); // Give jobs time to finish
});
```

## When to Use a Queue Instead

`BackgroundService` is great for lightweight, periodic tasks. But if:
- You need guaranteed delivery (jobs survive crashes)
- You need horizontal scaling (multiple workers processing the same queue)
- You need retry policies and dead-letter queues

Use a real message broker. See [Messaging](./09-messaging.md) for MassTransit + RabbitMQ.
