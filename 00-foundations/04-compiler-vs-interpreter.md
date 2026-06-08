# Compiler vs Interpreter



## The Two Approaches

Programming languages are designed for humans. CPUs execute machine code. Something must translate between the two. There are two fundamental approaches:

- **Compiler** -- translates the entire program to machine code before execution
- **Interpreter** -- translates and executes one instruction at a time during execution

C# uses neither approach directly. It uses a hybrid: compile to intermediate language, then JIT-compile to machine code at runtime.

## Compiled Languages

Source code is translated to native machine code by a compiler (like `gcc` for C, `rustc` for Rust). The output is a binary the CPU executes directly.

```text
Source Code --> Compiler --> Native Binary --> CPU executes
```

Pros: fast execution, optimizations applied to entire program, no runtime translation overhead.
Cons: platform-specific (a binary for x86 Linux will not run on ARM macOS), longer build times.

## Interpreted Languages

Source code is read and executed line by line by an interpreter program. Python, Ruby, and JavaScript (traditionally) work this way.

```text
Source Code --> Interpreter reads and executes each line
```

Pros: no compile step, cross-platform (interpreter handles differences), faster development cycle.
Cons: slower execution, errors surface at runtime, no whole-program optimization.

## C#'s Hybrid Approach: IL + JIT

C# compiles to Intermediate Language (IL), not machine code. The Common Language Runtime (CLR) JIT-compiles IL to native code when the program runs.

```text
C# Source Code
      |
      v
  Roslyn Compiler
      |
      v
  IL (Intermediate Language) in DLL/EXE
      |
      v
  CLR (Common Language Runtime)
      |-- JIT Compiler (RyuJIT) --> Native Machine Code
      |-- Garbage Collector
      |-- Thread Pool
      |
      v
  Operating System (Linux, macOS, Windows)
```

Step by step:

1. You write C# code and run `dotnet build`
2. The Roslyn compiler translates C# to IL and packages it into a DLL
3. When you run the app (`dotnet run`), the CLR loads the DLL
4. The JIT compiler (RyuJIT) translates IL methods to native machine code as they are called
5. Subsequent calls to the same method execute the cached native code directly

This is why C# is cross-platform. The IL is platform-agnostic. The CLR (specifically the JIT) handles platform differences. The same DLL runs on Linux, macOS, and Windows.

## Tiered Compilation

Modern .NET uses tiered JIT compilation:

- **Tier 0** -- when a method is first called, JIT compiles it quickly with minimal optimization. Fast startup.
- **Tier 1** -- if a method is called frequently (a "hot" method), the JIT recompiles it with aggressive optimizations. Better throughput.

This gives you fast startup without sacrificing peak performance.

## Native AOT

.NET 9+ supports Native AOT (Ahead-of-Time) compilation. The entire application is compiled to a native binary at build time, with no JIT at runtime:

```bash
dotnet publish -c Release -r linux-x64 -p:PublishAot=true
```

This produces a single native binary. Fast startup, no runtime compilation, lower memory usage. Tradeoff: no runtime reflection, larger binary, some libraries are incompatible.

Use Native AOT for serverless functions, CLI tools, and edge deployments where cold start matters.

## Why This Matters

Understanding the compilation model helps you:

- Debug performance issues (JIT behavior, tiered compilation)
- Choose deployment strategies (JIT vs Native AOT)
- Understand why the same DLL runs everywhere (IL is platform-neutral)
- Reason about startup time vs throughput tradeoffs
