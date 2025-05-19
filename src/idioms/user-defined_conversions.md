# User-defined conversions

In C++ user-defined conversions are created using [converting
constructors](https://en.cppreference.com/w/cpp/language/converting_constructor)
or [conversion
functions](https://en.cppreference.com/w/cpp/language/cast_operator). Because
converting constructors are opt-out (via the `explicit` specifier), implicit
conversions occur with regularity in C++ code. In the following example both the
assignments and the function calls make use of implicit conversions as provided
by a converting constructor.

```cpp
struct Widget {
  Widget(int) {}
  Widget(int, int) {}
};

void process(Widget w) {}

int main() {
  Widget w1 = 1;
  Widget w2 = {4, 5};
  process(1);
  process({4, 5});

  return 0;
}
```

Rust makes significantly less use of implicit conversions. Instead most
conversions are explicit. The
[`std::convert`](https://doc.rust-lang.org/std/convert/index.html) module
provides several traits for working with user-defined conversions.

In Rust, the above example would make use of explicit conversions by
implementing the [`From`
trait](https://doc.rust-lang.org/std/convert/trait.From.html).

```rust
struct Widget;

impl From<i32> for Widget {
    fn from(_x: i32) -> Widget {
        Widget
    }
}

impl From<(i32, i32)> for Widget {
    fn from(_x: (i32, i32)) -> Widget {
        Widget
    }
}

fn process(w: Widget) {}

fn main() {
    let w1: Widget = 1.into();
    // For construction this is more idiomatic:
    let w1b = Widget::from(1);

    let w2: Widget = (4, 5).into();
    // For construction this is more idiomatic:
    let w2b = Widget::from((4, 5));

    process(1.into());
    process((4, 5).into());
}
```

The `into` method used above is provided via a [blanket
implementations](https://doc.rust-lang.org/book/ch10-02-traits.html#using-trait-bounds-to-conditionally-implement-methods)
for the [`Into trait`](https://doc.rust-lang.org/std/convert/trait.Into.html)
for types that implement the `From` trait.

## Conversion functions

C++ conversion functions enable conversions in the other direction, from the
defined class to another type.

```cpp
#include <utility>

struct Point {
  int x;
  int y;

  operator std::pair<int, int>() const { return std::pair(x, y); }
};

void process(std::pair<int, int>) {}

int main() {
  Point p1{1, 2};
  Point p2{3, 4};

  std::pair<int, int> xy = p1;
  process(p2);

  return 0;
}
```

To achieve the same in Rust, the `From` trait can be implemented in the other
direction, so long as the one of the source or target type is defined in the
same module as the trait implementation.

```rust
struct Point {
    x: i32,
    y: i32,
}

impl From<Point> for (i32, i32) {
    fn from(p: Point) -> (i32, i32) {
        (p.x, p.y)
    }
}

fn process(x: (i32, i32)) {}

fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 3, y: 4 };

    let xy: (i32, i32) = p1.into();
    process(p2.into());
}
```

Conversion functions are is often used to implement the safe bool pattern,
[which is addressed in a different way in
Rust](/idioms/promotions_and_conversions.md#safe-bools).


## Borrowing conversions

An important part of `From` is that the conversion takes ownership of the
converted value. When this is not desired in C++, the conversion function can
just take and return references.

```cpp
#include <iostream>
#include <string>

struct Person {
  std::string name;

  operator std::string &() { return this->name; }
};

void process(std::string &name) { std::cout << name << std::endl; }

// Prints:
// Alice
// Alice
int main() {
  Person alice{"Alice"};

  process(alice);

  std::cout << alice.name << std::endl;

  return 0;
}
```

To achieve this in Rust the [`AsRef`
 trait](https://doc.rust-lang.org/std/convert/trait.AsRef.html) or [`AsMut`
 trait](https://doc.rust-lang.org/std/convert/trait.AsMut.html) can be used.

```rust
struct Person {
    name: String,
}

impl AsRef<str> for Person {
    fn as_ref(&self) -> &str {
        &self.name
    }
}

impl AsMut<str> for Person {
    fn as_mut(&mut self) -> &mut str {
        &mut self.name
    }
}

fn process<T: AsRef<str>>(name: T) {}

fn main() {
    let alice = Person {
        name: "Alice".to_string(),
    };

    process(alice);
}
```

It is common to use `AsRef` as a trait bound in function definitions. Providing
such a bound allows clients to call the functions with anything that can be
cheaply viewed as the type that the function wants to work with. With this in
mind, the above definition of `process` would be defined as in the following
example.

```rust
# struct Person {
#     name: String,
# }
#
# impl AsRef<str> for Person {
#     fn as_ref(&self) -> &str {
#         &self.name
#     }
# }
#
# impl AsMut<str> for Person {
#     fn as_mut(&mut self) -> &mut str {
#         &mut self.name
#     }
# }
#
fn process<T: AsRef<str>>(name: T) {}

fn main() {
    let alice = Person {
        name: "Alice".to_string(),
    };

    process(alice);
}
```

This technique is often used with functions that take file system paths, so that
literal strings can more easily be used as paths.

## Fallible conversions

In C++ when conversions might fail it is possible (though bad practice) to throw
an exception from the converting constructor or converting function.

```cpp
#include <stdexcept>
#include <string>

class NonEmpty {
  std::string s;

public:
  NonEmpty(std::string s) : s(s) {
    if (this->s.empty()) {
      throw std::domain_error("empty string");
    }
  }
};

int main() {
  std::string s("");
  NonEmpty x = s; // throws

  return 0;
}
```

Because error handling in Rust [does not use exceptions](/idioms/exceptions.md),
the [`TryFrom` trait](https://doc.rust-lang.org/std/convert/trait.TryFrom.html)
and [`TryInto` trait](https://doc.rust-lang.org/std/convert/trait.TryInto.html)
can be implemented instead. These traits differ from `From` and `Into` in that
they return a `Result`, which may indicate a failing case. When a conversion may
fail one should implement `TryFrom` and rely on the client to call `unwrap` on
the result, rather than panic in a `From` implementation.

```rust
use std::convert::TryFrom;
use std::convert::TryInto;

struct NonEmpty {
    s: String,
}

#[derive(Clone, Copy, Debug)]
struct NonEmptyStringError;

impl TryFrom<String> for NonEmpty {
    type Error = NonEmptyStringError;

    fn try_from(s: String) -> Result<NonEmpty, NonEmptyStringError> {
        if s.is_empty() {
            Err(NonEmptyStringError)
        } else {
            Ok(NonEmpty { s })
        }
    }
}

fn main() {
    let res: Result<NonEmpty, NonEmptyStringError> = "".to_string().try_into();
    match res {
        Ok(ne) => {
            println!("Converted!");
        }
        Err(err) => {
            println!("Couldn't convert");
        }
    }
}
```

## Implicit conversions

Rust does have one kind of user-defined implicit conversion, called [deref
coercions](https://doc.rust-lang.org/std/ops/trait.Deref.html#deref-coercion),
provided by the [`Deref`
trait](https://doc.rust-lang.org/std/ops/trait.Deref.html) and
[`DerefMut`trait](https://doc.rust-lang.org/std/ops/trait.DerefMut.html). These
coercions exist for making pointer-like types more ergonomic to use.

An [example](https://doc.rust-lang.org/book/ch15-02-deref.html) of implementing
the traits for a custom pointer-like type is given in the Rust book.

## Summary

A summary of when to use which kind of conversion interface is given in the
documentation for the [`std::convert`
module](https://doc.rust-lang.org/std/convert/index.html).
