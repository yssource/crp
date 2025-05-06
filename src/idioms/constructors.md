# Constructors

In C++, constructors initialize objects.

```cpp
class Person {
  int age;

public:
  Person(int a) : age(a) {}
};

int main() {
  Person person(42);
  // ...
}
```

At the point when a constructor is executed, storage for the object has been
allocated and the constructor is only performing initialization.

```cpp
class Person {
  int age;
  // non-owning pointer
  Person *best_friend;

public:
  Person(int a) : age(a), best_friend(this) {}
};

int main() {
  Person person(42);
  // ...
}
```

Rust does not have constructors in the same way as C++. In Rust, there is a
single fundamental way to create an object, which is to initialize all of its
members at once. The term "constructor" or "constructor method" in Rust refers
to something more like a factory: a static method associated with a type (i.e.,
a method that does not have a `self` parameter), which returns a value of the
type.

```rust
struct Person {
    age: i32
}

impl Person {
    const fn with_age(a: i32) -> Self {
        Self { age: a }
    }
}

fn main() {
    let person = Person::with_age(42);
    // ...
}
```

Typically the primary constructor for a type is named `new`, especially if it
takes no arguments. See the chapter on [default
constructors](constructors/default_constructors.html). Constructors based on
some specific property of the value are usually named `with_details` (e.g.,
`Person::with_name`). See the [naming
guidelines](https://rust-lang.github.io/api-guidelines/naming.html) for the
conventions on how to name constructor methods in Rust.

If the fields to be initialized are visible, there is a reasonable default
value, and the value does not manage a resource, then it is also common to use
record update syntax to initialize a value based on some default value.

```rust
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

impl Point {
    const fn zero() -> Self {
        Self { x: 0, y: 0, z: 0 }
    }
}

fn main() {
    let x_unit = Point {
        x: 1,
        ..Point::zero()
    };
    // ...
}
```

Despite the name, "record update syntax" does not modify a record but instead
creates a new value based on another one, taking ownership of it in order to do
so.

## Storage allocation vs initialization

The actual construction of a structure or enum value in Rust occurs where the
structure construction syntax `A { ... }` is, after the evaluation of the
expressions for the fields.

A significant implication of this difference is that storage is not allocated
for a struct in Rust at the point where the constructor method (such as
`Person::with_age`) is called, and in fact is not allocated until after the
values of the fields of a struct have been computed (in terms of the semantics
of the language &mdash; the optimizer may still avoid the copy). Therefore there is no
easy way in Rust to write the C++ example above where the class stores a pointer to
itself upon construction (this requires tools like [`Pin`](https://doc.rust-lang.org/std/pin/struct.Pin.html) and [`MaybeUninit`](https://doc.rust-lang.org/std/mem/union.MaybeUninit.html)).

## Fallible constructors

In C++ the only way constructors can indicate failure is by throwing exceptions. In Rust, because constructors are normal static methods, fallible constructors
can instead return `Result` (akin to `std::expected`) or `Option` (akin to
`std::optional`).

<div class="comparison">

```cpp
#include <iostream>
#include <stdexcept>

class Person {
  int age;

public:
  Person(int a) : age(a) {
    if (age < 0) {
      throw std::domain_error("Bad argument");
    }
  }
};

int main() {
  try {
    Person person(-4);
  } catch (const std::domain_error &e) {
    std::cout << e.what() << std::endl;
  }
}
```

```rust
struct Person {
    age: i32,
}

#[derive(Debug)]
struct NegativeAgeError(i32);

impl Person {
    fn with_age(a: i32) -> Result<Self, NegativeAgeError> {
        if a < 0 {
            Err(NegativeAgeError(a))
        } else {
            Ok(Self { age: a })
        }
    }
}

fn main() {
    match Person::with_age(-4) {
        Err(err) => {
            println!("{err:?}");
        }
        Ok(person) => {
            // ...
        }
    }
}
```

</div>


See [the chapter on exceptions](/exceptions.md) for more information on how C++
exceptions and exception handling translate to Rust.
