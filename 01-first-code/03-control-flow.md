# Control Flow



## if / else if / else

The basic decision construct. Execute a block when a condition is true.

```csharp
int score = 85;

if (score >= 90)
{
    Console.WriteLine("A");
}
else if (score >= 80)
{
    Console.WriteLine("B");
}
else if (score >= 70)
{
    Console.WriteLine("C");
}
else
{
    Console.WriteLine("F");
}
```

Single-statement bodies do not need braces, but always use them for readability:

```csharp
if (order.Status == OrderStatus.Cancelled)
    return; // works but prefer braces
```

## switch Statement (Classic)

Match a value against multiple cases.

```csharp
string GetDayType(DayOfWeek day)
{
    switch (day)
    {
        case DayOfWeek.Saturday:
        case DayOfWeek.Sunday:
            return "Weekend";
        case DayOfWeek.Friday:
            return "Almost weekend";
        default:
            return "Weekday";
    }
}
```

## switch Expression (Modern C# -- Preferred)

C# 8 introduced switch expressions. More concise, expression-based, pattern-matching capable.

```csharp
string GetDayType(DayOfWeek day) => day switch
{
    DayOfWeek.Saturday or DayOfWeek.Sunday => "Weekend",
    DayOfWeek.Friday => "Almost weekend",
    _ => "Weekday"
};
```

Pattern matching with conditions:

```csharp
string ClassifyOrder(Order order) => order.Status switch
{
    OrderStatus.Pending when order.CreatedAt < DateTime.UtcNow.AddHours(-24) => "Stale",
    OrderStatus.Pending => "New",
    OrderStatus.Shipped => "In transit",
    OrderStatus.Delivered => "Complete",
    _ => "Unknown"
};
```

The `when` clause adds a condition to a pattern match.

## for Loop

Use when you know the number of iterations.

```csharp
for (int i = 0; i < 10; i++)
{
    Console.WriteLine($"Item {i}");
}
```

## foreach Loop

Use to iterate over collections. The most common loop in C#.

```csharp
var products = new List<string> { "Laptop", "Phone", "Tablet" };

foreach (var product in products)
{
    Console.WriteLine(product);
}
```

In production code, you often replace `foreach` with LINQ:

```csharp
products.ForEach(Console.WriteLine);
// Or in a pipeline:
var upperNames = products.Select(p => p.ToUpper()).ToList();
```

## while and do-while

Use when the number of iterations is unknown.

```csharp
// while -- condition checked before each iteration
int retries = 0;
while (!connected && retries < 3)
{
    Connect();
    retries++;
}

// do-while -- body executes at least once
string input;
do
{
    input = Console.ReadLine();
} while (string.IsNullOrEmpty(input));
```

## break and continue

```csharp
for (int i = 0; i < 10; i++)
{
    if (i == 3) continue; // skip iteration 3
    if (i == 7) break;    // stop the loop entirely
    Console.WriteLine(i); // 0, 1, 2, 4, 5, 6
}
```

Use sparingly. Overuse of `break` and `continue` makes loops hard to follow.

## Ternary Operator

Inline conditional. Use for simple value selection.

```csharp
string label = isActive ? "Active" : "Inactive";
decimal discount = isMember ? 0.10m : 0m;
```

For anything more complex, use a switch expression or if/else.
