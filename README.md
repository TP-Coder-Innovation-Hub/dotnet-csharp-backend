# .NET C# Backend Fundamentals

A structured guide to building production-grade backend systems with .NET 9+ and C# 13.

## Learning Path

| # | Topic | Level | Description |
|---|-------|-------|-------------|
| 00-01 | [What Is Programming](00-foundations/01-what-is-programming.md) |  | Instructions, data, and output |
| 00-02 | [Programming Paradigms](00-foundations/02-paradigms.md) |  | OOP-first, multi-paradigm. How C# evolved |
| 00-03 | [Sequential, Decision, Iteration](00-foundations/03-sequential-decision-iteration.md) |  | The three structures every program uses |
| 00-04 | [Compiler vs Interpreter](00-foundations/04-compiler-vs-interpreter.md) |  | C# to IL to CLR JIT. What that means |
| 00-05 | [What Is C# and .NET](00-foundations/05-what-is-csharp-dotnet.md) |  | History, open source shift, cross-platform |
| 00-06 | [Why .NET, Why Not X](00-foundations/06-why-dotnet-why-not-x.md) |  | .NET vs Java/Go/Python/Node |
| 01-01 | [Setup](01-first-code/01-setup.md) |  | SDK, editor, `dotnet new`, `dotnet run` |
| 01-02 | [Variables and Types](01-first-code/02-variables-and-types.md) |  | Strong typing, value vs reference, `var` |
| 01-03 | [Control Flow](01-first-code/03-control-flow.md) |  | if/else, switch, loops |
| 01-04 | [Methods](01-first-code/04-methods.md) |  | Params, out, ref, expression-bodied |
| 01-05 | [Classes and Records](01-first-code/05-classes-and-records.md) |  | class, record, struct, interface |
| 02-01 | [LINQ](02-core-language/01-linq.md) |  | The killer feature. Deferred execution |
| 02-02 | [Async/Await](02-core-language/02-async-await.md) |  | Task, ValueTask, state machine |
| 02-03 | [Error Handling](02-core-language/03-error-handling.md) |  | try/catch/finally, custom exceptions |
| 02-04 | [Generics](02-core-language/04-generics.md) |  | Type parameters, constraints |
| 02-05 | [Dependency Injection](02-core-language/05-dependency-injection.md) |  | Built-in DI, service lifetimes |
| 03-01 | [HTTP and Web Servers](03-backend-fundamentals/01-http-and-web-servers.md) |  | How the web works for backend devs |
| 03-02 | [REST API Design](03-backend-fundamentals/02-rest-api-design.md) |  | Principles, status codes, conventions |
| 03-03 | [Your First API](03-backend-fundamentals/03-your-first-api.md) |  | Minimal APIs step by step |
| 03-04 | [Database Access](03-backend-fundamentals/04-database-access.md) |  | EF Core: code-first, migrations, queries |
| 03-05 | [Authentication](03-backend-fundamentals/05-authentication.md) |  | JWT, ASP.NET Identity |
| 03-06 | [Configuration](03-backend-fundamentals/06-configuration.md) |  | Options pattern, IOptions, validated config |
| 03-07 | [Caching](03-backend-fundamentals/07-caching.md) |  | IMemoryCache, IDistributedCache, Redis, output caching |
| 03-08 | [Background Services](03-backend-fundamentals/08-background-services.md) |  | IHostedService, BackgroundService, timed tasks |
| 03-09 | [Messaging](03-backend-fundamentals/09-messaging.md) |  | MassTransit, RabbitMQ, publish/consume, outbox |
| 03-10 | [Security Hardening](03-backend-fundamentals/10-security-hardening.md) |  | CORS, rate limiting, Data Protection, security headers |
| 04-01 | [Testing](04-production/01-testing.md) |  | xUnit, testing handlers, EF Core InMemory |
| 04-02 | [Logging](04-production/02-logging.md) |  | Structured logging, Serilog |
| 04-03 | [Deployment](04-production/03-deployment.md) |  | Docker, Native AOT |
| 04-04 | [Full-Stack Integration](04-production/04-full-stack-integration.md) |  | SignalR, OpenAPI clients, SPA static files, JWT cookies |
| 05-00 | [Progressive Workshops](99-workshop/01-workshop-foundation.md) |  | 5 workshops: CLI → Database → API → Production → Final |

## Objectives

After completing this path you will be able to:

1. Explain the .NET runtime stack (C# -> IL -> JIT -> native code)
2. Write idiomatic C# 13 with LINQ, async/await, records, and pattern matching
3. Build REST APIs with ASP.NET Core minimal APIs
4. Persist data with EF Core (code-first, migrations, LINQ-to-SQL)
5. Secure endpoints with JWT authentication, CORS, and rate limiting
6. Use caching, background services, and async messaging (MassTransit)
7. Integrate with SPA frontends (SignalR, OpenAPI clients, SPA hosting)
8. Test with xUnit and deploy with Docker
9. Make informed technology choices (.NET vs Java vs Go vs Node)

## Audience

 developers new to .NET.  developers deepening skills.  developers evaluating the platform.

---

Part of the **TP-Coder-Innovation-Hub** learning paths.
