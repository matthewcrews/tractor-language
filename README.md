# Tractor: A CLR Language for Numerical Computation

The original intention of .NET was to provide a runtime for many programming languages to run on with seamless interoperability. The CLR already has several excellent general-purpose languages which emphasize different paradigms (C#, F#, and VB.NET). The upside of a general-purpose language is that it gives you the tools for addressing a wide variety of problems. The downside is that, by definition, it not as efficient for domain specific problems. For example, SQL is the lingua franca language for querying data. It has an excellent set of abstractions for working with large sets of related data. HTML is a language for expressing the structure of a document.

Why not have a language that is specifically focused on expressing high-performance computation? Do not abstract away the fact that you are working with a CPU. Most languages are designed for an idealized computers and do not take into account that most CPUs are designed for working with arrays of data. Real world CPUs fetch and process data in chunks from the L1 cache all the way to pages of memory being managed by the operating system. Only at the register level does a CPU think of data in scalars.

Most modern systems, as of 2023, have virtual memory capabilities which allow us to address a 64-bit memory space. Technically this is not the case. x86-64 allows you to use memory addresses based on [48-bits](https://en.wikipedia.org/wiki/X86-64) which is still an incredibly large memory space. The operating system hides the fact that this much space does not physically exist. We can exploit this fact to simplify how memory is allocated and collected based on the ideas put forth by [Ryan Fleury](https://www.rfleury.com/p/untangling-lifetimes-the-arena-allocator).

The following is a proposal for a language targeting the CLR which allows us to express high-performance computation that would interoperate seamlessly with the general-purpose languages already targeting .NET. I am using a working name of Tractor. Most of the examples I use when teaching are based on farm animals so it felt natural to stick with that theme. This is a living document and will evolve as my learning grows.

Also, I have no intention of actually creating this language any time soon. I am most interested in starting a discussion and soliciting feedback. Perhaps it will inspire someone else 😊. Feedback and crique is welcome. The aim of the language is to be simple and allow the user to write code that has [mechanical sympathy](https://dzone.com/articles/mechanical-sympathy) with the CPU.

## Guiding Principles

Tractor takes inspiration from F#, Odin, and Nim.

The core assumptions of Tractor design are:

- **NOT** a general-purpose language
- No garbage collection
- Data and Logic are completely separate
- Zero hidden allocation, no hidden safe copying
- A rich domain model makes it easier to reason about and express algorithms
- No support for interoperating with external runtimes
- Languages call Tractor, Tractor does not call other languages
- Users can write their own data structures
- The runtime environment supports virtual memory and 64-bit memory addressing

## Features

- Statically Typed
- Parametric Polymorphism
- Expression oriented (everything returns something)
- Immutable by default
- Mutation is allowed and is explicit
- Memory Management through the use of Memory Contexts
- [Uniform Function Call Syntax](https://en.wikipedia.org/wiki/Uniform_Function_Call_Syntax)
- Procedure Overloading using order of declaration and "best fit" of types
- No implicit casting
- Automatic de-referencing of references
- White space is significant
- No reference types/objects. There are references to values though.

## Undesired Features

- Reflection
- Exceptions

## Primitives

The primitives of Tractor are:

- Signed integers: `i8`, `i16`, `i32`, `i64`
- Unsigned integers: `u8`, `u16`, `u32`, `u64`
- Floats: `f32`, `f64`
- Char (8-bit): `char`
- Rune (UTF-8 character): `rune`
- UTF-8 String: `str`
- Ref: `^T`
- Array: `'[]T`
- Unit

Other possibles
- Slice (subsection of an array)

## Bindings

There are several different types of bindings in Tractor.

- Compile Time: `name : [type] : expr`
- Runtime: `name : [type] = expr`
- Mutable: `name : [type] ! expr`

A Compile Time binding must be able to be evaluated at the time the program is compiled. It is allowed to include expressions like `1.0 / 10.0` but all information for evaluating the expression must be available at compile time. These bindings cannot change.

Runtime bindings are evaluated at the time the program is running. They are also immutable by default when they are evaluated. It is possible to shadow previous bindings by using the same name. The previous declaration will still exist though.

Mutable bindings are evaluated at the time the program is run and are allowed to be changed as the program runs using the set operation `<-`.

## User Defined Types

Tractor takes heavy inspiration from [Odin](https://odin-lang.org/) and [Jai](https://github.com/BSVino/JaiPrimer/blob/master/JaiPrimer.md) for the value declaration syntax.

```fsharp
// Struct definition syntax
struct-name :: struct
    field-name : field-type
    ...

// Struct example
Chicken :: struct
    Size: f32
    Age: i32
    Weight: f32

// Create an instance of Chicken
c : Chicken =
    Size 10
    Age 1
    Weight 10.0
// Or with type inference
c :=
    Size 10
    Age 1
    Weight 10.0
// Or with a single line
c := (Size 10, Age 1, Weight 10.0)

// You can create a new struct from an existing struct using `with`
c2 := c with Size 20
// Or on a separate line but within the scope
c2 := c with
    Size 20

// It is also possible to define structs with generic types
Entry<'a> :: struct
    Value: 'a
    Size: f64

// Union definition syntax
union-name :: union
    case-name [: case-type]
    ...

// Union Example
Animal :: union
    Chicken: Chicken
    Bear: Bear

// Union with generic type
Option<'a> :: union
    Some: 'a
    None

// An Enum is a special case of Union
Direction :: union
    North
    South
    East
    West
    
// Boolean is a special case of Union
Boolean :: union
    True
    False

// For ease of development, there is an alias for true and false
true :: Boolean.True
false :: Boolean.False

// Expressions
expr-name :: expr ([parameter-name[:parameter-type], ...]) [-> return-type]
    expr-body
    
// or in a more horizontally compact form
expr-name :: expr
    ([[parameter-name]:parameter-type ...])
    [-> return-type]
    expr-body
    
// An expression for adding two i32
Add2 :: expr (a: i32, b:i32) -> i32
    a + b
    
// Same expression with more type inference
Add2 :: expr (a: i32, b)
    a + b
// The type of `b` and the return type are inferred based on `a` being an `i32`
```

## Basic Syntax

```fsharp
// Syntax for declaring a value
value-name : [type] = value
// The Type Annotation can be ommited and type inference will resolve
// the type being declared

// The integers 8-bit through 64-bit
// i8, i16, i32, i64
x : i8  = 10   // 8-bit integer
x : i16 = 10 // 16-bit integer
x : i32 = 10 // 32-bit integer
x : i64 = 10 // 64-bit integer

// 32-bit integer is the default integer size so this would result
// in a 32-bit integer being bound to the value of x
x := 10

// The unsigned integers
// u8, u16, u32, u64
x : u8  = 10   // 8-bit integer
x : u16 = 10 // 16-bit integer
x : u32 = 10 // 32-bit integer
x : u64 = 10 // 64-bit integer

// The floats
x : f32 = 10
x : f64 = 10

// You may have noticed that the values being bound to `x` did not have a `.`
// at the end. This is because the type annotation made it clear that the values
// were to be interpreted as f32 and f64. If you wish to omit the type annotation
// you can include a `.` at the end to indicate the value is meant to be interpreted
// as a float. A 64-bit float is the default size for float values.
x := 10.0 // f64
// or
x := 10. // f64
// It is encouraged to include at least a single `0` after the `.` for style
// but is not enforced by the compiler.

// Values are immutable by default. If you wish to have a value which can be mutated
// you may use a mutable binding.
x :! 10 // mutable i32
```

## Uniform Function Call Syntax

Tractor does not have objects in the .NET sense. This means that types do not have methods associated with them. Instead, Tractor supports [Uniform Function Call Syntax](https://en.wikipedia.org/wiki/Uniform_Function_Call_Syntax) in the vein of [Nim](https://nim-lang.org/) and [D](https://dlang.org/). This means that we can still use dot-driven development.

```fsharp
Chicken :: struct
    Name: string
    Size: float
    
Grow :: expr (c: Chicken, sizeToAdd)
    c with Size (c.Size + sizeToAdd)
    
// You can call the `Grow` expression apart from the type
c1 := Name "Chicky", Size 10.0
c2 := Grow(c1, 1.0)
// or use UFCS
c2 := c1.Grow(1.0)
// By using UFCS, the compiler knows that we must be calling an expression where the first
// parameter is a `Chicken` and automatically puts that value in the first argument.
```

## Memory Management

Tractor is does not have manual memory management, nor does it have a Garbage collection. It uses what I refer to as manu-matic memory management. There is always an allocator that is in scope that allocates space using a simple [Arena Allocator](https://en.wikipedia.org/wiki/Region-based_memory_management). These types of allocators are also referred to as Region Allocators. When a Tractor program starts it creates a Region for the root of the program. As the program progresses, the user can allocate a value on in the Region using the `new` keyword.

```fsharp
// Stack allocated Chicken
c1 := Name "StackChicken" Size 1.0

// Heap allocated Chicken
c2 := new (Name "HeapChicken" Size 2.0)
// Heap allocated Chicken with explicit typing
c2 : ^Chicken = new (Name "HeapChicken" Size 2.0)
```

When using the `new` keyword to create an instance of a value a reference to the value is returned. This means that `c2` in the previous example is not a `Chicken` but a `^Chicken`. Fields of references are automatically de-referenced when accessed so there is not syntactic overhead when using a stack-allocated value versus a heap-allocated value.

If there was only ever a single Region, a Tractor program could potentially allocate a large amount of memory. To address this issue we take advantage of virtual memory on 64-bit based systems. We can create a new heap using the `region` keyword.

```fsharp
// Expression that creates a new Region
FrameLogic :: (state: State)
    // Declare that a new Region should be used for newly allocated values
    region
        // Do some work here
```

A Region is an allocation scope. When we declare a new Region using the `region` keyword we are saying that all uses of the `new` keyword will allocate to this new Region. When the Region is leaves scope, we return the allocation pointer to the beginning of the Region and therefore instantly de-allocate all of the memory.

For this to work we cannot have references going from a parent Region to a child Region. This means that the compiler is aware of which references belong to each Region. This somewhat analogous to the Borrow Check in Rust but far simpler. The compiler is ensuring that we will not create invalid references. It is possible for a Child Region to hold references to a Parent region since those references will not go out of scope.

This is all possible due to Memory Virtualization. Tractor will keep subdividing the Virtual Memory space when new Regions are required. Since the Virtual Memory space on an x86-64 system is 256TB of space, it is unlikely the program will run out of virtual space. Only when memory is actually required will the space be actually requested from the operating system.
