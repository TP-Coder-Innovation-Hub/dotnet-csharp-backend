# Authentication

`[Mid]`

## Authentication vs Authorization

- **Authentication** -- who are you? Verifying identity (login, token validation)
- **Authorization** -- what are you allowed to do? Checking permissions (role, policy)

Both are handled by middleware in ASP.NET Core.

## JWT (JSON Web Tokens)

JWT is the standard for API authentication. A JWT is a signed, encoded token that carries claims (user ID, role, expiration).

### How It Works

```text
1. Client sends credentials (email + password) to POST /auth/login
2. Server validates credentials
3. Server generates JWT and returns it
4. Client includes JWT in subsequent requests: Authorization: Bearer <token>
5. ASP.NET Core middleware validates the JWT on every request
```

### Step 1: Install Packages

```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

### Step 2: Configure Authentication

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(opt =>
    {
        opt.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]!))
        };
    });

builder.Services.AddAuthorization();

var app = builder.Build();

app.UseAuthentication(); // Who are you?
app.UseAuthorization();  // What can you do?
```

In `appsettings.json`:

```json
{
    "Jwt": {
        "Key": "your-secret-key-at-least-32-characters-long",
        "Issuer": "myapi",
        "Audience": "myapi-clients"
    }
}
```

In production, store the key in environment variables or a secret manager. Never in source control.

### Step 3: Generate Tokens

```csharp
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;
using Microsoft.IdentityModel.Tokens;

public static class TokenService
{
    public static string GenerateToken(User user, string key, string issuer, string audience)
    {
        var claims = new List<Claim>
        {
            new(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new(ClaimTypes.Email, user.Email),
            new(ClaimTypes.Role, user.Role)
        };

        var signingKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(key));
        var credentials = new SigningCredentials(signingKey, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: issuer,
            audience: audience,
            claims: claims,
            expires: DateTime.UtcNow.AddHours(1),
            signingCredentials: credentials
        );

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

### Step 4: Login Endpoint

```csharp
app.MapPost("/auth/login", async (LoginRequest req, AppDbContext db, IConfiguration config) =>
{
    var user = await db.Users.FirstOrDefaultAsync(u => u.Email == req.Email);
    if (user is null || !BCrypt.Verify(req.Password, user.PasswordHash))
        return Results.Unauthorized();

    var token = TokenService.GenerateToken(
        user,
        config["Jwt:Key"]!,
        config["Jwt:Issuer"]!,
        config["Jwt:Audience"]!);

    return Results.Ok(new { Token = token });
});

public record LoginRequest(string Email, string Password);
```

### Step 5: Protect Endpoints

```csharp
// Requires authentication
app.MapGet("/me", (HttpContext context) =>
{
    var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    var email = context.User.FindFirst(ClaimTypes.Email)?.Value;
    return Results.Ok(new { UserId = userId, Email = email });
}).RequireAuthorization();

// Requires specific role
app.MapDelete("/users/{id}", async (int id, AppDbContext db) =>
{
    var user = await db.Users.FindAsync(id);
    if (user is null) return Results.NotFound();
    db.Users.Remove(user);
    await db.SaveChangesAsync();
    return Results.NoContent();
}).RequireAuthorization("Admin");
```

### Step 6: Define Policies

```csharp
builder.Services.AddAuthorizationBuilder()
    .AddPolicy("Admin", policy => policy.RequireRole("Admin"))
    .AddPolicy("ManagerOrAbove", policy => policy.RequireRole("Manager", "Admin"));
```

## ASP.NET Core Identity

For full user management (registration, login, password reset, two-factor auth, external logins), use ASP.NET Core Identity:

```csharp
builder.Services.AddIdentityCore<User>(opt =>
{
    opt.Password.RequireDigit = true;
    opt.Password.RequiredLength = 8;
    opt.User.RequireUniqueEmail = true;
})
.AddEntityFrameworkStores<AppDbContext>();
```

Identity handles:

- Password hashing and verification
- Account lockout after failed attempts
- Email confirmation
- Password reset tokens
- External login providers (Google, Microsoft, etc.)

Use Identity when you need user accounts with passwords. Use raw JWT when you have an external identity provider (Auth0, Entra ID, Cognito).

## Security Principles

1. **Never store passwords in plaintext.** Use bcrypt or Identity's built-in hashing.
2. **Use HTTPS everywhere.** Tokens over HTTP are visible to anyone on the network.
3. **Set short token expiration.** 15-60 minutes. Use refresh tokens for longer sessions.
4. **Validate tokens on every request.** The middleware does this automatically.
5. **Keep signing keys secret.** Environment variables or secret managers only.
