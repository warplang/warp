# Quick start for Rust developers

Welcome to Warp! This guide goes over some basic Rust concepts and their equivalent in Warp. When you finish this guide, you’ll have a foundational understanding of Warp code that you can use to experiment on your own.

## Hello, world

Warp's equivalent of `println!` is `show`:

```warp
show "Hello, world!"
```

Notice that there's no semicolons in Warp code — just put each statement on its own line.

## Comments, numbers and strings

You can write a comment using `--`. Warp only has line comments:

```warp
-- This is a comment
this is executed -- this is not
```

Numbers are represented in base 10 instead of floating point by default, but they are written the same way:

```warp
42
3.14
-1
```

Strings are called "text" in Warp:

```warp
"Hello, world!"
"line 1\nline 2"
```

Just like in Rust, you can use `format` to do string interpolation. `_` is used as the delimiter instead of `{}`:

```warp
format "Hello, _!" "world" -- Hello, world!
```

## Variables

In Warp, you can declare variables using the `:` operator:

```warp
answer : 42
name : "Warp"
```

Warp has type inference, so you don't need to write the type of the variable — Warp will infer it automatically! If you really want to declare the type, you can do so using the `::` operator:

```warp
answer : (42 :: Number)
name : ("Warp" :: Text)
```

Alternatively, you can write the type on its own line just above the variable declaration:

```warp
answer :: Number
answer : 42

name :: Text
name : "Warp"
```

> **Note:** This syntax actually transforms the variable into a constant that's lazily evaluated. It's primarily intended for use in libraries and not in the bodies of functions, top-level code, or other places where the evaluation order matters. The separate-line syntax is required if you want to use generics or recursion.

Warp doesn't allow you to change variable's value after declaring it. If you need access to mutable state, you can do so using `mutable` (which works like an `Rc<RefCell<T>>`). By convention, functions that change a mutable value end in `!`.

```warp
counter : mutable (0 :: Natural)

show (get counter) -- 0

increment! counter
show (get counter) -- 1
```

## `if` statement

Warp's `if x a b` is equivalent to Rust's `if x { a } else { b }`:

```warp
password : "letmein123"
valid : password = "password123!" -- use a single '=' to compare values
show (if valid "Access granted" "Access denied") -- Access denied
```

If you want to execute multiple statements inside an `if`, you can use a block expression:

```warp
-- This is OK...
if (1 + 1 = 2) {
    show "Woohoo!"
} {
    show "Oh no"
}

-- But this is better...
result : if (1 + 1 = 2) "Woohoo!" "Oh no"
show result
```

## Basic types

Warp has a very similar type system to Rust. You can use `::` to annotate the type of a value:

```warp
42 :: Number
"Hello" :: Text
```

If you mismatch the types, Warp will emit an error:

```warp
42 :: Text -- mismatched types: expected `Text`, but found `Number`
```

## Structs

Warp calls structs "types", which you can create using `type`:

```warp
Person : type {
    name :: Text
    age :: Number
}
```

Just like in Rust, you instantiate a type by writing the name of the type followed by its fields:

```warp
bob : Person {
    name : "Bob"
    age : 35
}
```

And instead of `bob.name` and `bob.age`, you can use destructuring or the `of` operator:

```warp
-- Preferred way
{ name age } : bob

-- Alternative way
name : name of bob
age : age of bob
```

## Functions

Warp's functions work like Rust's closures. `a -> b` is equivalent to Rust's `|a| b`:

```warp
increment : x -> x + 1
show (increment 42) -- 43
```

One big difference is that Warp functions may only accept a single parameter. If you want multiple parameters, use multiple functions!

```warp
add : a -> b -> a + b
show (add 1 2) -- 3
```

If that's confusing, here's the equivalent Rust code:

```rust,noplaypen
let add = |a| { move |b| { a + b } };
println!("{}", add(1.0)(2.0)); // 3
```

## Methods

Warp doesn't allow you to `impl` methods for a type (although you can store functions inside types like any other value). Instead, you can declare functions like this:

```warp
greet :: Person -> Text
greet : { name } -> format "Hello, _!" name

greet bob -- Hello, Bob!
```

Alternatively, you can use the `.` operator to chain function calls:

```warp
bob . greet -- Hello, Bob!
```

## Traits

Warp's traits are similar to Rust's traits, but they are even more powerful. Instead of being limited to being implemented for a single `Self` type, Warp traits can represent a relationship between multiple types at once. Let's start with a simple example though — here is how you would define a `Greet` trait and implement it for `Person` and `Earth`:

### Rust

```rust
// Greet is a trait that can be implemented with a function returning a string
trait Greet {
    fn greet(&self) -> &str;
}

// For any value implementing Greet, return a greeting
fn greet<A>(x: A) -> String
where
    A: Greet,
{
    format!("Hello, {}!", x.greet())
}

struct Person {
    name: String,
}

impl Person {
    fn new(name: impl ToString) -> Self {
        Person {
            name: name.to_string(),
        }
    }
}

// Greet for Person values is defined as the person's name
impl Greet for Person {
    fn greet(&self) -> &str {
        &self.name
    }
}

struct Earth;

    // Greet for Earth values is defined as "world"
impl Greet for Earth {
    fn greet(&self) -> &str {
        "world"
    }
}

fn main() {
    println!("{}", greet(Person::new("Bob"))); // Hello, Bob!
    println!("{}", greet(Earth)); // Hello, world!
}
```

### Warp

```warp
-- Greet is a trait that can be defined with a function returning text
Greet : A => trait (A -> Text)

-- For any value where Greet is defined, return a greeting
greet :: A where (Greet A) => A -> Text
greet : x -> format "Hello, _!" (Greet x)


Person : type {
    name :: Text
}

-- Greet for Person values is defined as the person's name
instance (Greet Person) : { name } -> name


Earth : type

-- Greet for Earth values is defined as "world"
instance (Greet Earth) : just "world"


show (greet (Person { name : "Bob" })) -- Hello, Bob!
show (greet Earth) -- Hello, world!
```

Warp also allows you to derive implementations of traits like `Equal` — just omit the implementation and Warp will generate it for you!

```warp
instance Equal Person -- auto-generates an implementation
```

## `Option<T>` and `Result<T, E>`

Warp's equivalent of `Option<T>` is `Maybe A`, and `Result<T, E>` is `Result Success Failure`. Otherwise, they work in the same way!

-   Use `when` instead of `match` to do pattern matching.
-   Use `end` to exit the current block with a value.
-   Use `try` to exit the current block if the provided value is `None`, `Error`, or another type that can be converted into a `Result`.

Here's an example:

```warp
Database-Error : type {
    message :: Text
}

instance (Show Database-Error) : { message } -> format "database error: _" message


fetch-user :: Integer -> Database -> Result User Database-Error
fetch-user : id -> database -> {
    table : database . table "users"

    if (table . contains? id)
        (OK (table . get id))
        (Error (Database-Error { message : format "no user with id _" id }))
}


bob : try (database . fetch-user 42)
show bob
```
