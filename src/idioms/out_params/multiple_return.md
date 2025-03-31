# Multiple return values

One idiom for returning multiple values in C++ is to pass in references to which
the values can be assigned. E.g.,

```c++
void get_point(int &x, int &y) {
    x = 5;
    y = 6;
}

int compute_norm() {
    int x, y;

    get_point(x, y);

    return x + y;
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
pub fn get_point() -> (i32, i32) {
    (5, 6)
}

pub fn compute_norm() -> i32 {
    let (x, y) = get_point();
    x + y
}
```

Rust has a dedicated tuple syntax and pattern matching with `let` bindings in
part to support use cases like this one.

## The problems with the direct transliteration

It is possible to transliterate this to Rust, but Rust requires the
initialization of the variables. The resulting program is not idiomatic Rust.

```rust
// NOT IDIOIMATIC RUST
fn get_point(x: &mut i32, y: &mut i32) {
    *x = 5;
    *y = 6;
}

fn compute_norm() -> i32 {
    let mut x = 0; // initliazed to arbitrary values
    let mut y = 0;

    get_point(&mut x, &mut y);

    x + y
}
```

In addition to sacrificing conciseness, this approach requires assigning
arbitrary initial values to the variables and making the variables mutable, both
of which make it harder for the compiler to help with avoiding programming
errors.

Additionally, the Rust compiler is tuned for optimizing the idiomatic version of
the program, and does produce a significantly faster binary for that version.

## Similarities with idiomatic C++ since C++11

In C++11 and later, it `std::pair` and `std::tuple` are available for returning
multiple values instead of assigning to reference parameters.

```c++
#include <utility>
#include <tuple>

std::pair<int,int> get_point() {
	return std::pair<int,int>(5, 6);
}

int compute_norm() {
	int x, y;
	std::tie(x, y) = get_point();

    return x + y;
}
```

This more closely aligns with the normal Rust idiom for returning multiple
values.
