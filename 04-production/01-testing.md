# Testing



## The Testing Strategy

- **Unit tests** -- test a single class or method in isolation. Fast. No database, no HTTP.
- **Integration tests** -- test multiple components together. Uses real database (in-memory or test container). Uses the full ASP.NET Core pipeline.
- **End-to-end tests** -- test the entire system from HTTP request to database and back.

For .NET backend, xUnit is the standard test framework.

## Unit Tests with xUnit

### Setup

```bash
dotnet new xunit -n ProductApi.Tests
dotnet add ProductApi.Tests reference ProductApi
dotnet add ProductApi.Tests package FluentAssertions
```

### Testing a Service

```csharp
// The service being tested
public class PricingService
{
    public decimal CalculateDiscount(CustomerTier tier, decimal orderTotal)
    {
        return tier switch
        {
            CustomerTier.Bronze => orderTotal * 0.05m,
            CustomerTier.Silver => orderTotal * 0.10m,
            CustomerTier.Gold => orderTotal * 0.15m,
            _ => 0m
        };
    }
}

public enum CustomerTier { Bronze, Silver, Gold }
```

```csharp
// The test
public class PricingServiceTests
{
    [Fact]
    public void CalculateDiscount_GoldCustomer_Returns15Percent()
    {
        var service = new PricingService();
        var discount = service.CalculateDiscount(CustomerTier.Gold, 100m);
        discount.Should().Be(15m);
    }

    [Theory]
    [InlineData(CustomerTier.Bronze, 100, 5)]
    [InlineData(CustomerTier.Silver, 100, 10)]
    [InlineData(CustomerTier.Gold, 100, 15)]
    public void CalculateDiscount_ReturnsCorrectPercentage(
        CustomerTier tier, decimal total, decimal expected)
    {
        var service = new PricingService();
        var discount = service.CalculateDiscount(tier, total);
        discount.Should().Be(expected);
    }
}
```

- `[Fact]` -- single test with no parameters
- `[Theory]` -- parameterized test. `[InlineData]` provides the parameters.

## Testing with EF Core InMemory

```csharp
public class ProductRepositoryTests
{
    private AppDbContext CreateDbContext()
    {
        var options = new DbContextOptionsBuilder<AppDbContext>()
            .UseInMemoryDatabase(Guid.NewGuid().ToString()) // Unique DB per test
            .Options;
        return new AppDbContext(options);
    }

    [Fact]
    public async Task GetByIdAsync_ExistingProduct_ReturnsProduct()
    {
        // Arrange
        await using var db = CreateDbContext();
        db.Products.Add(new Product { Name = "Laptop", Price = 999m });
        await db.SaveChangesAsync();

        var repository = new ProductRepository(db);

        // Act
        var product = await repository.GetByIdAsync(1);

        // Assert
        product.Should().NotBeNull();
        product!.Name.Should().Be("Laptop");
    }

    [Fact]
    public async Task GetAllAsync_ReturnsOnlyActiveProducts()
    {
        await using var db = CreateDbContext();
        db.Products.AddRange(
            new Product { Name = "Active", IsActive = true },
            new Product { Name = "Inactive", IsActive = false }
        );
        await db.SaveChangesAsync();

        var repository = new ProductRepository(db);
        var products = await repository.GetActiveAsync();

        products.Should().HaveCount(1);
        products[0].Name.Should().Be("Active");
    }
}
```

Key points:

- `UseInMemoryDatabase` -- no real database. Fast. Data is isolated per test.
- `Guid.NewGuid().ToString()` -- each test gets a fresh database.
- `await using` -- ensures the DbContext is disposed after the test.

## Integration Tests with WebApplicationFactory

Test the full HTTP pipeline:

```csharp
public class ProductApiTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;

    public ProductApiTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory.WithWebHostBuilder(builder =>
        {
            builder.ConfigureServices(services =>
            {
                // Replace real database with in-memory
                var descriptor = services.Single(
                    d => d.ServiceType == typeof(DbContextOptions<AppDbContext>));
                services.Remove(descriptor);
                services.AddDbContext<AppDbContext>(opt =>
                    opt.UseInMemoryDatabase("TestDb"));
            });
        });
    }

    [Fact]
    public async Task PostProduct_Returns201()
    {
        var client = _factory.CreateClient();
        var command = new { Name = "Test Product", Category = "Test", Price = 10m };

        var response = await client.PostAsJsonAsync("/products", command);

        response.StatusCode.Should().Be(HttpStatusCode.Created);
        response.Headers.Location.Should().NotBeNull();
    }

    [Fact]
    public async Task GetProduct_ReturnsProduct()
    {
        var client = _factory.CreateClient();

        // Create a product first
        var createResponse = await client.PostAsJsonAsync("/products",
            new { Name = "Keyboard", Category = "Electronics", Price = 79m });
        var created = await createResponse.Content.ReadFromJsonAsync<ProductResponse>();

        // Get it
        var getResponse = await client.GetAsync($"/products/{created!.Id}");
        getResponse.StatusCode.Should().Be(HttpStatusCode.OK);
    }
}
```

`WebApplicationFactory<Program>` starts the full ASP.NET Core application in-memory. You get an `HttpClient` that makes real requests through the entire pipeline: routing, middleware, DI, database.

## Running Tests

```bash
# Run all tests
dotnet test

# Run with verbose output
dotnet test --verbosity normal

# Run only unit tests (exclude integration)
dotnet test --filter "FullyQualifiedName~UnitTests"

# Run a specific test
dotnet test --filter "CalculateDiscount_GoldCustomer_Returns15Percent"
```

## Test Naming Convention

```csharp
// MethodName_Scenario_ExpectedBehavior
[Fact]
public void CalculateDiscount_GoldCustomer_Returns15Percent() { }

[Fact]
public async Task GetByIdAsync_NonExistentProduct_ReturnsNull() { }

[Fact]
public async Task CreateOrder_ValidInput_SendsEmail() { }
```

This naming makes test results readable at a glance. When a test fails, the name tells you exactly what went wrong.
