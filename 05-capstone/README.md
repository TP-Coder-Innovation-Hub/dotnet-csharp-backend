# Capstone Project: Product Order Management API

`[Mid]`

## Objective

Build a production-ready REST API for managing products and orders. Apply everything from this learning path: minimal APIs, EF Core, authentication, testing, logging, and Docker deployment.

## Requirements

### Data Model

```csharp
public class User
{
    public int Id { get; set; }
    public string Email { get; set; } = "";
    public string PasswordHash { get; set; } = "";
    public string Role { get; set; } = "Customer"; // Customer, Manager, Admin
    public DateTime CreatedAt { get; set; }
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Category { get; set; } = "";
    public decimal Price { get; set; }
    public int StockQuantity { get; set; }
    public bool IsActive { get; set; } = true;
    public DateTime CreatedAt { get; set; }
}

public class Order
{
    public int Id { get; set; }
    public int UserId { get; set; }
    public User User { get; set; } = null!;
    public decimal Total { get; set; }
    public OrderStatus Status { get; set; }
    public DateTime CreatedAt { get; set; }
    public List<OrderLine> Lines { get; set; } = [];
}

public class OrderLine
{
    public int Id { get; set; }
    public int OrderId { get; set; }
    public int ProductId { get; set; }
    public Product Product { get; set; } = null!;
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
}

public enum OrderStatus { Pending, Processing, Shipped, Delivered, Cancelled }
```

### Endpoints

| Method | URL | Auth | Description |
|--------|-----|------|-------------|
| POST | /auth/register | None | Register a new user |
| POST | /auth/login | None | Login, return JWT |
| GET | /products | None | List active products (paginated) |
| GET | /products/{id} | None | Get product details |
| POST | /products | Admin | Create a product |
| PUT | /products/{id} | Admin | Update a product |
| DELETE | /products/{id} | Admin | Soft-delete a product |
| POST | /orders | Authenticated | Place an order |
| GET | /orders | Authenticated | List own orders |
| GET | /orders/{id} | Authenticated | Get order details (own only, or any for Admin) |
| PATCH | /orders/{id}/status | Manager, Admin | Update order status |

### Business Rules

1. Registration requires unique email. Password must be at least 8 characters.
2. Product price must be greater than 0. Stock quantity must be non-negative.
3. When placing an order, verify sufficient stock for each product. Deduct stock atomically.
4. Order total is calculated as sum of (quantity * unit price) for each line.
5. Users can only view their own orders. Admins can view all orders.
6. Only Manager or Admin can update order status.
7. Pending orders can be cancelled by the owner. Shipped/Delivered orders cannot.
8. All operations return appropriate HTTP status codes and consistent error shapes.

### Non-Functional Requirements

1. All database queries use `AsNoTracking()` for reads
2. All I/O operations are async
3. Structured logging with Serilog for all operations
4. Health check endpoint at `/health` (checks database connectivity)
5. Pagination for product and order lists (default 20 per page)
6. Global exception handling middleware returning consistent JSON errors
7. Docker deployment with multi-stage build
8. Unit tests for business logic. Integration tests for API endpoints.

## Architecture

```text
ProductApi/
├── Program.cs                 # App setup, middleware, endpoint definitions
├── Data/
│   └── AppDbContext.cs        # EF Core context, entity configuration
├── Models/
│   ├── Product.cs
│   ├── Order.cs
│   ├── OrderLine.cs
│   └── User.cs
├── Requests/
│   ├── CreateProductRequest.cs
│   ├── PlaceOrderRequest.cs
│   └── LoginRequest.cs
├── Responses/
│   ├── ProductResponse.cs
│   ├── OrderResponse.cs
│   └── TokenResponse.cs
├── Services/
│   ├── IOrderService.cs
│   ├── OrderService.cs
│   ├── IAuthService.cs
│   └── AuthService.cs
├── Middleware/
│   └── ExceptionHandler.cs
├── Migrations/
├── Dockerfile
├── ProductApi.csproj
└── appsettings.json

ProductApi.Tests/
├── Services/
│   └── OrderServiceTests.cs   # Unit tests
└── Integration/
    └── OrderApiTests.cs       # Integration tests with WebApplicationFactory
```

## Success Criteria

Your implementation is complete when:

1. `dotnet build` succeeds with no warnings
2. `dotnet test` passes all tests (minimum 10: 6 unit, 4 integration)
3. `docker build -t product-api .` produces a working image
4. All endpoints return correct status codes
5. Authentication and authorization work correctly
6. Structured logs include request ID, user ID, and relevant entity IDs
7. The API handles edge cases: out-of-stock, duplicate email, invalid status transitions
