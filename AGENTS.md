# AGENTS.md

## Context

This repository is part of the TP-Coder-Innovation-Hub educational platform. It serves as the ".NET C# Backend Developer" learning path, structured as a directory-per-topic guide from foundations through production deployment.

The content targets developers learning or improving their .NET backend skills, from entry-level through senior.

## Audience

- **Entry-level developers** learning .NET backend development for the first time
- **Mid-level developers** deepening their understanding of the platform
- **Senior developers** evaluating .NET for new projects or expanding their toolkit
- **Instructors and mentors** using this material to teach

Assume readers have basic programming knowledge (variables, loops, functions, HTTP) but may be new to C# or .NET specifically.

## Repository Structure

```text
dotnet-csharp-backend/
├── README.md                           # Navigation table + objectives
├── AGENTS.md                           # This file
├── 00-foundations/
│   ├── 01-what-is-programming.md       # [Entry] Instructions, data, output
│   ├── 02-paradigms.md                 # [Entry][Mid] OOP-first, multi-paradigm
│   ├── 03-sequential-decision-iteration.md # [Entry] Three control structures
│   ├── 04-compiler-vs-interpreter.md   # [Entry] C# -> IL -> CLR JIT
│   ├── 05-what-is-csharp-dotnet.md     # [Entry] History, open source, cross-platform
│   └── 06-why-dotnet-why-not-x.md      # [Mid] .NET vs Java/Go/Python/Node
├── 01-first-code/
│   ├── 01-setup.md                     # [Entry] SDK, editor, dotnet new/run
│   ├── 02-variables-and-types.md       # [Entry] Strong typing, value vs reference
│   ├── 03-control-flow.md              # [Entry] if/else, switch, loops
│   ├── 04-methods.md                   # [Entry] Params, out, ref, expression-bodied
│   └── 05-classes-and-records.md       # [Entry] class, record, struct, interface
├── 02-core-language/
│   ├── 01-linq.md                      # [Mid] The killer feature, deferred execution
│   ├── 02-async-await.md               # [Mid] Task, ValueTask, state machine
│   ├── 03-error-handling.md            # [Entry] try/catch/finally, custom exceptions
│   ├── 04-generics.md                  # [Mid] Type parameters, constraints
│   └── 05-dependency-injection.md      # [Mid] Built-in DI, service lifetimes
├── 03-backend-fundamentals/
│   ├── 01-http-and-web-servers.md      # [Entry] HTTP for backend devs
│   ├── 02-rest-api-design.md           # [Entry] REST principles, status codes
│   ├── 03-your-first-api.md            # [Entry] Minimal APIs step by step
│   ├── 04-database-access.md           # [Mid] EF Core: code-first, migrations
│   └── 05-authentication.md            # [Mid] JWT, ASP.NET Identity
├── 04-production/
│   ├── 01-testing.md                   # [Mid] xUnit, EF Core InMemory
│   ├── 02-logging.md                   # [Mid] Structured logging, Serilog
│   └── 03-deployment.md                # [Senior] Docker, Native AOT
└── 05-capstone/
    └── README.md                       # [Mid] Enterprise REST API project spec
```

## How to Help

- Reference the specific file when explaining .NET concepts and expand with practical examples
- Provide code examples using .NET 9+ and C# 13 features (primary constructors, collection expressions, pattern matching, records)
- Use ASP.NET Core minimal APIs as the default approach for new code examples unless the user specifically asks for controllers
- Prefer EF Core for data access examples; mention Dapper only when raw SQL performance is the explicit concern
- When reviewing code, check for: async void, DbContext lifetime, N+1 queries, blocking on async
- Suggest `dotnet test` for running tests, and xUnit as the default test framework
- For NuGet packages, verify compatibility with .NET 9 before recommending
- Use the `[Entry]`, `[Mid]`, `[Senior]` badges when creating new content to indicate difficulty level
- Reference official Microsoft documentation (learn.microsoft.com) as the authoritative source

