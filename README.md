# .NET C# Backend Fundamentals

A structured guide to building production-grade backend systems with .NET 9+ and C# 13.

## Learning Path

| # | Topic | Level | Description |
|---|-------|-------|-------------|
| 00-01 | [What Is Programming](00-foundations/01-what-is-programming.md) | `[Entry]` | Instructions, data, and output |
| 00-02 | [Programming Paradigms](00-foundations/02-paradigms.md) | `[Entry]` | OOP-first, multi-paradigm. How C# evolved |
| 00-03 | [Sequential, Decision, Iteration](00-foundations/03-sequential-decision-iteration.md) | `[Entry]` | The three structures every program uses |
| 00-04 | [Compiler vs Interpreter](00-foundations/04-compiler-vs-interpreter.md) | `[Entry]` | C# to IL to CLR JIT. What that means |
| 00-05 | [What Is C# and .NET](00-foundations/05-what-is-csharp-dotnet.md) | `[Entry]` | History, open source shift, cross-platform |
| 00-06 | [Why .NET, Why Not X](00-foundations/06-why-dotnet-why-not-x.md) | `[Mid]` | .NET vs Java/Go/Python/Node |
| 01-01 | [Setup](01-first-code/01-setup.md) | `[Entry]` | SDK, editor, `dotnet new`, `dotnet run` |
| 01-02 | [Variables and Types](01-first-code/02-variables-and-types.md) | `[Entry]` | Strong typing, value vs reference, `var` |
| 01-03 | [Control Flow](01-first-code/03-control-flow.md) | `[Entry]` | if/else, switch, loops |
| 01-04 | [Methods](01-first-code/04-methods.md) | `[Entry]` | Params, out, ref, expression-bodied |
| 01-05 | [Classes and Records](01-first-code/05-classes-and-records.md) | `[Entry]` | class, record, struct, interface |
| 02-01 | [LINQ](02-core-language/01-linq.md) | `[Mid]` | The killer feature. Deferred execution |
| 02-02 | [Async/Await](02-core-language/02-async-await.md) | `[Mid]` | Task, ValueTask, state machine |
| 02-03 | [Error Handling](02-core-language/03-error-handling.md) | `[Entry]` | try/catch/finally, custom exceptions |
| 02-04 | [Generics](02-core-language/04-generics.md) | `[Mid]` | Type parameters, constraints |
| 02-05 | [Dependency Injection](02-core-language/05-dependency-injection.md) | `[Mid]` | Built-in DI, service lifetimes |
| 03-01 | [HTTP and Web Servers](03-backend-fundamentals/01-http-and-web-servers.md) | `[Entry]` | How the web works for backend devs |
| 03-02 | [REST API Design](03-backend-fundamentals/02-rest-api-design.md) | `[Entry]` | Principles, status codes, conventions |
| 03-03 | [Your First API](03-backend-fundamentals/03-your-first-api.md) | `[Entry]` | Minimal APIs step by step |
| 03-04 | [Database Access](03-backend-fundamentals/04-database-access.md) | `[Mid]` | EF Core: code-first, migrations, queries |
| 03-05 | [Authentication](03-backend-fundamentals/05-authentication.md) | `[Mid]` | JWT, ASP.NET Identity |
| 04-01 | [Testing](04-production/01-testing.md) | `[Mid]` | xUnit, testing handlers, EF Core InMemory |
| 04-02 | [Logging](04-production/02-logging.md) | `[Mid]` | Structured logging, Serilog |
| 04-03 | [Deployment](04-production/03-deployment.md) | `[Senior]` | Docker, Native AOT |
| 05-00 | [Capstone Project](05-capstone/README.md) | `[Mid]` | Enterprise REST API spec |

## Objectives

After completing this path you will be able to:

1. Explain the .NET runtime stack (C# -> IL -> JIT -> native code)
2. Write idiomatic C# 13 with LINQ, async/await, records, and pattern matching
3. Build REST APIs with ASP.NET Core minimal APIs
4. Persist data with EF Core (code-first, migrations, LINQ-to-SQL)
5. Secure endpoints with JWT authentication
6. Test with xUnit and deploy with Docker
7. Make informed technology choices (.NET vs Java vs Go vs Node)

## Audience

`[Entry]` developers new to .NET. `[Mid]` developers deepening skills. `[Senior]` developers evaluating the platform.

---

Part of the **TP-Coder-Innovation-Hub** learning paths.
