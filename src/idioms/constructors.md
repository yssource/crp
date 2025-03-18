# Constructors

In C++, constructors initialize objects.

```c++
class Person {
  int age;
public:
  Person(int a) : age(a) {}
};
```

At the point when a constructor is executed, storage for the object has been
allocated and the constructor is only performing initialization.

```c++
class Person {
  int age;
  A* best_friend;
public:
  A(int a) : age(a), best_friend(this) {}
};
```

## Constructors in Rust

Rust does not have constructors in the same way as C++. Instead the term
"constructor" in Rust refers to a static method associated with a type (i.e., a
method that does not have a `self` parameter), which returns a value of the
type.

```rust
struct Person {
    age: i32
}

impl Person {
    pub const fn with_age(a: i32) -> Self {
        Self { age: a }
    }
}
```

Typically the primary constructor for a type is named `new`, especially if it
takes no arguments. See the chapter on [default
constructors](constructors/default_constructors.html). Additional constructors
are usually named `with_details` (e.g., `Person::with_name`). See the [naming
guidelines](https://rust-lang.github.io/api-guidelines/naming.html) for the
conventions on how to name constructor methods in Rust.

If the fields to be initialized are public, there is a reasonable default value,
and the value does not manage a resource, then it is also common to use record
update syntax to initialize a value based on some default value:

```rust
struct Point {
    pub x: i32,
    pub y: i32,
    pub z: i32,
}

impl Point {
    pub const fn zero() -> Self {
        Self { x: 0, y: 0, z: 0 }
    }
}

fn go() {
    let x_unit = Point {
        x: 1,
        ..Point::zero()
    };
    // ...
}
```

A similar approach in C++ would likely be discouraged because it would require
mutating a value, and so would prevent the variable from being `const`. In Rust
this is not required. This way of creating values is idiomatic.

## Storage allocation vs initialization

The actual construction of a structure or enum value in Rust occurs where the
structure construction syntax `A { ... }` is, after the evaluation of the
expressions for the fields.

A significant implication of this difference is that storage is not allocated
for a struct in Rust at the point where the "constructor" is called, and in fact
is not allocated until after the values of the fields of a struct have been
computed (in terms of the semantics of the language--the optimizer may still
avoid the copy). Therefore there is no way in Rust to write the C++ example
above where the class stores a pointer to itself upon construction.

<!-- TODO refer to resource on how to model self-referential data structures. -->

## Fallible constructors

In C++ the only way constructors can indicate failure is by throwing exceptions.

```c++
#include <stdexcept>

class Person {
  int age;
public:
  A(int a) : age(a) {
    if myint < 0 {
        throw std::domain_error("Bad argument");
    }
  }
};
```

In Rust, because constructors are normal static methods, fallible constructors
can instead return `Result` (akin to `std::expected`) or `Option` (akin to
`std::optional`).

```rust
struct Person {
    age: i32,
}

impl Person {
    # // TODO showcase better practice for error type?
    pub fn with_age(a: i32) -> Result<Self, &'static str> {
        if a < 0 {
            Err("Bad argument")
        } else {
            Ok(Self { age: a })
        }
    }
}
```

See [the chapter on exceptions](/exceptions.md) for more information on how C++
exceptions and exception handling translate to Rust.
