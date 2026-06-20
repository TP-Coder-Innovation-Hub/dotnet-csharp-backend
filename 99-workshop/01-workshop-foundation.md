# Workshop 1: Thai Tax Calculator CLI

> **Scenario:** A Thai company needs a command-line tool that calculates monthly income tax, social security, and provident fund deductions for employees. Your finance team will use this to verify payroll before the full system is built.

## What You'll Build

A .NET console application that takes an employee's monthly salary and optional provident fund percentage, then outputs a formatted breakdown of all deductions and net pay.

## Skills Proved

- C# basics: variables, control flow, methods
- `decimal` arithmetic (never use `double` for money)
- Console I/O and string formatting
- Error handling (invalid input, negative salary)

## Requirements

1. Accept input via command-line args: `dotnet run --salary 50000 [--provident-fund 5]`
2. Calculate **Thai progressive income tax** using 2024 brackets:
   - 0 - 150,000 THB/year: 0%
   - 150,001 - 300,000: 5%
   - 300,001 - 500,000: 10%
   - 500,001 - 750,000: 15%
   - 750,001 - 1,000,000: 20%
   - 1,000,001 - 2,000,000: 25%
   - 2,000,001 - 5,000,000: 30%
   - Over 5,000,000: 35%
   - **Hint:** Annualize the monthly salary, calculate annual tax, divide by 12
3. Calculate **social security**: 5% of salary, capped at 750 THB/month (salary cap 15,000 THB)
4. Calculate **provident fund**: employee contribution percentage (default 0%), capped at 500,000 THB/year
5. Output a formatted breakdown:

```
=== Monthly Payroll Summary ===
Gross Salary:     50,000.00 THB
Income Tax:       3,541.67 THB
Social Security:    750.00 THB
Provident Fund:   2,500.00 THB (5%)
--------------------------------
Net Pay:          43,208.33 THB
```

6. Handle invalid input (negative salary, non-numeric args) with clear error messages and exit code 1

## Stretch Goals

- Add `--help` flag with usage instructions
- Support annual salary input mode (`--annual` flag)
- Output as JSON (`--format json`) for integration with other tools
- Read tax brackets from `appsettings.json` so they can be updated without recompiling

## Connection to Final Project

This calculator becomes the core tax computation engine in your Payroll & Benefits Management System. Getting the math right here — especially `decimal` precision — prevents expensive payroll errors in production.
