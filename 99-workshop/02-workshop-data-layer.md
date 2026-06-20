# Workshop 2: Employee Records Database

> **Scenario:** Your company's HR team manages employee data in spreadsheets. It's error-prone and hard to query. Build a database-backed tool that stores departments and employees, with proper relationships and data integrity.

## What You'll Build

A .NET console application with Entity Framework Core that manages employee records — create, read, update, and delete (CRUD) operations against a PostgreSQL database.

## Skills Proved

- Entity Framework Core code-first approach
- Database migrations (`dotnet ef`)
- LINQ queries (filtering, joining, projecting)
- Navigation properties and relationships

## Requirements

1. Create two entities with a one-to-many relationship:

   ```
   Department (1) ─── (*) Employee
   - Id (int, PK)         - Id (int, PK)
   - Name (string)        - FirstName (string)
   - Code (string)        - LastName (string)
                         - Email (string)
                         - Salary (decimal)
                         - DepartmentId (int, FK)
                         - HireDate (DateTime)
   ```

2. Configure EF Core with PostgreSQL provider
3. Create and apply a migration: `dotnet ef migrations add InitialCreate && dotnet ef database update`
4. Implement CRUD operations via a CLI menu:

   ```
   === Employee Records ===
   1. List all employees (with department name)
   2. Add employee
   3. Update employee salary
   4. Delete employee
   5. List employees by department
   6. Exit
   ```

5. Use `AsNoTracking()` for read-only queries
6. Validate email format before saving — reject invalid emails with a clear error

## Stretch Goals

- Add a search function: find employees by name (partial match, case-insensitive)
- Add pagination to the employee list (10 per page)
- Seed the database with sample departments (Engineering, HR, Finance, Marketing) and 5 employees
- Add a `--export` option that writes all employees to a CSV file

## Connection to Final Project

This database schema and CRUD foundation becomes the Employee Service in your Payroll & Benefits Management System. The `Employee` entity and `Department` relationship carry directly into the final system.
