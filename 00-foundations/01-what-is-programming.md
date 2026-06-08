# What Is Programming



## The Core Idea

Programming is writing instructions a machine executes to transform input into output. Every program, regardless of language or scale, does three things:

1. **Takes input** -- from a user, a file, a network request, a sensor
2. **Processes it** -- applies logic: calculations, decisions, transformations
3. **Produces output** -- a result displayed, stored, sent, or returned

This is not a simplification. This is the entire discipline at its root.

## Instructions and Data

A program is two things combined:

- **Data** -- the information the program works with. Numbers, text, lists, records, files.
- **Instructions** -- the steps that manipulate data. Read, write, compare, branch, repeat.

```csharp
// Data
int price = 100;
int quantity = 3;

// Instructions
int total = price * quantity;
Console.WriteLine(total); // 300
```

The CPU executes instructions sequentially: line by line, top to bottom. Branching (if/else) and looping (for/while) are the only mechanisms that change this flow. Everything else -- functions, classes, modules, APIs -- is organization built on top of these primitives.

## Why It Matters for Backend

A backend server is a program that:

1. **Receives input** -- an HTTP request (method, URL, headers, body)
2. **Processes it** -- validates, queries a database, applies business rules
3. **Returns output** -- an HTTP response (status code, headers, body)

```text
Client --HTTP Request--> Your Code --> Database
Client <--HTTP Response-- Your Code <-- Database
```

Every backend framework in every language wraps this cycle. ASP.NET Core, Express, Spring Boot, Flask -- all of them abstract the same input-process-output pattern around HTTP.

## Abstractions Layer On

Raw instructions and data are enough to write any program. But humans cannot manage millions of raw instructions. So we build abstractions:

- **Variables** name data so we can refer to it
- **Functions** group instructions so we can reuse them
- **Classes** bundle data and functions together
- **Modules** organize classes into cohesive units
- **Frameworks** provide patterns for entire application types

Each layer makes the code more manageable. Each layer also adds concepts you must learn. The skill is knowing when an abstraction helps and when it obscures.

## The Mental Model

Think of programming as recipe writing for a very literal chef:

- The chef follows steps in exact order
- The chef does not infer or assume anything
- Every detail must be explicit
- Mistakes in the recipe produce wrong results, not corrected results

This is why precision matters. This is why testing matters. This is why type systems (like C#'s) exist: they catch mistakes before the chef starts cooking.
