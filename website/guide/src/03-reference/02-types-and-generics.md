# Types and generics

A **type** is a way to identify what "kind" of value something is. For example, the expression `"hello"` has type `Text`, and `1 + 2` has type `Number`.

There are five main kinds of types in Warp:

-   **Marker types** have a single value and contain no information.
-   **Structure types** represent a collection of values ("fields"), where each field has a name and stores a single value.
-   **Enumeration types** represent a fixed set of values ("variants"), where each variant has zero or more associated values.
-   **Tuple types** represent a fixed-size, heterogeneous collection of values.
-   **Function types** represent a function that accepts a value of one type and returns a value of another type.

## Type annotations

You can use the `::` operator to explicitly declare the type of a value. If Warp determines that your type is incorrect, it will raise an error. For example, to explicitly declare that `42` has type `Number`:

```warp
42 :: Number
```

> **Note:** If you want to give a variable `x` a type `T`, you can't write `x :: T` at the statement level, as this defines a [constant](03-files-and-constants.html). To get around this, wrap the type annotation in parentheses: `(x :: T)`.

## Catalog of types

### Markers

Marker types can be declared using the `type` template:

```warp
Marker : type
```

To refer to the value the marker represents, just write the name of the marker type. For example, if `x : Marker`, then `x` has type `Marker`.

### Structures

Structure types can be declared using the `type` template followed by a block of type annotations:

```warp
Structure : type {
    x :: Number
    y :: Text
}
```

To create a new structure, write the structure's name followed by a block of variable assignments:

```warp
s : Structure {
    x : 42
    y : "hello"
}
```

### Enumerations

Enumeration types can be declared using the `type` template followed by a block of variants:

```warp
Grade : type {
    A
    B
    C
    D
    F
}
```

You can also add associated values to each variant:

```warp
Either : type {
    Left Number
    Right Text
}
```

To create a variant of the enumeration, write the enumeration's name, followed by the variant's name, followed by any associated values:

```warp
g : Grade A
e : Either Left 42
```

If you want to refer to the variants directly without having to write the enumeration's name every time, you can use the `use` template:

```warp
use Grade
g : A

use Either
e : Left 42
```

### Tuples

Tuples and tuple types can be declared using the `,` operator:

```warp
(1 , "a" , True) :: (Number , Text , Boolean)
```

The empty tuple `()` is also valid. Usually, `()` is used for a function that accepts and/or returns no meaningful value:

```warp
show 42 :: ()
```

### Function types

Functions and function types can be declared using the `->` operator:

```warp
(x -> x) :: (Number -> Number)
```

In Warp, functions may only accept one value. To accept another value, make the function return another function and move your computation into that new function:

```warp
f : (x -> y -> x + y) :: (Number -> Number -> Number)
g : (f 1) :: (Number -> Number)
h : (g 2) :: Number
```

## Generics

Warp supports generics in the form of **type functions**, which accept one or more types and produce a new type as a result. For example, we can redefine `Either` from above to be more generic:

```warp
Either : A B => type {
    Left A
    Right B
}
```

To use such a type function, you call it by its name, providing the specific types as input:

```warp
Left 42 :: Either Number Text
```

Here, the annotation is required because `Left` only refers to `A`, meaning there's no way for Warp to automatically determine `B`.

### Type placeholders

You can use `_` to represent a placeholder, the type of which Warp should determine automatically. For example, we know the type of `A` in the above example to be `Number`, so we can make the type annotation more concise using a placeholder:

```warp
Left 42 :: Either _ Text
```

In a type function, you can use `_` to create an implicit type parameter:

```warp
left :: Left => Either Left _ -> Maybe Left
left : ...

right :: Right => Either _ Right -> Maybe Right
right : ...
```
