# Errors indicating bugs

In C++, exceptions are sometimes used to indicate an error that is due to a
programming bug. In many situations no exception is produced, and instead the
invalid use of an API is simply undefined behavior.

In Rust, `panic!` is used for these kinds of errors, often via an
[`expect`](https://doc.rust-lang.org/std/result/enum.Result.html#method.expect)
or
[`unwrap`](https://doc.rust-lang.org/std/result/enum.Result.html#method.unwrap)
method on `Result` or `Option` or via [assertions like `assert!`](#assert).
While a panic in Rust may unwind the stack or abort a program, it is never
undefined behavior.

<div class="comparison">

```cpp
#include <cstddef>
#include <vector>

int main() {
  std::vector<int> v{1, 2, 3};
  // undefined behavior!
  int x(v[4]);
}
```

```rust,no_run
fn main() {
    let v = vec![1,2,3];
    // panics!
    let x = v[4];
}
```

</div>

## Converting `Result` or `Option` to `panic!`

It is easier to convert from a `Result` or `Option` to a panic than to go the
other way around. Therefore, many libraries in Rust are written to return
`Result` or `Option` and allow the caller to determine whether a `None` result
indicates a bug by using `unwrap` or `expect` to extract the value, panicking if
there isn't one.

```rust,should_panic
/// Returns `None` if the number cannot be divided evenly.
fn divide_exact(dividend: i32, divisor: i32) -> Option<i32> {
    let quotient = dividend / divisor;
    if quotient * divisor == dividend {
        Some(quotient)
    } else {
        None
    }
}

// Returns `None` if the number cannot be divided by 2
fn divide_by_two_exact(dividend: i32) -> Option<i32> {
    // divide_exact returning None here isn't a bug
    divide_exact(dividend, 2)
}

fn main() {
    let res = divide_exact(10, 3); // Oops, a bug!
    let x = res.unwrap();
    // ...
}
```

When designing an API, if only one of a `Result`-based (or `Option`-based) or
panicking interface is going to be offered, it is generally better to offer the
`Result`-based interface. That way that the caller can choose to omit the
pre-condition checks and handle the error instead or to panic because
pre-conditions should have been met.
