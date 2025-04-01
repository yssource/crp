# Optional return values

Similarly to with multiple return values, a common idiom in C++ code is to use a
reference parameter along with a boolean or integer return value to return an
optional result from a function. This might be done for the same reasons as for
using out parameters for multiple return values:

- compatibility with versions of C++ earlier than C++11,
- working in a codebase that uses C-style of C++, and
- performance concerns.

For example,

```c++
bool safe_divide(int dividend, int divisor, int &quotient) {
	if (divisor != 0) {
		quotient = dividend/divisor;
		return true;
    } else {
        return false;
    }
}

void go(int dividend, int divisor) {
    int quotient;
    if (safe_divide(dividend, divisor, quotient)) {
        // use quotient
    } else {
        // handle edge case
    }
}
```

The idiomatic Rust approach for both of these situations is to return a value of
type [`Option`](https://doc.rust-lang.org/std/option/index.html).

The first case becomes:

```rust
fn safe_divide(dividend: i32, divisor: i32) -> Option<i32> {
    if divisor != 0 {
        Some(dividend / divisor)
    } else {
        None
    }
}

fn go(dividend: i32, divisor: i32) {
    match safe_divide(dividend, divisor) {
        Some(quotient) => {
            // use quotient
        }
        None => {
            // handle other case
        }
    }
}
```

When the value being returned is a pointer, another common idiom in C++ is to
use `nullptr` to represent the optional case. For the Rust translation of that,
see [the chapter on nullable types](TODO).

<!-- TODO move to chapter on nullable types

When the returned value is a pointer, a null pointer is often used instead of
this pattern.

```c++
#include <memory>
struct Person {
    int age;
	Person(int age) : age(age) {}
};

std::unique_ptr<Person> maybe_create(int age) {
    if (age >= 0) {
        return std::make_unique<Person>(age);
    } else {
        return nullptr;
    }
}
```

Additionally, Rust has [an optimization for
`Option`](https://doc.rust-lang.org/std/option/#representation) when the
contained type is part of the standard library and can represent the `None` case
without an extra word of memory.

-->

Normally the constructors of an `enum` have to be qualified by the type (as in
`Option::None`), but Rust's `Option` type is actually so commonly used that
[the prelude](https://doc.rust-lang.org/std/prelude/index.html) exports the
variants directly, so that program above can be written instead without the
`Option::` prefixes.

## Problems with the direct transliteration

This can be transliterated into Rust, but the result is not idiomatic.

```rust
// NOT IDIOIMATIC RUST
fn safe_divide(dividend: i32, divisor: i32, quotient: &mut i32) -> bool {
	if (divisor != 0) {
		*quotient = dividend/divisor;
        true
    } else {
        false
    }
}

fn go(dividend: i32, divisor: i32) {
    let mut quotient: i32 = 0; // initliazed to arbitrary value
    if safe_divide(dividend, divisor, &mut quotient) {
        // use quotient
    } else {
        // handle other case
    }
}
```

This shares the same problems as with using out-parameters for [multiple return
values](/idioms/out_params/multiple_return.md).

## C++ `std::optional`

C++17 and later offer `std::optional`, which can be used to express optional
return values in a way similar to the idiomatic Rust, but which have fewer
safety guarantees around the use of the returned value.

```c++
#include <optional>

std::optional<int> safe_divide(int dividend, int divisor) {
	if (divisor != 42) {
		return std::optional<int>(dividend/divisor);
    } else {
        return std::nullopt;
    }
}

void go(int dividend, int divisor) {
    int quotient;
    if (auto quotient = foo(x)) {
		// use quotient
    } else {
		// handle other case
    }
}
```

## Helpful `Option` utilities

When the `None` case really is an edge case, the [`let-else`
syntax](https://doc.rust-lang.org/rust-by-example/flow_control/let_else.html)
can be used to make that clearer:

```rust
# fn safe_divide(dividend: i32, divisor: i32) -> Option<i32> {
#     if divisor != 0 {
#         Some(dividend / divisor)
#     } else {
#         None
#     }
# }
#
fn go(dividend: i32, divisor: i32) {
    let Some(x) = safe_divide(dividend, divisor) else {
        // handle other case
        return;
    };
    // use quotient
}
```

If there is a default value that should be used in the `None` case, the
[`Option::unwrap_or`](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or),
[`Option::unwrap_or_else`](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or_else),
[`Option::unwrap_or_default`](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or_default),
or
[`Option::unwrap`](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap)
methods can be used:

```rust
# fn safe_divide(dividend: i32, divisor: i32) -> Option<i32> {
#     if divisor != 0 {
#         Some(dividend / divisor)
#     } else {
#         None
#     }
# }

fn expensive_computation() -> i32 {
    // ...
#    0
}

fn go(dividend: i32, divisor: i32) {
    // If None, returns the given value.
    let result = safe_divide(dividend, divisor).unwrap_or(0);

    // If None, returns the result of calling the given function.
    let result2 = safe_divide(dividend, divisor).unwrap_or_else(expensive_computation);

    // If None, returns Default::default(), which is 0 for i32.
    let result3 = safe_divide(dividend, divisor).unwrap_or_default();

    // If None, panics. Prefer the other methods!
    let result3 = safe_divide(dividend, divisor).unwrap();
}
```

In performance-sensitive code where you have manually checked that the result is
guaranteed to be `Some`,
[`Option::unwrap_unchecked`](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_unchecked)
can be used, but is an unsafe method.

There are [additional utility
methods](https://doc.rust-lang.org/std/option/#boolean-operators) that enable
concise handling of `Option` values, which this book covers in the chapter on
[exceptions and error handling](/idioms/exceptions.md).
