# AGENTS.md

## Context

This repository is part of the TP-Coder-Innovation-Hub educational platform. It serves as the entry point for the ".NET C# Backend Developer" learning path. The content targets developers learning or improving their .NET backend skills, from entry-level through senior.

The primary content is a comprehensive guide (`README.md`) covering .NET fundamentals, C# language features, ASP.NET Core, EF Core, and the broader ecosystem.

## Audience

- **Entry-level developers** learning .NET backend development for the first time
- **Mid-level developers** deepening their understanding of the platform
- **Senior developers** evaluating .NET for new projects or expanding their toolkit
- **Instructors and mentors** using this material to teach

Assume readers have basic programming knowledge (variables, loops, functions, HTTP) but may be new to C# or .NET specifically.

## How to Help

- When asked to explain .NET concepts, reference the specific sections in README.md and expand with practical examples
- Provide code examples using .NET 9+ and C# 13 features (primary constructors, collection expressions, pattern matching, records)
- Use ASP.NET Core minimal APIs as the default approach for new code examples unless the user specifically asks for controllers
- Prefer EF Core for data access examples; mention Dapper only when raw SQL performance is the explicit concern
- When reviewing code, check for the common pitfalls documented in Section 8 of README.md (async void, DbContext lifetime, N+1 queries, blocking on async)
- Suggest `dotnet test` for running tests, and xUnit as the default test framework unless the project uses another
- For NuGet packages, verify compatibility with .NET 9 before recommending
- Use the `[Entry]`, `[Mid]`, `[Senior]` badges when creating new content to indicate difficulty level
- Include Mermaid diagrams for architectural concepts
- Reference official Microsoft documentation (learn.microsoft.com) as the authoritative source

## How NOT to Help

- Do not recommend .NET Framework (the legacy Windows-only version) unless the user explicitly asks about it. Always default to .NET 9+ (the modern cross-platform runtime)
- Do not suggest third-party DI containers (Autofac, etc.) unless the user has a specific need that the built-in container cannot meet
- Do not provide code examples using the old `Startup.cs` pattern unless asked. Modern .NET uses the top-level `Program.cs` pattern
- Do not recommend the Repository pattern on top of EF Core by default. EF Core already implements Unit of Work and Repository patterns
- Do not suggest `DataTable`, `DataSet`, or ADO.NET directly unless the user explicitly needs maximum raw SQL performance
- Do not use synchronous I/O methods (`ReadAllText`, `FirstOrDefault` without `Async`, etc.) in ASP.NET Core examples
- Do not recommend Visual Studio as the only option. Mention VS Code with the C# Dev Kit and JetBrains Rider as alternatives
- Do not provide examples that mix concerns (business logic in controllers, database access in middleware, etc.)

## Key Concepts

These are the foundational ideas that should be reinforced throughout all content:

1. **The CLR runtime** -- JIT compilation, garbage collection, async state machine. Developers should understand what their code compiles to and how it runs.
2. **LINQ as a first-class tool** -- Not just for databases. LINQ is the idiomatic way to transform and query data in C#.
3. **Async/await everywhere** -- ASP.NET Core is async throughout. Synchronous I/O in request handlers causes thread pool starvation.
4. **Built-in dependency injection** -- Scoped, Singleton, Transient lifetimes and when each is appropriate. No third-party containers needed for most projects.
5. **Minimal APIs as the default** -- Less ceremony, more readable, fully featured. Controllers are valid but should not be the default recommendation.
6. **EF Core as the standard ORM** -- Code-first, migrations, LINQ-to-SQL translation. Understand tracking vs no-tracking, N+1 queries, and compiled queries.
7. **Native AOT for specific scenarios** -- Not for everything. Cold-start-sensitive serverless and edge deployments. Tradeoffs: no runtime reflection, larger binary sizes.
8. **Testing with `dotnet test`** -- xUnit for unit tests, `WebApplicationFactory` for integration tests, in-memory databases for EF Core test scenarios.

## .NET Guidelines (2026)

### Runtime and Language

- **Target framework:** .NET 9 (LTS) or .NET 10 (current) -- use `net9.0` or `net10.0` in project files
- **Language version:** C# 13 -- enabled by default when targeting .NET 9+
- **Nullable reference types:** Enabled by default. All new code should use nullable reference types (`#nullable enable`)
- **Implicit usings:** Enabled. `<ImplicitUsings>enable</ImplicitUsings>` in project file