## How NOT to Help

- Do not recommend .NET Framework (the legacy Windows-only version). Always default to .NET 9+
- Do not suggest third-party DI containers (Autofac, etc.) unless the user has a specific need
- Do not provide code examples using the old `Startup.cs` pattern. Modern .NET uses top-level `Program.cs`
- Do not recommend the Repository pattern on top of EF Core by default. EF Core already implements Unit of Work and Repository patterns
- Do not suggest `DataTable`, `DataSet`, or ADO.NET directly unless the user explicitly needs maximum raw SQL performance
- Do not use synchronous I/O methods in ASP.NET Core examples
- Do not recommend Visual Studio as the only option. Mention VS Code with C# Dev Kit and JetBrains Rider
- Do not provide examples that mix concerns (business logic in controllers, database access in middleware)

## Key Concepts

1. **The CLR runtime** -- JIT compilation, garbage collection, async state machine
2. **LINQ as a first-class tool** -- Not just for databases. The idiomatic way to transform and query data
3. **Async/await everywhere** -- ASP.NET Core is async throughout. Synchronous I/O causes thread pool starvation
4. **Built-in dependency injection** -- Scoped, Singleton, Transient lifetimes
5. **Minimal APIs as the default** -- Less ceremony, more readable, fully featured
6. **EF Core as the standard ORM** -- Code-first, migrations, LINQ-to-SQL translation
7. **Native AOT for specific scenarios** -- Cold-start-sensitive serverless, edge deployments
8. **Testing with xUnit** -- Unit tests, `WebApplicationFactory` for integration tests

## .NET Guidelines (2026)

### Runtime and Language

- **Target framework:** .NET 9 (LTS) or .NET 10 -- use `net9.0` or `net10.0`
- **Language version:** C# 13 -- enabled by default when targeting .NET 9+
- **Nullable reference types:** Enabled by default (`#nullable enable`)
- **Implicit usings:** Enabled (`<ImplicitUsings>enable</ImplicitUsings>`)

### ASP.NET Core

- **Minimal APIs** for new projects. `MapGet`, `MapPost`, `MapPut`, `MapDelete`
- **Top-level statements** in `Program.cs`. No `Startup.cs`
- **Built-in OpenAPI support** in .NET 9+ (`Microsoft.AspNetCore.OpenApi`)
- **Health checks** via `builder.Services.AddHealthChecks()` and `app.MapHealthChecks("/health")`
- **Structured logging** with `ILogger<T>`. Message templates, not string interpolation

### Entity Framework Core

- **EF Core 9** for .NET 9 projects
- **Code-first with migrations** as the default approach
- **`AsNoTracking()`** for all read-only queries
- **`AsSplitQuery()`** when using `Include` with multiple collections
- **PostgreSQL** (`Npgsql.EntityFrameworkCore.PostgreSQL`) as the recommended database

### Testing

- **xUnit** as the default test framework
- **`WebApplicationFactory<Program>`** for integration tests
- **`dotnet test`** to run all tests
- Test naming: `MethodName_Scenario_ExpectedBehavior`

### Containerization

- **Docker** with multi-stage builds: SDK for build, ASP.NET runtime for production
- **Chiseled containers** for minimal attack surface
- **Health check endpoints** (`/health`, `/health/ready`) for orchestrator integration

## Making Changes

1. Ensure all code examples compile against .NET 9
2. Verify API references against [learn.microsoft.com](https://learn.microsoft.com/dotnet)
3. Keep each file between 200-500 words. Short, concise, direct
4. Use `[Entry]`, `[Mid]`, `[Senior]` badges. Include runnable code examples
5. Prefer minimal APIs and modern C# features
6. Run `dotnet build` and `dotnet test` on any sample projects before committing
7. Prefer official Microsoft documentation links over blog posts
8. No emojis. Use tables, code blocks for clarity. Direct and technical
