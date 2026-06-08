# HTTP and Web Servers

`[Entry]`

## How the Web Works for Backend Developers

The web runs on HTTP (HyperText Transfer Protocol). Every interaction between a client (browser, mobile app, another server) and your backend is an HTTP request and response.

```text
Client (browser/app)                          Server (your code)
       |                                            |
       | --- HTTP REQUEST -----------------------> |
       |     Method: GET                            |
       |     URL: /api/products/42                  |
       |     Headers: Authorization, Content-Type   |
       |     Body: (empty for GET)                  |
       |                                            |
       | <--- HTTP RESPONSE -------------------- |
       |     Status: 200 OK                         |
       |     Headers: Content-Type: application/json|
       |     Body: {"id":42,"name":"Laptop",...}    |
```

## HTTP Methods

| Method | Purpose | Idempotent | Has Body |
|--------|---------|------------|----------|
| GET | Read a resource | Yes | No |
| POST | Create a resource | No | Yes |
| PUT | Replace a resource entirely | Yes | Yes |
| PATCH | Partially update a resource | No | Yes |
| DELETE | Remove a resource | Yes | No |

Idempotent means making the same request multiple times produces the same result. GET, PUT, and DELETE are idempotent. POST is not (each POST may create a new resource).

## HTTP Status Codes

| Range | Meaning | Common Codes |
|-------|---------|--------------|
| 2xx | Success | 200 OK, 201 Created, 204 No Content |
| 3xx | Redirection | 301 Moved Permanently, 304 Not Modified |
| 4xx | Client error | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found |
| 5xx | Server error | 500 Internal Server Error, 502 Bad Gateway, 503 Unavailable |

Return the correct status code. It is how clients know what happened.

## What a Web Server Does

A web server is a program that:

1. Listens for incoming HTTP requests on a port
2. Routes each request to the correct handler based on URL and method
3. Executes the handler (your code)
4. Returns an HTTP response

In ASP.NET Core, the web server is Kestrel. It is built into the framework. You do not need IIS, Nginx, or Apache to serve requests (though you can put them in front for reverse proxying).

```csharp
// This entire file is a web server
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/hello", () => "Hello, World!");

app.Run(); // Starts Kestrel, listens on port 5000 (or configured port)
```

## The Request Pipeline (Middleware)

Requests flow through a pipeline of middleware components before reaching your handler:

```text
Request --> Exception Handling --> HTTPS Redirect --> Authentication --> Routing --> Your Handler --> Response
```

Each middleware can:

- Process the request before passing it on (e.g., check authentication)
- Short-circuit and return a response immediately (e.g., return 401 if not authenticated)
- Process the response on the way back (e.g., add headers, log timing)

```csharp
var app = builder.Build();

app.UseExceptionHandler();
app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();

// Custom middleware -- log request timing
app.Use(async (context, next) =>
{
    var sw = Stopwatch.StartNew();
    await next(context);
    sw.Stop();
    logger.LogInformation("{Method} {Path} - {Status} in {Ms}ms",
        context.Request.Method, context.Request.Path,
        context.Response.StatusCode, sw.ElapsedMilliseconds);
});

app.MapGet("/products", () => GetProducts());
app.Run();
```

## Headers and Content Types

Headers carry metadata about the request and response:

```csharp
// Request headers
Authorization: Bearer eyJhbGciOi...
Content-Type: application/json
Accept: application/json

// Response headers
Content-Type: application/json
Cache-Control: no-cache
X-Request-Id: abc-123
```

`Content-Type` tells the recipient what format the body is in. For APIs, `application/json` is standard.
