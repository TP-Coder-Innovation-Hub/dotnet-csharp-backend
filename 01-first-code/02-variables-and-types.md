# Variables and Types



## Strong Typing

C# is strongly typed. Every variable has a type known at compile time. The compiler enforces type correctness before the code runs.

```csharp
int count = 5;
string name = "Alice";
bool active = true;
decimal price = 19.99m;
```

The `m` suffix marks a decimal literal. Use `decimal` for money. Use `double` for scientific calculations. Use `float` rarely.

## The var Keyword

`var` lets the compiler infer the type. The variable is still strongly typed -- `var` is not `dynamic`.

```csharp
var count = 5;          // int
var name = "Alice";     // string
var price = 19.99m;     // decimal
var users = new List<User>(); // List<User>
```

Use `var` when the type is obvious from the right side. Use explicit types when the type is not clear or when you want to emphasize it.

## Value Types vs Reference Types

This is the most important distinction in C#.

### Value Types

Store data directly. Copied by value.

```csharp
int a = 10;
int b = a;  // b gets a copy of the value
b = 20;
Console.WriteLine(a); // 10 -- a is unchanged
```

Value types: `int`, `long`, `double`, `decimal`, `bool`, `char`, `DateTime`, `struct`, `enum`, `record struct`.

### Reference Types

Store a reference (pointer) to data on the heap. Copied by reference.

```csharp
var list1 = new List<int> { 1, 2, 3 };
var list2 = list1;  // list2 points to the same list
list2.Add(4);
Console.WriteLine(list1.Count); // 4 -- list1 sees the change
```

Reference types: `string`, `class`, `record`, `object`, arrays, `List<T>`, all custom classes.

### The Practical Difference

```csharp
struct Point  // value type
{
    public int X, Y;
}

class Person  // reference type
{
    public string Name;
}

var p1 = new Point { X = 1, Y = 2 };
var p2 = p1;        // copy
p2.X = 99;
Console.WriteLine(p1.X); // 1

var person1 = new Person { Name = "Alice" };
var person2 = person1;  // reference copy
person2.Name = "Bob";
Console.WriteLine(person1.Name); // Bob
```

Use `struct` for small, immutable data (points, coordinates, money). Use `class` for everything else.

## Common Types

| Type | Description | Example |
|------|-------------|---------|
| `int` | 32-bit integer | `int count = 42;` |
| `long` | 64-bit integer | `long id = 9999999999L;` |
| `double` | 64-bit floating point | `double ratio = 3.14;` |
| `decimal` | 128-bit (money) | `decimal price = 9.99m;` |
| `bool` | true/false | `bool active = true;` |
| `string` | Unicode text | `string name = "Alice";` |
| `DateTime` | Date and time | `DateTime.UtcNow` |
| `Guid` | Globally unique ID | `Guid.NewGuid()` |
| `List<T>` | Dynamic collection | `new List<int> { 1, 2, 3 }` |
| `Dictionary<K,V>` | Key-value map | `new Dictionary<string, int>()` |

## Nullable Reference Types

Since C# 8, reference types are non-nullable by default. The compiler warns you if you might access `null`.

```csharp
string name = null;    // Warning: possible null
string? name = null;   // OK: explicitly nullable

int? age = null;       // Nullable value type (Nullable<int>)
```

Enable nullable reference types in all new projects. It prevents `NullReferenceException` at compile time.
