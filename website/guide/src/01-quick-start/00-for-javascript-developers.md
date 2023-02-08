# Quick start for JavaScript developers

Welcome to Warp! This guide goes over some basic JavaScript concepts and their equivalent in Warp. When you finish this guide, you'll have a foundational understanding of Warp code that you can use to experiment on your own.

## Hello, world

Warp's equivalent of `console.log` is `show`:

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

Numbers are represented in base 10 instead of floating point, but they are written the same way:

```warp
42
3.14
-1
```

Strings are called "text" in Warp, and must use double quotes:

```warp
"Hello, world!"
"line 1\nline 2"
```

You can use `format` to do string interpolation:

```warp
format "Hello, _!" "world" -- Hello, world!
```

## Variables

In Warp, you can declare variables using the `:` operator:

```warp
answer : 42
name : "Warp"
```

Warp uses static single assignment, which means that you can't change the value of an existing variable after you create it. However, you can declare the same variable twice — the new variable shadows the old one:

```warp
x : 42
x : x + 1
show x -- 43
```

## `if` statement

Warp doesn't have an `if` statement like in JavaScript. Instead, `if` works more like the ternary operator, and can be used anywhere an expression is needed. By convention, boolean variables end in a question mark.

```warp
password : "letmein123"
valid? : password = "password123!" -- use a single '=' to compare values
show (if valid? "Access granted" "Access denied") -- Access denied
```

## Basic types

Warp is a statically-typed language, which means that your code is verified at compile-time. Luckily, Warp has type inference, so you usually don't need to think about types at all! You can use `::` to annotate the type of a value.

```warp
42 :: Number
"Hello" :: Text
```

If you mismatch the types, Warp will emit an error:

```warp
42 :: Text -- mismatched types: expected `Text`, but found `Number`
```

## Objects

Warp calls objects "types", which you can create using `type`:

```warp
Person : type {
    name :: Text
    age :: Number
}
```

You can create an instance of this object like so:

```warp
bob : Person {
    name : "Bob"
    age : 35
}
```

And you can use destructuring to get the inner values:

```warp
{ name age } : bob
```

## Functions

Warp's functions work like JavaScript's arrow functions. In fact, they both use the arrow notation!

```warp
increment : x -> x + 1
show (increment 42) -- 43
```

One big difference is that Warp functions may only accept a single parameter. If you want multiple parameters, use multiple functions!

```warp
add : a -> b -> a + b
show (add 1 2) -- 3
```

If that's confusing, here's the equivalent JavaScript code:

```javascript
const add = (a) => (b) => a + b;
console.log(add(1)(2)); // 3
```

## Methods

Warp doesn't allow you to add methods to an object (although you can store functions inside types like any other value). Instead, you can declare functions like this:

```warp
greet :: Person -> Text
greet : { name } -> format "Hello, _!" name

greet bob -- Hello, Bob!
```

Alternatively, you can use the `.` operator to chain function calls:

```warp
bob . greet -- Hello, Bob!
```

## Inheritance

Warp has neither classes nor inheritance. Instead, you can use traits! Traits are pretty advanced, but here's a simple example in TypeScript and in Warp:

### TypeScript

```typescript
// Greet is an interface that can be implemented with a function returning text
interface Greet {
    greet(): string;
}

// For any value implementing Greet, return a greeting
function greet<A extends Greet>(x: A): string {
    return `Hello, ${x.greet()}`;
}

class Person implements Greet {
    name: string;

    constructor(name: string) {
        this.name = name;
    }

    // Greet for Person values is defined as the person's name
    greet() {
        return this.name;
    }
}

class Earth implements Greet {
    constructor() {}

    // Greet for Earth values is defined as "world"
    greet() {
        return "world";
    }
}

greet(new Person("Bob")); // Hello, Bob!
greet(new Earth()); // Hello, world!
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
