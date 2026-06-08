# Programming Paradigms

 

## What Is a Paradigm

A programming paradigm is a way of organizing code. It defines what the fundamental building blocks are and how they relate. Most languages support multiple paradigms. C# is multi-paradigm, but its primary identity is object-oriented.

## The Major Paradigms

### Imperative Programming

Tell the computer exactly what to do, step by step. Most code you write is imperative.

```csharp
int sum = 0;
for (int i = 1; i <= 10; i++)
{
    sum += i;
}
Console.WriteLine(sum); // 55
```

### Object-Oriented Programming (OOP)

Organize code around objects that combine data and behavior. C# is OOP-first.

```csharp
public class BankAccount
{
    private decimal _balance;

    public BankAccount(decimal initialBalance) => _balance = initialBalance;

    public void Deposit(decimal amount) => _balance += amount;

    public decimal GetBalance() => _balance;
}

var account = new BankAccount(100m);
account.Deposit(50m);
Console.WriteLine(account.GetBalance()); // 150
```

The four pillars of OOP:

- **Encapsulation** -- hide internal state, expose behavior through methods
- **Abstraction** -- work with interfaces, not implementations
- **Inheritance** -- share behavior through class hierarchies
- **Polymorphism** -- treat different types uniformly through shared interfaces

### Functional Programming

Treat computation as evaluating functions. Prefer immutability. Avoid shared state.

```csharp
// Functional style: pure transformation, no mutation
int Sum(int start, int end) =>
    Enumerable.Range(start, end - start + 1).Sum();

Console.WriteLine(Sum(1, 10)); // 55
```

C# adopted many functional ideas over the years: LINQ, lambda expressions, pattern matching, records (immutable data types), and expression-bodied members.

## How C# Evolved

C# started in 2000 as a Java-like OOP language for the Windows-only .NET Framework. Each version added paradigms:

> 🖼️ **[IMAGE_PLACEHOLDER]** — C# paradigm evolution timeline OOP functional async

| Version | Year | Key Addition | Paradigm Influence |
|---------|------|--------------|-------------------|
| C# 1.0 | 2002 | Classes, interfaces, delegates | Pure OOP |
| C# 2.0 | 2005 | Generics | Stronger typing |
| C# 3.0 | 2007 | LINQ, lambda expressions | Functional |
| C# 5.0 | 2012 | async/await | Asynchronous |
| C# 7.0 | 2017 | Pattern matching, tuples | Functional |
| C# 8.0 | 2019 | Nullable reference types | Safety |
| C# 9.0 | 2020 | Records, primary constructors | Functional, immutability |
| C# 11+ | 2022+ | List patterns, required members | Pattern matching, safety |

The result: modern C# is not purely OOP or purely functional. It is pragmatic. You write classes for structure and services, records for data, LINQ for queries, and async/await for concurrency. You pick the tool that fits the problem.

## Why Multi-Paradigm Matters

A backend service needs OOP for organizing domain logic and service layers. It needs functional patterns for transforming data (LINQ queries, projection). It needs async for I/O. C# gives you all of these in one language without forcing a single approach.

The practical guideline:

- Use **OOP** for services, controllers, entities with behavior
- Use **functional style** for data transformations, queries, pure logic
- Use **async** for all I/O operations
- Use **records** for DTOs, commands, events -- immutable data carriers
