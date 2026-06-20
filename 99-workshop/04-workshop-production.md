# Workshop 4: Payslip PDF Service

> **Scenario:** The payroll API works, but generating PDF payslips takes too long to do synchronously. The HR team wants payslips generated automatically at the end of each month and downloadable anytime. Build a background service that handles this.

## What You'll Build

A `BackgroundService` that runs on a schedule, generates PDF payslips for all active employees, stores them on disk, and exposes a health check endpoint. The whole service runs in Docker.

## Skills Proved

- `BackgroundService` with `PeriodicTimer`
- Dependency injection in hosted services (scoped services)
- Health checks (`AddHealthChecks`, `MapHealthChecks`)
- Docker multi-stage build
- xUnit testing for background services

## Requirements

1. Create a `PayslipGeneratorService : BackgroundService` that:
   - Runs on the 1st of each month at 2:00 AM (simulated with a configurable interval for testing)
   - Queries all active employees from the database
   - Calculates monthly payroll (using Workshop 1 tax logic)
   - Generates a PDF payslip for each employee
   - Saves to `./payslips/{year}/{month}/{employeeId}.pdf`
   - Logs progress with structured logging (`ILogger<T>`)

2. PDF generation — use QuestPDF (MIT license, no AGPL):
   ```bash
   dotnet add package QuestPDF
   ```
   Include: employee name, department, gross salary, tax, SSO, provident fund, net pay, pay period

3. Add health checks:
   ```csharp
   builder.Services.AddHealthChecks()
       .AddDbContextCheck<AppDbContext>();
   app.MapHealthChecks("/health");
   ```

4. Dockerize with a multi-stage Dockerfile:
   - Build stage: `mcr.microsoft.com/dotnet/sdk:9.0`
   - Runtime stage: `mcr.microsoft.com/dotnet/aspnet:9.0`
   - Non-root user
   - Volume mount for `/app/payslips`

5. Write at least one unit test verifying the payroll calculation logic and one integration test verifying the service starts and responds to health checks.

## Stretch Goals

- Add retry logic with exponential backoff if the database is unavailable
- Store payslip metadata (file path, generated date) in the database
- Add a `GET /api/payslips/{employeeId}/latest` endpoint that returns the most recent PDF
- Add configuration via `appsettings.json` for the generation schedule (use `IOptions<PayrollSettings>`)
- Add rate limiting to the health check endpoint (prevent abuse)

## Connection to Final Project

This background service is the `IHostedService` component of your Payroll & Benefits Management System (see ADR-004 in the final workshop spec). It demonstrates the scheduled payroll run that triggers RabbitMQ events (`payroll.calculated`, `payroll.run.completed`) in the full system.
