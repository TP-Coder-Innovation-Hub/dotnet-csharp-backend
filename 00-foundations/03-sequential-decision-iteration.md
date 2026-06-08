# Sequential, Decision, Iteration

`[Entry]`

## The Three Structures

Every program, in every language, is built from exactly three control structures:

1. **Sequence** -- execute statements one after another
2. **Selection (Decision)** -- choose a path based on a condition
3. **Iteration (Repetition)** -- repeat a block of code

These three are sufficient to express any computation. Functions, classes, frameworks -- they all compile down to these primitives. Understanding them is non-negotiable.

## Sequence

Statements execute top to bottom. Each line completes before the next begins.

```csharp
decimal price = 29.99m;
int quantity = 3;
decimal subtotal = price * quantity;
decimal tax = subtotal * 0.08m;
decimal total = subtotal + tax;
Console.WriteLine($"Total: {total:C}");
```

The order matters. `tax` cannot be computed before `subtotal`. The CPU does not reorder or optimize your logic.

## Selection (Decision)

### if / else

```csharp
int age = 17;

if (age >= 18)
{
    Console.WriteLine("Adult");
}
else if (age >= 13)
{
    Console.WriteLine("Teenager");
}
else
{
    Console.WriteLine("Child");
}
```

### switch (Modern C# Pattern Matching)

```csharp
string Classify(int score) => score switch
{
    >= 90 => "A",
    >= 80 => "B",
    >= 70 => "C",
    >= 60 => "D",
    _ => "F"
};

Console.WriteLine(Classify(85)); // B
```

The `_` pattern is the default case. This switch expression syntax (C# 8+) is preferred over the old switch statement for simple value selection.

## Iteration

### for -- known number of repetitions

```csharp
for (int i = 0; i < 5; i++)
{
    Console.WriteLine($"Iteration {i}");
}
```

### foreach -- iterate over a collection

```csharp
string[] languages = ["C#", "Java", "Go", "Python"];

foreach (var lang in languages)
{
    Console.WriteLine(lang);
}
```

The `["C#", "Java", ...]` syntax is a C# 12 collection expression. It works for arrays, lists, and spans.

### while -- repeat until condition is false

```csharp
int countdown = 3;
while (countdown > 0)
{
    Console.WriteLine(countdown);
    countdown--;
}
Console.WriteLine("Go");
```

## Why These Three Are Enough

A theorem from computer science (the Bohn-Jacopini structured program theorem) proves that any algorithm can be expressed using only sequence, selection, and iteration. No `goto`. No special constructs. Just these three.

Every higher-level construct is syntactic sugar:

- **Functions** are named sequences that you can invoke from other sequences
- **Classes** organize functions and data together
- **LINQ** is iteration with selection baked in
- **async/await** is iteration over a state machine

When you debug, you trace sequence, decision, and iteration. When you design, you compose them. When code is confusing, it is almost always because one of these three is unclear -- usually a decision with too many branches or an iteration with unclear termination.
