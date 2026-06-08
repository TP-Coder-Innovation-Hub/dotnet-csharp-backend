# Error Handling



## try / catch / finally

```csharp
try
{
    var order = await dbContext.Orders.FindAsync(orderId);
    if (order is null)
        throw new NotFoundException($"Order {orderId} not found");

    order.Status = OrderStatus.Processing;
    await dbContext.SaveChangesAsync();
}
catch (NotFoundException ex)
{
    logger.LogWarning(ex, "Order not found: {OrderId}", orderId);
    return Results.NotFound(new { Error = ex.Message });
}
catch (DbUpdateException ex)
{
    logger.LogError(ex, "Database error updating order {OrderId}", orderId);
    return Results.Problem("A database error occurred");
}
finally
{
    // Always runs -- use for cleanup
    logger.LogInformation("Order processing attempt completed for {OrderId}", orderId);
}
```

- **try** -- code that might throw
- **catch** -- handles specific exception types
- **finally** -- cleanup that runs regardless of success or failure

## Exception Types

Use specific exception types. Do not catch `Exception` broadly in production code.

```csharp
// Built-in exceptions you will use
throw new ArgumentException("Invalid email format");       // Bad input
throw new InvalidOperationException("Order already shipped"); // Invalid state
throw new KeyNotFoundException($"User {id} not found");   // Missing entity
throw new UnauthorizedAccessException("Invalid credentials"); // Auth failure
```

## Custom Exceptions

Define domain-specific exceptions for your application:

```csharp
public class NotFoundException : Exception
{
    public NotFoundException(string message) : base(message) { }
}

public class BusinessException : Exception
{
    public string Code { get; }
    public BusinessException(string code, string message) : base(message)
    {
        Code = code;
    }
}

public class PaymentDeclinedException : BusinessException
{
    public decimal Amount { get; }
    public PaymentDeclinedException(decimal amount, string reason)
        : base("PAYMENT_DECLINED", $"Payment of {amount:C} declined: {reason}")
    {
        Amount = amount;
    }
}
```

Usage:

```csharp
public async Task ChargeAsync(decimal amount, string cardToken)
{
    if (amount <= 0)
        throw new ArgumentException("Amount must be positive");

    var result = await _gateway.ProcessAsync(amount, cardToken);
    if (!result.Success)
        throw new PaymentDeclinedException(amount, result.DeclineReason);
}
```

## Exception Filters

C# supports exception filters -- catch only when a condition is true:

```csharp
try
{
    await ProcessPaymentAsync(order);
}
catch (PaymentDeclinedException ex) when (ex.Amount < 10)
{
    logger.LogWarning("Small payment declined: {Amount}", ex.Amount);
    return Results.BadRequest(new { Error = ex.Message });
}
catch (PaymentDeclinedException ex)
{
    logger.LogError("Large payment declined: {Amount}", ex.Amount);
    return Results.StatusCode(502);
}
```

## Global Exception Handling in ASP.NET Core

Use middleware to catch unhandled exceptions consistently:

```csharp
app.UseExceptionHandler(appBuilder =>
{
    appBuilder.Run(async context =>
    {
        var exception = context.Features.Get<IExceptionHandlerFeature>()?.Error;

        context.Response.ContentType = "application/json";

        var (statusCode, message) = exception switch
        {
            NotFoundException _ => (StatusCodes.Status404NotFound, exception.Message),
            BusinessException _ => (StatusCodes.Status400BadRequest, exception.Message),
            UnauthorizedAccessException _ => (StatusCodes.Status401Unauthorized, "Unauthorized"),
            _ => (StatusCodes.Status500InternalServerError, "An unexpected error occurred")
        };

        context.Response.StatusCode = statusCode;
        await context.Response.WriteAsJsonAsync(new { Error = message });
    });
});
```

This keeps error handling consistent across all endpoints without repeating try/catch in every handler.

## When to Throw vs Return Errors

- **Throw** when the caller cannot reasonably handle the error (database down, null reference, invalid state)
- **Return a result type** when failure is an expected outcome (validation, not found, business rule violation)

```csharp
// Result pattern for expected failures
public record Result<T>(T? Value, string? Error, bool IsSuccess)
{
    public static Result<T> Success(T value) => new(value, null, true);
    public static Result<T> Failure(string error) => new(default, error, false);
}

// Usage
var result = await ValidateOrderAsync(order);
if (!result.IsSuccess)
    return Results.BadRequest(new { Error = result.Error });
```

In practice, most ASP.NET Core APIs use exceptions with global handling. The Result pattern is useful for domain logic where you want explicit error handling without try/catch.
