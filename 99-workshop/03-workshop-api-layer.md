# Workshop 3: Payroll API

> **Scenario:** The HR team wants a web interface for payroll management instead of the CLI tool. Build a REST API that a React frontend can consume. Different roles need different access levels.

## What You'll Build

An ASP.NET Core minimal API with JWT authentication and role-based authorization. Admins can manage all employees, HR can run payroll, and employees can view their own payslips.

## Skills Proved

- REST API design with minimal APIs
- JWT authentication with `AddJwtBearer`
- Role-based authorization (Admin, HR, Employee)
- DTOs / request and response records
- Error handling middleware

## Requirements

1. Implement these endpoints:

   ```
   POST   /auth/login              → Returns JWT (public)
   GET    /api/employees           → List employees (Admin, HR)
   GET    /api/employees/{id}      → Get employee (Admin, HR, or self)
   POST   /api/employees           → Create employee (Admin)
   PUT    /api/employees/{id}      → Update salary (Admin, HR)
   DELETE /api/employees/{id}      → Delete employee (Admin)
   POST   /api/payroll/calculate   → Calculate payroll for employee (Admin, HR)
   GET    /api/payroll/{empId}     → View payslip (Admin, HR, or self)
   ```

2. JWT must contain claims: `sub` (employee ID), `role`, `name`
3. Use the tax calculator logic from Workshop 1 for the `/payroll/calculate` endpoint
4. Use the Employee entity from Workshop 2 for data storage
5. Return proper HTTP status codes:
   - `200 OK` — success
   - `201 Created` — resource created (with `Location` header)
   - `400 Bad Request` — validation error
   - `401 Unauthorized` — missing/invalid token
   - `403 Forbidden` — valid token, insufficient role
   - `404 Not Found` — resource doesn't exist

6. Add a global exception handler that returns a consistent JSON error format:
   ```json
   { "error": "Employee not found", "statusCode": 404 }
   ```

## Stretch Goals

- Add pagination to `GET /api/employees` (`?page=1&size=20`)
- Add input validation using data annotations or FluentValidation
- Add an OpenAPI/Swagger endpoint (`/openapi/v1.json`)
- Implement a refresh token endpoint (`POST /auth/refresh`)

## Connection to Final Project

This API becomes the Gateway entry point for the Payroll & Benefits Management System. The JWT auth, role-based access, and payroll calculation endpoints carry directly into the final 4-service architecture.
