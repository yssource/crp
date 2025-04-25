# Multiple return values

One idiom for returning multiple values from a function or method in C++ is to
pass in references to which the values can be assigned.

```cpp
void get_point(int &x, int &y) {
  x = 5;
  y = 6;
}

int main() {
  int x, y;
  get_point(x, y);
  // ...
}
```

There are several reasons why this idiom might be used:

- compatibility with versions of C++ earlier than C++11,
- working in a codebase that uses C-style of C++, or
- performance concerns.

The idiomatic translation of this program into Rust makes use of either
[tuples](https://doc.rust-lang.org/std/primitive.tuple.html) or a named
structure for the return type.

```rust
fn get_point() -> (i32, i32) {
    (5, 6)
}

fn main() {
    let (x, y) = get_point();
    // ...
}
```

Rust has a dedicated tuple syntax and supports pattern matching with `let`
bindings in part to support use cases like this one.

## Problems with the direct transliteration

It is possible to transliterate the original example that uses out parameters to
Rust, but Rust requires the initialization of the variables before they can be
passed to a function. The resulting program is not idiomatic Rust.

```rust
// NOT IDIOIMATIC RUST
fn get_point(x: &mut i32, y: &mut i32) {
    *x = 5;
    *y = 6;
}

fn main() {
    let mut x = 0; // initliazed to arbitrary values
    let mut y = 0;
    get_point(&mut x, &mut y);
    // ...
}
```

This approach requires assigning arbitrary initial values to the variables and
making the variables mutable, both of which make it harder for the compiler to
help with avoiding programming errors.

Additionally, the Rust compiler is tuned for optimizing the idiomatic version of
the program, and produces a significantly faster binary for that version.

In situations where the performance of memory allocation is a concern (such as
when it is necessary to reuse entire buffers in memory), the trade-offs may be
different. That situation is discussed in the chapter on [pre-allocated
buffers](/idioms/out_params/pre-allocated_buffers.md).

## Similarities with idiomatic C++ since C++11

In C++11 and later, `std::pair` and `std::tuple` are available for returning
multiple values instead of assigning to reference parameters.

```cpp
#include <tuple>
#include <utility>

std::pair<int, int> get_point() {
  return std::pair<int, int>(5, 6);
}

int main() {
  int x, y;
  std::tie(x, y) = get_point();
  // ...
}
```

This more closely aligns with the normal Rust idiom for returning multiple
values.
