# Separate construction and initialization

The approach to take in Rust depends on the reason for separating construction
and initialization.

- For incremental initialization, use a [builder pattern](#rust-builder-pattern).
- Using virtual methods during construction is [not applicable to
  Rust](#using-virtual-methods-during-initialization).
- For pre-allocating storage or re-using allocated objects, the techniques and
  limitations described in the chapter on [pre-allocated
  buffers](../out_params/pre-allocated_buffers.md) apply.

## Rust builder pattern

Implementing the builder pattern in Rust involves defining a second "builder"
type to represent the partially constructed value, where each field of type `T`
has type `Option<T>` in the builder. This differs from C++ where either null
values or uninitialized memory can be used to construct an object incrementally.

<div class="comparison">

```cpp
#include <memory>
#include <string>

struct Pet {
  std::string name;
};

struct Person {
  int age;
  std::shared_ptr<Pet> pet;
};

int main() {
  Person person;
  // Can initialize incrementally without a builder.
  //
  // Initilizes with age indeterminate and pet as
  // nullptr.
  person.age = 42;
  person.pet = std::make_shared<Pet>("Mittens");
}
```

```rust
use std::rc::Rc;

struct Pet {
    name: String,
}

struct Person {
    age: i32,
    pet: Rc<Pet>,
}

struct PersonBuilder {
    age: Option<i32>,
    pet: Option<Rc<Pet>>,
}

impl PersonBuilder {
    fn new() -> PersonBuilder {
        PersonBuilder {
            age: None,
            pet: None,
        }
    }

    fn age(&mut self, age: i32) -> &mut Self {
        self.age = Some(age);
        self
    }

    fn pet(&mut self, pet: Rc<Pet>) -> &mut Self {
        self.pet = Some(pet);
        self
    }

    fn build(&self) -> Option<Person> {
        Some(Person {
            age: self.age?,
            pet: self.pet.clone()?,
        })
    }
}

fn main() {
    let mut builder = PersonBuilder::new();
    let pet = Rc::new(Pet {
        name: "Mittens".to_string(),
    });
    let person = builder.age(42).pet(pet).build();
}
```

</div>

This pattern is sufficiently common that there are libraries to support it, such
as the [`derive_builder` crate](https://crates.io/crates/derive_builder). Using
that crate, the above example is much shorter.

```rust,ignore
#[derive(Builder)]
struct Person {
    age: i32,
    name: String,
}
```

The resulting API also includes additional features, such as the `build` method
returning a `Result::Err` with an informative error, rather than just `None`,
when not all of required fields are set.

### An alternative: updating based on a default value

If there is a reasonable default value for a type, then instead of the builder
pattern, the `Default` trait can be implemented. [Values can be constructed
based on the default value proved by the `Default`
implementation](./default_constructors.md#struct-update).

### Why builders are more common in Rust than in C++

The builder pattern is used more often in C++ than in Rust because

1. Rust models ownership of pointers orthogonally to optionality, and
2. Rust requires handling all variants of a tagged union.

This encourages using the type system to model invariants more explicitly, which
means that if different invariants hold before and after construction is
completed, different structs need to be defined to represent those different
states.

In particular, while a value is in the middle of being incrementally
constructed, the fields are optional. Once fully constructed, the fields are no
longer optional.

## Using virtual methods during initialization

Separate initialization is sometimes used in C++ to overcome the limitation that
calling virtual methods during construction in is undefined behavior. The
difference in mechanics in construction in Rust make this kind of workaround
unnecessary. The code that usually runs as part of a constructor in C++ is
defined as a static method in Rust. The kind of partially-constructed state that
exists during the execution of the constructor in C++ does not exist in Rust.