### ASP.NET Core

- **Minimal APIs** for new projects. Use `MapGet`, `MapPost`, `MapPut`, `MapDelete` with lambda expressions or method groups
- **Top-level statements** in `Program.cs`. No `Startup.cs`, no `class Program`, no `static void Main`
- **Built-in OpenAPI support** in .NET 9+ (`Microsoft.AspNetCore.OpenApi`). No need for Swashbuckle in new projects
- **Health checks** via `builder.Services.AddHealthChecks()` and `app.MapHealthChecks("/health")`
- **Structured logging** with `ILogger<T>` injected via DI. Use message templates, not string interpolation: `logger.LogInformation("Order {OrderId} created for user {UserId}", orderId, userId)`

### Entity Framework Core

- **EF Core 9** (`Microsoft.EntityFrameworkCore` 9.x) for .NET 9 projects
- **Code-first with migrations** as the default approach
- **`AsNoTracking()`** for all read-only queries
- **`AsSplitQuery()`** when using `Include` with multiple collections to avoid Cartesian explosion
- **PostgreSQL** (`Npgsql.EntityFrameworkCore.PostgreSQL`) as the recommended database for new projects, unless the team has strong SQL Server expertise
- **Connection strings** stored in `appsettings.json` for development, environment variables or secret management for production

### Testing

- **xUnit** as the default test framework
- **`FluentAssertions`** or standard xUnit assertions based on project preference
- **`WebApplicationFactory<Program>`** for integration tests
- **`dotnet test`** to run all tests
- **Test project naming:** `{ProjectName}.Tests` or `{ProjectName}.IntegrationTests`
- Run tests before committing: `dotnet test --filter "Category!=Integration"` for unit tests only

### Tools and IDEs

- **Visual Studio 2022** (Windows/macOS) -- full-featured IDE
- **VS Code + C# Dev Kit** -- lightweight, cross-platform
- **JetBrains Rider** -- cross-platform, popular for .NET development on macOS/Linux
- **`dotnet` CLI** -- the universal tool for creating, building, testing, and publishing projects
- **NuGet** -- the package manager. `dotnet add package {PackageName}` to add dependencies

### Containerization and Deployment

- **Docker** with official .NET images: `mcr.microsoft.com/dotnet/aspnet:9.0` for runtime, `mcr.microsoft.com/dotnet/sdk:9.0` for build
- **Multi-stage Docker builds** for smaller images (build stage + runtime stage)
- **Chiseled containers** for minimal attack surface
- **Health check endpoints** (`/health`, `/health/ready`) for orchestrator integration

## Repository Structure

```
dotnet-csharp-backend/
├── README.md              # This guide -- comprehensive .NET backend fundamentals
├── AGENTS.md              # This file -- context and guidelines for contributors
└── src/                   # (Future) Sample projects and exercises
    ├── 01-minimal-api/    # Entry-level: Build your first API
    ├── 02-ef-core/        # Entry/Mid-level: Database access with EF Core
    ├── 03-auth/           # Mid-level: Authentication and authorization
    ├── 04-testing/        # Mid-level: Unit and integration testing
    └── 05-advanced/       # Senior-level: Performance, distributed systems
```

The `src/` directory is planned for future sample projects. When creating exercises, follow the numbered convention and include a README.md in each subdirectory with instructions, learning objectives, and difficulty badges.

## Making Changes

When updating this repository:

1. **README.md changes:** Ensure all code examples compile against .NET 9. Verify API references against [learn.microsoft.com](https://learn.microsoft.com/dotnet). Keep the word count in the 3000-4000 range for the main guide.
2. **New content:** Use `[Entry]`, `[Mid]`, `[Senior]` badges. Include runnable code examples. Prefer minimal APIs and modern C# features.
3. **Code quality:** Run `dotnet build` and `dotnet test` on any sample projects before committing.
4. **Links:** Prefer official Microsoft documentation links over blog posts. Ensure all URLs are valid.
5. **Style:** No emojis in content. Use tables, code blocks, and Mermaid diagrams for clarity. Keep language direct and technical.
