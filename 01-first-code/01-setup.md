# Setup

`[Entry]`

## Install the .NET SDK

Download from https://dotnet.microsoft.com/download. Use .NET 9 (LTS) or newer.

Verify the installation:

```bash
dotnet --version
# 9.0.xxx
```

## Choose an Editor

| Editor | Platform | Notes |
|--------|----------|-------|
| Visual Studio 2022 | Windows, macOS | Full-featured IDE. Best debugging experience. |
| VS Code + C# Dev Kit | All platforms | Lightweight. Free. |
| JetBrains Rider | All platforms | Cross-platform. Popular on macOS/Linux. |

All three provide IntelliSense, debugging, and terminal integration. Pick whichever fits your workflow.

## Create Your First Project

```bash
# Create a minimal web API project
dotnet new webapi -minimal -n MyFirstApi

# Navigate into the project
cd MyFirstApi

# Run it
dotnet run
```

Output:

```text
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://localhost:5000
```

Open `http://localhost:5000/swagger` in a browser. You see the Swagger UI with default endpoints.

## What Was Created

```text
MyFirstApi/
├── MyFirstApi.csproj    # Project file (dependencies, target framework)
├── Program.cs           # Entry point and endpoint definitions
├── appsettings.json     # Configuration
└── appsettings.Development.json
```

Open `Program.cs`. It looks like this:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddOpenApi();

var app = builder.Build();

app.MapGet("/weatherforecast", () =>
{
    var summaries = new[] { "Freezing", "Bracing", "Chilly" };
    return summaries.Select((s, i) => new WeatherForecast(i + 1, s, 20));
})
.WithName("GetWeatherForecast");

app.MapOpenApi();
app.Run();

record WeatherForecast(int Id, string Summary, int TemperatureC);
```

Line by line:

1. `WebApplication.CreateBuilder(args)` -- creates a web app builder with defaults (configuration, logging, DI)
2. `builder.Services.AddOpenApi()` -- registers OpenAPI/Swagger services
3. `builder.Build()` -- builds the application pipeline
4. `app.MapGet(...)` -- maps a GET endpoint to `/weatherforecast`
5. `app.Run()` -- starts the server and blocks until shutdown

## Essential CLI Commands

```bash
dotnet new console -n MyApp        # Console application
dotnet new webapi -minimal -n Api  # Minimal API
dotnet build                        # Compile without running
dotnet run                          # Build and run
dotnet test                         # Run tests
dotnet add package PackageName      # Add a NuGet package
dotnet watch                        # Hot reload during development
```

`dotnet watch` restarts the app when you save file changes. Use it during development instead of `dotnet run`.
