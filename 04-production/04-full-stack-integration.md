# Full-Stack Integration

> **Concept:** See [Frontend Integration](https://github.com/TP-Coder-Innovation-Hub/learning-content/blob/main/api-design/05-frontend-integration.md) for SPA auth flow, BFF pattern, and API client generation. This page covers the .NET implementation.

## SignalR: Real-Time Communication

SignalR is ASP.NET Core's built-in real-time library. It handles WebSocket, Server-Sent Events, and long-polling automatically — choosing the best transport for the client.

### Setup

```bash
dotnet add package Microsoft.AspNetCore.SignalR.Common
```

```csharp
// Define a hub
public class NotificationHub : Hub
{
    // Client calls this method
    public async Task JoinGroup(string groupName)
    {
        await Groups.AddToGroupAsync(Context.ConnectionId, groupName);
    }

    public async Task LeaveGroup(string groupName)
    {
        await Groups.RemoveFromGroupAsync(Context.ConnectionId, groupName);
    }
}
```

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddSignalR();

var app = builder.Build();

// CORS must be configured for browser clients
app.UseCors(policy => policy
    .WithOrigins("http://localhost:3000") // React dev server
    .AllowAnyHeader()
    .AllowAnyMethod()
    .AllowCredentials()); // Required for SignalR

app.MapHub<NotificationHub>("/hubs/notifications");
```

### Pushing from Server to Client

```csharp
public class OrderService(IHubContext<NotificationHub> hub) : IOrderService
{
    public async Task UpdateOrderStatusAsync(Guid orderId, string status)
    {
        // ... update logic ...

        // Push notification to all clients in the order group
        await hub.Clients
            .Group($"order-{orderId}")
            .SendAsync("OrderStatusChanged", new { OrderId = orderId, Status = status });
    }
}
```

### React Client (TypeScript)

```bash
npm install @microsoft/signalr
```

```typescript
import * as signalr from "@microsoft/signalr";

const connection = new signalr.HubConnectionBuilder()
    .withUrl("/hubs/notifications")
    .withAutomaticReconnect()
    .build();

connection.on("OrderStatusChanged", (data) => {
    console.log(`Order ${data.orderId} is now ${data.status}`);
});

await connection.start();
await connection.invoke("JoinGroup", `order-${orderId}`);
```

## OpenAPI and Client Generation

ASP.NET Core 9+ has built-in OpenAPI support. Generate a TypeScript client for your React frontend automatically.

### Enable OpenAPI

```csharp
builder.Services.AddOpenApi();

var app = builder.Build();

app.MapOpenApi(); // Serves /openapi/v1.json
```

### Generate TypeScript Client with Kiota

```bash
# Install Kiota (one-time)
dotnet tool install --global Microsoft.OpenApi.Kiota

# Generate a TypeScript client
kiota generate -l typescript \
    -d https://localhost:5001/openapi/v1.json \
    -o ./src/api-client \
    -n MyApp.ApiClient
```

This generates typed TypeScript classes with full IntelliSense. When your API changes, regenerate:

```bash
kiota update
```

### Usage in React

```typescript
import { OrdersClient } from "./api-client";

const client = new OrdersClient();

const order = await client.orders.get(orderId);
//    ^? Typed as Order — compile-time checking
```

## Serving SPA Static Files

If your React/Vue app is built and served from the .NET backend:

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddSpaStaticFiles(opt =>
{
    opt.RootPath = "wwwroot"; // Where `npm run build` outputs
});

var app = builder.Build();

app.UseStaticFiles();
app.UseSpaStaticFiles();

// API routes
app.MapGet("/api/health", () => "ok");

// All other routes → serve index.html (React Router handles routing)
app.MapFallbackToFile("index.html");

app.Run();
```

## JWT Storage for SPA Clients

### Recommended: HttpOnly Cookie

```csharp
app.MapPost("/auth/login", async (LoginRequest req, ITokenService tokens) =>
{
    var user = await ValidateCredentials(req);
    if (user is null) return Results Unauthorized();

    var (access, refresh) = tokens.Generate(user);

    // Set tokens as HttpOnly cookies — JavaScript can't read them
    Response.Cookies.Append("access_token", access, new CookieOptions
    {
        HttpOnly = true,
        Secure = true,       // HTTPS only
        SameSite = SameSiteMode.Strict,
        Expires = DateTimeOffset.UtcNow.AddMinutes(15)
    });

    Response.Cookies.Append("refresh_token", refresh, new CookieOptions
    {
        HttpOnly = true,
        Secure = true,
        SameSite = SameSiteMode.Strict,
        Expires = DateTimeOffset.UtcNow.AddDays(7),
        Path = "/auth/refresh" // Only sent to refresh endpoint
    });

    return Results.Ok();
});

// JWT middleware reads from cookie automatically
builder.Services.AddAuthentication("Cookie")
    .AddCookie("Cookie", opt =>
    {
        opt.Cookie.Name = "access_token";
        opt.Events.OnMessageReceived = ctx =>
        {
            ctx.Token = ctx.Request.Cookies["access_token"];
            return Task.CompletedTask;
        };
    });
```

## CORS for SPA Development

Your React dev server (`localhost:3000`) and API (`localhost:5001`) are different origins. Configure CORS for development:

```csharp
if (builder.Environment.IsDevelopment())
{
    builder.Services.AddCors(opt =>
    {
        opt.AddDefaultPolicy(p => p
            .WithOrigins("http://localhost:3000")
            .AllowAnyHeader()
            .AllowAnyMethod()
            .AllowCredentials());
    });
}
```

## Common Mistakes

- **Not enabling CORS for SignalR.** SignalR requires `AllowCredentials()` and specific origins. Without it, the WebSocket connection fails silently.
- **Storing JWT in localStorage.** Any XSS attack steals the token. Use HttpOnly cookies for web apps.
- **Hand-writing API clients.** When the API changes, the React client breaks at runtime. Generate from OpenAPI for compile-time safety.
- **Missing `MapFallbackToFile`.** React Router routes (e.g., `/orders/123`) return 404 from the backend without a fallback to `index.html`.
