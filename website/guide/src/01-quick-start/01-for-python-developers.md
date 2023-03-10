# Quick start for Python developers

Welcome to Warp! This guide goes over some basic Python concepts and their equivalent in Warp. When you finish this guide, you'll have a foundational understanding of Warp code that you can use to experiment on your own.

## Hello, world

Warp's equivalent of `print` is `show`:

```warp
show "Hello, world!"
```

## Comments, numbers and strings

You can write a comment using `--`, similar to Python's `#`.

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

You can use `format` to do string interpolation, similar to Python's `%` operator:

```warp
format "Hello, _!" "world" -- Hello, world!
```

## Variables

In Warp, you can declare variables using the `:` operator:

```warp
answer : 42
name : "Warp"
```

Warp uses lexical scoping instead of function scoping. Basically, that means a variable is only accessible within the block that created it. For example, this wouldn't work:

```warp
if True {
    a : 1
} {
    a : 2
}

show a -- error: cannot find `a`
```

If you want to assign to `a` based on a condition, use the `if` on the right-hand side of the `:`, like so:

```warp
a : if True 1 2
```

## `if` statement

Warp's `if x a b` is equivalent to Python's `x if a else b`:

```warp
password : "letmein123"
valid : password = "password123!" -- use a single '=' to compare values
show (if valid "Access granted" "Access denied") -- Access denied
```

Note that Warp doesn't have an `if` statement. If you want to execute multiple statements inside an `if`, either refactor the code to use functions or use a block expression:

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

Warp is a statically-typed language, which means that your code is verified at compile-time. Luckily, Warp has type inference, so you usually don't need to think about types at all! You can use `::` to annotate the type of a value.

```warp
42 :: Number
"Hello" :: Text
```

If you mismatch the types, Warp will emit an error:

```warp
42 :: Text -- mismatched types: expected `Text`, but found `Number`
```

## Classes and objects

Warp calls classes "types", which you can create using `type`. If you've used type annotations in Python, this should look pretty familiar:

```warp
Person : type {
    name :: Text
    age :: Number
}
```

Instead of defining an `__init__` function, in Warp you write the name of the type followed by its fields:

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

Warp's functions work like Python's `lambda` expressions. The left-hand side of the arrow is the input, and the right-hand side is the output:

```warp
increment : x -> x + 1
show (increment 42) -- 43
```

One big difference is that Warp functions may only accept a single parameter. If you want multiple parameters, use multiple functions!

```warp
add : a -> b -> a + b
show (add 1 2) -- 3
```

If that's confusing, here's the equivalent Python code:

```python
add = lambda a: lambda b: a + b
print(add(1)(2))  # 3
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

## Magic methods

Since Warp doesn't have methods, a different approach is needed to implement "magic methods" like `__eq__`. Warp solves this problem by using "traits". Here is a simple example that implements a `Greet` trait for a `Person`:

### Python

```python
# For any value with the `greet()` method, return a greeting
def greet(x):
    return f"Hello, {x.greet()}"

class Person:
    def __init__(self, name: str):
        self.name = name

    # Greet for Person values is defined as the person's name
    def greet(self):
        return self.name

class Earth:
    # Greet for Earth values is defined as "world"
    def greet(self):
        return "world"

greet(Person("Bob"))  # Hello, Bob!
greet(Earth())  # Hello, world!
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

One important difference is that the Python code assumes `x` has a `greet()` method ?????if it doesn't, then the program will crash at runtime. Warp, on the other hand, verifies that `Greet` is implemented for `x` at compile time. It takes some getting used to, but your code will have far fewer bugs!

In Warp, the `=` operator is just shorthand for `Equal left right`, where `Equal` is a trait representing a function that accepts two values and returns a `Boolean`. So we can implement the `Equal` trait for our `Person` type to get equality checking for free, just like Python's `__eq__` method!

### Python

```python
class Person:
    def __init__(self, name: str):
        self.name = name

    def __eq__(self, other: Person):
        return self.name == other.name
```

### Warp

```warp
Person : type {
    name :: Text
}

instance (Equal Person) : p1 -> p2 ->
    name of p1 = name of p2
        and age of p1 = age of p2
```

Warp also allows you to derive traits like `Equal` automatically!

```warp
Person : type {
    name :: Text
}

instance Equal Person -- auto-generates an implementation
```

## List comprehensions

Another popular Python feature is list comprehensions:

```python
def birthday(person):
    return Person(person.name, person.age + 1)

older_people = [birthday(person) for person in people]
```

Warp supports this syntax using the `|` operator:

```warp
birthday :: Person -> Person
birthday : { name age } -> Person {
    name
    age : age + 1
}

older-people : people | birthday
```

## Handling `None`

In Python, you represent the absence of a value using `None`. Warp also has `None`, but Warp helps ensure that you handle `None` throughout your program. It achieves this using a `Maybe` type, which holds `Some x` (for any `x`) or `None`. Imagine a Python program that fetches a user from a database:

```python
class Database:
    def fetch_user(self, id):
        table = self.table("users")

        if table.contains(id):
            return table.get(id)
        else:
            return None

if __name__ == "__main__":
    database = ...

    bob = database.fetch_user(42)
    print(bob.name)
```

Uh oh, this program has a bug ?????we forgot to handle the case where `fetch_user` returns `None`! Now let's write the same program in Warp:

```warp
fetch-user :: Integer -> Database -> Maybe User
fetch-user : id -> database -> {
    table : database . table "users"

    if (table . contains? id)
        (Some (table . get id))
        None
}


database : ...

bob : fetch-user 42
show bob
```

Now we get the following error:

```
error: missing instance
   ?????? playground:11:1
   ???
11 ??? show bob
   ??? ^^^^ could not find instance `Show (Maybe User)`
   ???
   ?????? https://pkg.warp.za16.co/std/output.wpl:16:17
   ???
16 ??? show :: A where (Show A) => A -> ()
   ???                 -------- required by this bound here
```

As you can see, Warp doesn't know how to `show` a `Maybe User`. We have to handle the case where the user is `None`! In Warp, we can accomplish handle the `None` case using `when`:

```warp
when (database . fetch-user 42) {
    Some bob -> show bob
    None -> show "error: no such user"
}
```

Great ?????now our code won't crash and we can choose how to handle the error in an application-specific way!

## Exceptions

Warp doesn't have exceptions. Instead, functions that can produce errors return the `Result` type. Similar to `Maybe`, `Result` stores either an `OK x` or an `Error e`. Let's refactor our `fetch-user` example to return a `Result` instead of a `Maybe`:

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
```

And we can handle the error using `when`:

```warp
when (database . fetch-user 42) {
    OK bob -> show bob
    Error error -> show error
}
```

To propagate the error up, you can use `end`:

```warp
bob : when (database . fetch-user 42) {
    OK user -> user
    Error error -> end (Error error)
}

show bob
```

This pattern can be written more succinctly using `try`:

```warp
bob : try (database . fetch-user 42)
show bob
```
