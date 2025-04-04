# Zero-length arrays

In C++ codebases that are written in a C style or that make use of C libraries,
null pointers may be used to represent empty arrays. This is because there is
little practical difference between an array of size zero and a null pointer.

```c++
#include <cstddef>
#include <cassert>

int c_style_sum(std::size_t len, int arr[]) {
    int sum = 0;
    for (size_t i = 0; i < len; i++) {
        sum += arr[i];
    }
    return sum;
}

int main() {
    int sum = c_style_sum(0, nullptr);
    assert(sum == 0);
    return 0;
}
```

In Rust, arrays of arbitrary size are represented as
[slices](https://doc.rust-lang.org/book/ch04-03-slices.html). These slices can
have zero length. Since [Rust vectors are convertible to
slices](https://doc.rust-lang.org/std/vec/struct.Vec.html#impl-Deref-for-Vec%3CT,+A%3E),
defining functions that work with slices enables them to be used with vectors as
well.

```rust
fn sum_slice(arr: &[i32]) -> i32 {
    let mut sum = 0;
    for x in arr {
        sum += x;
    }
    sum
}

fn main() {
    let sum = sum_slice(&[]);
    assert!(sum == 0);

    let sum2 = sum_slice(&vec![]);
    assert!(sum2 == 0);
}
```

