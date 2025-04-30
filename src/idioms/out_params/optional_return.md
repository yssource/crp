# Optional return values

One idiom in C++ for optionally producing a result from a method or function is
to use a reference parameter along with a boolean or integer return value to
indicate whether the result was produced. This might be done for the same
reasons as for using [out parameters for multiple return
values](/idioms/out_params/multiple_return.md):

- compatibility with versions of C++ earlier than C++11,
- working in a codebase that uses C-style of C++, and
- performance concerns.

The idiomatic Rust approach for optionally returning a value is to return a
value of type [`Option`](https://doc.rust-lang.org/std/option/index.html).

<div class="comparison">

```cpp
#include <iostream>

bool safe_divide(unsigned int dividend,
                 unsigned int divisor,
                 unsigned int &quotient) {
  if (divisor != 0) {
    quotient = dividend / divisor;
    return true;
  } else {
    return false;
  }
}

void go(unsigned int dividend,
        unsigned int divisor) {
  unsigned int quotient;
  if (safe_divide(dividend, divisor, quotient)) {
    std::cout << quotient << std::endl;
  } else {
    std::cout << "Division failed!" << std::endl;
  }
}

int main() {
  go(10, 2);
  go(10, 0);
}
```

```rust
fn safe_divide(
    dividend: u32,
    divisor: u32,
) -> Option<u32> {
    if divisor != 0 {
        Some(dividend / divisor)
    } else {
        None
    }
}

fn go(dividend: u32, divisor: u32) {
    match safe_divide(dividend, divisor) {
        Some(quotient) => {
            println!("{}", quotient);
        }
        None => {
            println!("Division failed!");
        }
    }
}

fn main() {
    go(10, 2);
    go(10, 0);
}
```

</div>

When there is useful information to provide in the failing case, the [`Result`
type](https://doc.rust-lang.org/std/result/) can be used instead. The [chapter
on error handling](/idioms/exceptions.md) describes the use of `Result`.

## Returning a pointer

When the value being returned is a pointer, another common idiom in C++ is to
use `nullptr` to represent the optional case. In the Rust translation of that
idiom, `Option` is also used, along with a reference type, such as `&` or `Box`.
See [the chapter on using `nullptr` as a sentinel
value](/idioms/null/sentinel_values.md#nullptr) for more details.

## Problems with the direct transliteration

It is possible to transliterate the original example that uses out parameters to
Rust, but the resulting code is not idiomatic.

```rust
// NOT IDIOIMATIC RUST
fn safe_divide(dividend: u32, divisor: u32, quotient: &mut u32) -> bool {
    if divisor != 0 {
        *quotient = dividend / divisor;
        true
    } else {
        false
    }
}

fn go(dividend: u32, divisor: u32) {
    let mut quotient: u32 = 0; // initliazed to arbitrary value
    if safe_divide(dividend, divisor, &mut quotient) {
        println!("{}", quotient);
    } else {
        println!("Division failed!");
    }
}

fn main() {
    go(10, 2);
    go(10, 0);
}
```

This shares the same problems as with using out-parameters for [multiple return
values](/idioms/out_params/multiple_return.md#problems-with-the-direct-transliteration).

## Similarities with C++ since C++17

C++17 and later offer `std::optional`, which can be used to express optional
return values in a way similar to the idiomatic Rust example.

```cpp
#include <iostream>
#include <optional>

std::optional<unsigned int> safe_divide(unsigned int dividend,
                                        unsigned int divisor) {
  if (divisor != 0) {
    return std::optional<unsigned int>(dividend / divisor);
  } else {
    return std::nullopt;
  }
}

void go(unsigned int dividend, unsigned int divisor) {
  if (auto quotient = safe_divide(dividend, divisor)) {
    std::cout << *quotient << std::endl;
  } else {
    std::cout << "Division failed!" << std::endl;
  }
}

int main() {
  go(10, 2);
  go(10, 0);
}
```

## Helpful `Option` utilities

When the `None` case really is an edge case, the [`let-else`
syntax](https://doc.rust-lang.org/rust-by-example/flow_control/let_else.html)
can be used to make that clearer:

```rust
# fn safe_divide(dividend: u32, divisor: u32) -> Option<u32> {
#     if divisor != 0 {
#         Some(dividend / divisor)
#     } else {
#         None
#     }
# }
#
fn go(dividend: u32, divisor: u32) {
    let Some(quotient) = safe_divide(dividend, divisor) else {
        println!("Division failed!");
        return;
    };
    println!("{}", quotient);
}
#
# fn main() {
#     go(10, 2);
#     go(10, 0);
# }
```

If there is a default value that should be used in the `None` case, the
[`Option::unwrap_or`](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or),
[`Option::unwrap_or_else`](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or_else),
[`Option::unwrap_or_default`](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or_default),
or
[`Option::unwrap`](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap)
methods can be used:

```rust
# fn safe_divide(dividend: u32, divisor: u32) -> Option<u32> {
#     if divisor != 0 {
#         Some(dividend / divisor)
#     } else {
#         None
#     }
# }
#
fn expensive_computation() -> u32 {
    // ...
#    0
}

fn go(dividend: u32, divisor: u32) {
    // If None, returns the given value.
    let result = safe_divide(dividend, divisor).unwrap_or(0);

    // If None, returns the result of calling the given function.
    let result2 = safe_divide(dividend, divisor).unwrap_or_else(expensive_computation);

    // If None, returns Default::default(), which is 0 for u32.
    let result3 = safe_divide(dividend, divisor).unwrap_or_default();

    // If None, panics. Prefer the other methods!
    // let result3 = safe_divide(dividend, divisor).unwrap();
}
#
# fn main() {
#     go(10, 2);
#     go(10, 0);
# }
```

In performance-sensitive code where you have manually checked that the result is
guaranteed to be `Some`,
[`Option::unwrap_unchecked`](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_unchecked)
can be used, but is an unsafe method.

There are [additional utility
methods](https://doc.rust-lang.org/std/option/#boolean-operators) that enable
concise handling of `Option` values, which this book covers in the chapter on
[exceptions and error handling](/idioms/exceptions.md).

## An alternative approach

An alternative approach in Rust to returning optional values is to require that
the caller of a function prove that the value with which they call a function
will not result in the failing case.

For the above safe division example, this involves the caller guaranteeing that
the provided divisor is non-zero. In the following example this is done with a
dynamic check. In other contexts the evidence needed may be available
statically, provided from callers further upstream, or used more than once. In
those cases, this approach reduces both runtime cost and code complexity.

```rust
use std::convert::TryFrom;
use std::num::NonZero;

fn safe_divide(dividend: u32, divisor: NonZero<u32>) -> u32 {
    // This is more efficient because the overflow check is skipped.
    dividend / divisor
}

fn go(dividend: u32, divisor: u32) {
    let Ok(safe_divisor) = NonZero::try_from(divisor) else {
        println!("Can't divide!");
        return;
    };

    let quotient = safe_divide(dividend, safe_divisor);
    println!("{}", quotient);
}

fn main() {
    go(10, 2);
    go(10, 0);
}
```
