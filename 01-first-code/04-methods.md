# Methods



## Basic Syntax

A method is a named block of code that performs a task. It can accept input (parameters) and return output.

```csharp
int Add(int a, int b)
{
    return a + b;
}

int result = Add(3, 4); // 7
```

C# methods are verbose compared to Python or JavaScript, but that verbosity makes intent explicit. Every parameter has a type. The return type is declared. The compiler catches mismatches.

## Expression-Bodied Members

When a method body is a single expression, use `=>` syntax:

```csharp
int Add(int a, int b) => a + b;

string Greet(string name) => $"Hello, {name}";

bool IsAdult(int age) => age >= 18;
```

This is not a different feature. It is the same method, written more concisely.

## Parameters

### Optional Parameters

```csharp
void CreateUser(string name, string role = "User", bool active = true)
{
    Console.WriteLine($"{name}, {role}, active={active}");
}

CreateUser("Alice");                     // Alice, User, active=True
CreateUser("Bob", "Admin");              // Bob, Admin, active=True
CreateUser("Eve", "Admin", false);       // Eve, Admin, active=False
```

### Named Arguments

```csharp
CreateUser(name: "Alice", active: false); // skip role, set active
```

### out Parameters

Return multiple values through parameters:

```csharp
bool TryParseInt(string input, out int result)
{
    return int.TryParse(input, out result);
}

if (TryParseInt("42", out int value))
{
    Console.WriteLine(value); // 42
}
```

### ref Parameters

Pass by reference. Changes inside the method affect the caller:

```csharp
void Double(ref int number)
{
    number *= 2;
}

int x = 5;
Double(ref x);
Console.WriteLine(x); // 10
```

Use `ref` sparingly. It makes code harder to reason about because the caller's variables change invisibly.

### params

Accept a variable number of arguments:

```csharp
int Sum(params int[] numbers)
{
    return numbers.Sum();
}

Console.WriteLine(Sum(1, 2, 3, 4, 5)); // 15
```

## Method Overloading

Multiple methods with the same name but different parameters:

```csharp
string Format(decimal amount) => $"{amount:C}";
string Format(decimal amount, string currency) => $"{currency}{amount:N2}";
```

The compiler picks the correct overload based on the arguments you pass.

## Static vs Instance Methods

```csharp
class MathHelper
{
    // Static -- called on the type, not an instance
    public static double Square(double x) => x * x;
}

// Usage:
double area = MathHelper.Square(5.0); // 25
```

```csharp
class Order
{
    public decimal Total { get; set; }

    // Instance -- operates on a specific object
    public decimal ApplyDiscount(decimal percent) => Total * (1 - percent);
}

var order = new Order { Total = 100m };
var discounted = order.ApplyDiscount(0.10m); // 90
```

## C# Methods Are Verbose but Clear

Compare:

```python
# Python
def add(a, b):
    return a + b
```

```csharp
// C#
int Add(int a, int b) => a + b;
```

C# requires you to declare types. This is intentional. The compiler catches type errors at build time, not runtime. In a backend system handling thousands of requests, catching errors before deployment is valuable.
