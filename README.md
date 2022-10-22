# Tractor: A CLR Language for Numerical Computation

The original intention of the dotnet was to provide a runtime for many programming languages to run on. Two factors have kept that vision from fruition: tying .NET Framework to Windows and the dominance of C#.

Dotnet can now run on almost any modern system. A significant push has been made toward performance features for C# and the CLR. The intention is to close the performance gap between systems-level programming languages (C, C++, Rust, Zig, and Odin, to name a few) and what can be achieved on the dotnet platform.

A general-purpose language like C# and F# must support 

## Guiding Principles

The core assumptions of Tractor design are:

- **NOT** a general-purpose language
- Data and Logic are completely separate
- Give the programmer control
- Clarity is more important than brevity
- Zero hidden allocation, no hidden safe copying
- Data and Behavior are separate
- A rich domain model makes it easier to reason

## Desired Features

- Statically Typed
- Parametric Polymorphism
- Immutable by default
- Mutation is allowed and is explicit
- Polish Notation?
- Order of precedence is left to right

## Undesired Features

- Reflection
- Strings
- Exceptions

## Code Examples

```fsharp
// Defining a Struct
Chicken struct
    Size : f32
    Age : i32
    Weight : f32

// Defining a Union
Animal union
    Chicken : Chicken
    Bear : Bear

// Define an Enum
Direction union
    North
    South
    East
    West

// Create a tuple
x = 1, 2.0

// Same thing with type declaration
x : i32 * f32 = 1, 2.0

// Unpack a tuple
a, b = x

// Define a function for adding i32 with explicity
// type annotation
addInt32 : i32 -> i32 -> i32 = { a b ->
    // Polish notation
    + a b
}

// Same thing but putting the type information on the variables
addInt32 = { a:i32 b:i32 ->
    + a b
}

// Define a function to add f32
addFloat32 = { a:f32 b:f32 ->
    + a b
}

// Define the overloads for the `Add` function
add = { addInt32; addFloat32; }

addInt8 = { a:i8 b:i8 ->
    + a b
}

// Extend the overloads for `add` function
add = { addInt8; add; }

// Call the add function on i32
x = add 1 2
// Call the add function on i8
x = add 1i8 2i8
// COMPILER ERROR: No compatible overloads
x = add 1 2i8


// Binding a value. x is immutable
x = 10

// Bind a value while specifying the type
x : i32 = 10

// Creating a variable. y is mutable
y =! 10

// Create a variable while specifying the type
y : i32 =! 10

// Assign a new value to y
y <- 42

// Bind c to a new instance of Chicken
c = Chicken { Size 1.0; Age 10; Weight 12.2 }
// or
c = Chicken {
    Size 1.0
    Age 10
    Weight 12.2
}

// Create an Animal Union
a = Animal.Chicken c

// Unpack a Union
match a with
| Chicken c -> // Chicken code
| Bear b -> // Bear code

// Define a Struct with a mutable field, Weight
Bear struct
    Size : f32
    Weight : f32!

// Bind b to a new instance of Bear
b = Bear { Size 10.0; Weight 100.0 }

// COMPILER ERROR: This will not work because b is not mutable
b.Weight <- 110.0

// Create mutable binding for b2
b2 =! Bear { Size 10.0; Weight 100.0 }

// This works now because b2 is a mutable binding and the Weight
// field is mutable
b2.Weight <- 100.0

// COMPILER ERROR: This will not work because the field Size is not mutable
b2.Size <- 12.0

// Looping and iteration
// There is only one looping construct, the while loop
while <condition> do
    // Loop body

// The core collections can define their own looping constructs like
// map, iter, mapi, iteri with higher ordered functions.

// A classic for loop
i =! 0 // Note this is mutable so it can be incremented

while i < myArray.Length do
    // do something here
    // Increment the counter
    i <- + i 1

// Of course, the collections library would include more convenient looping constructs
// that use higher-ordered functions
myArray
|> map {i ->
    // Mapping logic
}

```


## Types

There are only Value types. Types are always copied by value. That value could be a typed pointer, though.

## Primitives

The primitive types are the building blocks upon which all other types are derived. They are divided into two categories: Data and Memory. 

__Data Types__
- Boolean: `true`, `false`
- Integers: `i8`, `i16`, `i32`, `i64`
- Unsigned Integers: `u8`, `u16`, `u32`, `u64`
- Floats: `f16`, `f32`, `f64`

__Memory Types__

- Pointer: `p32`, `p64`
- Typed Pointer: `Ptr<'T>`

## Derived Types

New types can be derived from the Primitives. Derived Types can be one of two forms: Struct or Union. Structs are a collection of fields. Unions are a collection of possible cases with the different type associated with each case.

### Structs

Structs are a collection of fields. Each field has a name and an associated type. The syntax for declaring a Struct is the following:

```
<Struct Name> struct
    <Field Name> : <Field Type>
    <Field Name> : <Field Type>
    ...
```

An example of defining a `Chicken` type with a `Size` field of `f32` and an `Age` field of `i16` can be seen in the following code snippet.

```
Chicken struct
    Size : f32
    Age : i16
```

Structs can be created using a syntax similar to F#. What is different from F# is that you give the name of the Struct that you are trying to construct before the `{...}` construction syntax. When it comes to Records, I have found that type inference to not always be as helpful as I hoped. I often had to specify the return value to get the Record type I wanted.

Here's an example of creating an instance of the `Chicken` type we declared before. We show both a single-line syntax and a multi-line syntax.

```
// Compiler should see that we are making a Chicken and automatically see that
// 1.0 can be a `f32` and that 10 can be a `i16`
myChicken = Chicken { Size 1.0; Age 10 }

// Equivalent creation but using newlines
myChicken = Chicken {
    Size 1.0
    Age 10
}
```

