# Default constructors

C++ has a special concept of default constructors to support several scenarios
in which they are implicitly called.

```cpp
class Person {
    int age;

public:
    // Default constructor
    Person() : age(0) {}
}
```

Rust does not have a notion of a default constructor in the same way as in C++.
Some of the uses cases are achieved via a different mechanism or with different
conventions, and others do not apply to Rust.

If a structure has a useful default value (such as would be constructed by a
default constructor in C++), then the type should provide
[both](https://rust-lang.github.io/api-guidelines/interoperability.html?highlight=default#types-eagerly-implement-common-traits-c-common-traits)
a `new` method that takes no arguments and an implementation of the [`Default`
trait](https://doc.rust-lang.org/std/default/trait.Default.html).

```rust
struct Person {
   age: i32,
}

impl Person {
    pub const fn new() -> Self {
        Self { age: 0 }
    }
}

impl Default for Person {
    fn default() -> Self {
        Self::new()
    }
}
```

## Implicit initialization of class members

In C++ if a member is not explicitly initialized by a constructor, then it is
default-initialized. When the type of the member is a class, the
default-initialization invokes the default constructor.

In Rust, if all of the fields of a struct implement the `Default` trait, then an
implementation for the structure can be provided by the compiler.

<div class="comparison">

```cpp
class Person {
  int age;

public:
  Person() : age(0) {}
}

class Student {
  Person person;
}
```

```rust
#[derive(Default)]
struct Person {
    age: i32,
}

#[derive(Default)]
struct Student {
    person: Person,
}
```

</div>

The use of the derive macros in Rust is equivalent to writing the following.

```rust
struct Person {
    age: i32,
}

impl Default for Person {
    fn default() -> Self {
        Self {
            age: Default::default()
        }
    }
}

struct Student {
    person: Person,
}

impl Default for Student {
    fn default() -> Self {
        Self {
            person: Default::default()
        }
    }
}
```

Unlike C++ where the default initialization value for integers is indeterminate,
in Rust the default value for the primitive integer and floating point types is
zero.

<a name="struct-update"></a> Deriving the `Default` trait has a similar effect
on code concision as eliding initialization in C++. In situations where all of
the types implement the `Default` trait, but only some of the fields should have
their default values, one can use [struct update
syntax](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax)
to define a constructor method without enumerating the values for all of the
fields.

```rust
#[derive(Default)]
struct Person {
    age: i32,
}

#[derive(Default)]
struct Student {
    person: Person,
    favorite_color: Option<String>,
}

impl Student {
    pub fn with_favorite_color(color: String) -> Self {
        Student {
            favorite_color: Some(color),
            ..Default::default()
        }
    }
}
```

## Implicit initialization of array values

In C++, arrays without explicit initialization are default-initialized using the
default constructors.

In Rust, the value with which to initialize the array must be provided.

<div class="comparison">

```cpp
class Person {
  int age;

public:
  Person() : age(0) {}
};

int main() {
  Person people[3];
  // ...
}
```

```rust
#[derive(Default)]
struct Person {
    age: i32,
}

fn main() {
    // std::array::from_fn provides the index to the callback
    let people: [Person; 3] = 
        std::array::from_fn(|_| Default::default());
    // ...
}
```

</div>

If the type happens to be [trivially
copyable](/idioms/constructors/copy_and_move_constructors.md#trivially-copyable-types),
then a shorthand can be used.

```rust
#[derive(Clone, Copy, Default)]
struct Person {
    age: i32,
}

fn main() {
    let people: [Person; 3] = [Default::default(); 3];
    // ...
}
```

## Container element initialization

In C++, the default constructor could be used to implicitly define collection
types, such as `std::vector`. Before C++11 one value would be default
constructed, and the elements would be copy constructed from that initial
element. Since C++11, all elements are default constructed.

As with array initialization, the values must be explicitly specified in Rust.
The vector can be constructed from an array, enabling the same syntax as with
arrays.

<div class="comparison">

```cpp
#include <vector>

class Person {
  int age;

public:
  Person() : age(0) {}
}

int main() {
  std::vector<Person> v(3);
  // ...
}
```

```rust
#[derive(Default)]
struct Person {
    age: i32,
}

fn main() {
    let people_arr: [Person; 3] = std::array::from_fn(|_| Default::default());
    let people: Vec<Person> = Vec::from(people_arr);
    // ...
}
```

</div>

If the type implements the `Clone` trait, then the array can be constructed
using the `vec!` macro. See the chapter on [copy
constructors](/idioms/constructors/copy_and_move_constructors.md) for more
details on `Clone`.

```rust
#[derive(Clone, Default)]
struct Person {
    age: i32,
}

fn main() {
    let people: Vec<Person> = vec![Default::default(); 3];
    // ...
}
```

## Implicit initialization of local variables

In C++, the default constructor is used to perform default-initialization of
local variables that are not explicitly initialized.


In Rust, initialization of local variables is always explicit.

<div class="comparison">

```cpp
class Person {
  int age;

public:
  Person() : age(0) {}
};

int main() {
  Person person;
  // ...
}
```

```rust
#[derive(Clone, Default)]
struct Person {
    age: i32,
}

fn main() {
    let person = Person::default();
    // ...
}
```

</div>

## Implicit initialization of the base class object

In C++, the default constructor is used to initialize the base class object if
it no other constructor is specified.

```cpp
class Base {
  int x;

public:
  Base() : x(0) {}
};

class Derived : Base {
public:
  // Calls the default constructor for Base
  Derived() {}
};
```

Since Rust does not have inheritance, there is no equivalent to this case.
See the chapter on [implementation
reuse](/idioms/data_modeling/inheritance_and_reuse.md) or the section on [traits
in the Rust book](https://doc.rust-lang.org/book/ch10-02-traits.html) for
alternatives.

## `std::unique_ptr`

There are some additional cases where the `Default` trait is used in Rust, but
default constructors are not used for initialization in C++.

Rust's equivalent of smart pointers implement `Default` by delegating to the
`Default` implementation of the contained type.

```rust
#[derive(Default)]
struct Person {
  age: i32,
}

fn main() {
    let b: Box<Person> = Default::default();
    // ...
}
```

This differs from the treatment of `std::unique_ptr` C++ because unlike `Box`,
`std::unique_ptr` is nullable, and so the default constructor for
`std:unique_ptr` produces a pointer that owns nothing. The equivalent type in
Rust is `Option<Box<Person>>`, for which the `Default` implementation produces
`None`.

## Other uses of `Default`

[`Option::unwrap_or_default`](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or_default)
makes use of `Default`, which makes getting a default value when the `Option`
does not contain a value more convenient.

```rust
fn go(x: Option<i32>) {
    let a: i32 = x.unwrap_or_default();
    // if x was None, then a is 0

    // ...
}
```

In C++, `std::optional` does not have an equivalent method.

{{#quiz default_constructors.toml}}
