# Deployment



## Docker

Docker is the standard way to deploy .NET applications. Multi-stage builds produce small images.

### Dockerfile

```dockerfile
# Build stage
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /src

COPY ProductApi.csproj .
RUN dotnet restore

COPY . .
RUN dotnet publish -c Release -o /app/publish

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS runtime
WORKDIR /app

COPY --from=build /app/publish .

# Non-root user for security
USER $APP_UID
EXPOSE 8080

ENTRYPOINT ["dotnet", "ProductApi.dll"]
```

Step by step:

1. **Build stage** -- uses the full SDK image. Restores packages, compiles, publishes.
2. **Runtime stage** -- uses the smaller ASP.NET runtime image. Copies only the published output. No SDK, no source code.
3. **`USER $APP_UID`** -- runs as a non-root user. Security best practice.
4. **`EXPOSE 8080`** -- documents the port. ASP.NET Core defaults to 8080 in containers.

### Build and Run

```bash
# Build the image
docker build -t product-api .

# Run the container
docker run -d -p 8080:8080 \
  -e ConnectionStrings__Default="Host=host.docker.internal;Port=5432;Database=products;Username=postgres;Password=secret" \
  product-api
```

Override configuration with environment variables. `ConnectionStrings__Default` (double underscore) maps to `ConnectionStrings:Default` in `appsettings.json`.

### Chiseled Containers

For minimal attack surface, use chiseled images:

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:9.0-chiseled AS runtime
```

Chiseled images contain only the .NET runtime and its dependencies. No shell, no package manager, no unnecessary OS components. Image size drops from ~200MB to ~80MB.

## Native AOT

Compile your entire application to a native binary at build time. No JIT at runtime.

### When to Use

- Serverless functions with cold-start requirements
- CLI tools that need instant startup
- Edge deployments with constrained resources

### When Not to Use

- APIs that use reflection-heavy libraries
- Applications needing runtime code generation
- Projects requiring EF Core (limited AOT compatibility)

### Publish

```bash
dotnet publish -c Release -r linux-x64 -p:PublishAot=true
```

Output: a single native binary. No .NET runtime needed on the target machine.

```bash
# Check the binary
./ProductApi
# Starts instantly. No JIT warmup.
```

Tradeoffs:

| Factor | JIT (default) | Native AOT |
|--------|---------------|------------|
| Startup time | ~100-500ms | ~10-50ms |
| Binary size | ~80MB (with runtime) | ~30-80MB (self-contained) |
| Peak throughput | Higher (tiered JIT optimizes hot paths) | Slightly lower |
| Reflection | Full support | Limited |
| EF Core | Full support | Partial |
| Build time | Fast | Slower (full AOT compilation) |

## Health Checks

Add health check endpoints for orchestrators (Kubernetes, ECS) and monitoring:

```csharp
builder.Services.AddHealthChecks()
    .AddNpgSql(builder.Configuration.GetConnectionString("Default")!)
    .AddRedis(builder.Configuration.GetConnectionString("Redis")!);

app.MapHealthChecks("/health");        // Basic liveness
app.MapHealthChecks("/health/ready");  // Readiness with dependencies
```

- `/health` -- is the app running? (liveness probe)
- `/health/ready` -- is the app ready to serve traffic? Checks database, Redis, etc. (readiness probe)

Kubernetes example:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
readinessProbe:
  httpGet:
    path: /health/ready
    port: 8080
```

## Environment-Specific Configuration

```text
appsettings.json              -- Base configuration
appsettings.Development.json  -- Dev overrides (detailed errors, debug logging)
appsettings.Production.json   -- Prod overrides (minimal logging, HTTPS)
```

Environment variables override everything:

```bash
ASPNETCORE_ENVIRONMENT=Production
ASPNETCORE_URLS=http://0.0.0.0:8080
ConnectionStrings__Default=Host=prod-db;...
```

Never put secrets in `appsettings.json`. Use environment variables, Azure Key Vault, or AWS Secrets Manager.

## CI/CD Checklist

1. Run `dotnet test` in CI. Fail the build on test failures.
2. Run `dotnet build --warnaserror` to treat warnings as errors.
3. Publish Docker image to a registry (ACR, ECR, Docker Hub).
4. Apply EF Core migrations as a pre-deployment step.
5. Deploy with rolling updates. Health checks gate traffic.
6. Monitor logs and metrics post-deployment.