You can also create new instances of a Struct from an existing one while updating fields using the `{ <Old Struct> with <Updated Fields> }` syntax.

```
newChicken = { myChicken with Size 2.0 }
```

By default, Structs are immutable. It is possible to declare that a field is immutable, though, by using the `!` symbol after the field type. Let's return to our `Chicken` type, add a field for Weight, and make it a mutable float 32.

```
Chicken struct
    Size : f32
    Age : i16
    Weight : f32! // <-- This field can be mutated
```

### Unions

Unions are a collection of cases with types associated with the cases. The syntax for a Union declaration is as follows:

```
<Union Name> union
    <Case Name> : <Case Type>
    <Case Name> : <Case Type>
    ...
```

An example of defining an `Animal` Union with two cases, `Chicken` and `Dog,` can be seen in the following code snippet. Note that the Case Name matches the type name associated with the Union Case. This is unnecessary but common when domain modeling.

```
Animal union
    Chicken : Chicken
    Dog : Dog
```

A Union's size will equal the size of the largest possible case + `u32`. The `u32` serves as the tag for which case the Union is.

Enums are a special case of Union. There are a set of Union cases with no types associated with the different cases. Here is an example of an Enum using a Union:

```
MyFauxEnum union
    CaseA
    CaseB
    CaseC
```

To check the case of a Union, you use a `match...with` a statement like in F#.

```
myChicken = Chicken { Size 1.0; Age 10 }
myAnimal = Animal.Chicken myChicken

match myAnimal with
| Chicken c -> // Chicken logic
| Dog d -> // Dog logic
```

## Functions

While Tractor is not a functional language in the Computer Science since, it does have first class functions.

```
myFunction func
    (a: int -> b: int -> int)
    a + b
```

## Units of Measure

It is possible to annotate the Data Primitives with additional type information using Units of Measure (UoM). UoM ensures that the algebra between the data types behaves as it should. To declare a new UoM, use the `measure` keyword. The syntax for declaring a Unit of Measure is as follows.

```
<Unit Name> measure
```

To declare a `Meter` as a Unit of Measure, you can do the following:

```
Meter measure
```

You can now annotate your Data Primitives with UoM using the following syntax:

```
<Value Name> = <Data Type><Measure Name>
```

Here is an example of declaring a value `x` and assigning it the float32 value of 12.0 meters.

```
x = 12 f32<Meter>
```

## Distinct

> See Odin's `distinct` feature: https://odin-lang.org/docs/faq/#what-does-distinct-do

The `distinct` keyword allows you to define a new type that has all the same functionality as the source type, but makes it unique type unto itself. This means that if you have a type `'T` that defines addition as `'T -> 'T -> 'T` then the new type `'TDistinct` will also have addition defined, `'TDistinct -> 'TDistinct -> 'TDistinct`, but it will not be compatible with the type `'T`.

Here is an example of this in action:

```
NewType = distinct int32

x1 = 1
x2 = 2
z = x1 + x2 // This works because int32 defines addition

myNewType = NewType 1
/*
This will not work because int32 cannot be
added with the distinct type NewType even
though they are both int32 in memory
*/
z2 = x1 + myNewType

```

## Equality, Hashing, and Comparison

There are two different types of equality, structural and referential. To test whether two values contain the same data and therefore structurally equivalent, use the `=` operator. If you want to know if 

## Loops

Tractor steals the looping concepts of Odin and only has a single looping mechanism, the `for` loop.
Odin Loops: https://odin-lang.org/docs/overview/#for-statement

```
for <Pre Loop Action>; <Loop Condition>; <Post Iteration Action> do
    // Loop actions
```

An example of looping 


## Automatic Looping

Looping is ubiquitous in programming, so it would be natural for there to be a way to express operations on collections of values. Most of the C-derived languages use a `for` loop for iteration. Languages in the functional/ML tradition use higher-ordered functions like `iter` and `map`. Array-oriented languages raise the level of abstraction by automatically operating on arrays of values. This can lead to some terse and powerful programs. The Julia language incorporates an interesting feature which is the addition of a broadcasting macro that turns a scalar function into something that operates over the elements of a collection by prepending the function with a broadcast operator, which is a period in Julia.

Tractor believes in the idea of keyed data. Much like a database table, every collection is not just a set of values but a key/value collection. This means that

```

```

## Memory Management

I would say that Tractor has manumatic memory management through Memory Contexts. When a Tractor process starts up, an implicit Memory Context is created. The default Memory Context has an Arena Allocator (AA), which it uses for providing memory to code that requests allocation. All memory allocation requests are handled by whichever Memory Context is in scope. When a Memory Context scope is left, the allocator's `free_all` method is called, and the allocator is returned to the allocator pool.

If there were only a single Memory Context, then everything would be allocated to the main thread's Arena Allocator, and it would be easy to allocate excessive memory. To curb this, it is possible to create new Memory Contexts using the `context` keyword. An example of creating a new Memory Context with the default settings can be seen here.

```fsharp

let processFrame () =
    context () {// All code after this point will be allocated to the new Memory Context
        // Do lots of work here
    } // When the Context exits here, the Memory Context will clear the memory
```

By using memory contexts, it is possible to allow for the allocation to a heap-like space while not incurring excessive overhead from a Garbage Collector.

What happens when you need to return data that would be allocated to a child context

## Collections

The Tractor language would not include these collections as a default. These would exist as data structures composed of the existing primitives.

- Array
- HashTable
- HashSet
- Map
- Set
- Row
- Bar
