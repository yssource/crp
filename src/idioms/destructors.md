# Destructors and resource cleanup

In C++, destructors are defined by providing a special member function. To
achieve the equivalent in Rust, implement the [`Drop`
trait](https://doc.rust-lang.org/std/ops/trait.Drop.html).

For an example, see [the chapter on copy and move
constructors](/idioms/constructors/copy_and_move_constructors.md#user-defined-constructors).

`Drop` implementations play the same role as destructors in C++ for types that
manage resources. That is, they enable cleanup of resources owned by the value
at the end of the value's lifetime (i.e., [RAII](/idioms/raii.md)).

In Rust the `Drop::drop` method is called the "destructor", but we will refer to
it as "the drop method" here, to clearly distinguish between it and C++
destructors.

## Lifetimes and destructors

C++ destructors are called in reverse order of construction when variables go out
of scope, of for dynamically allocated objects, when they are deleted. Because
of how move constructors work, this includes destructors of moved objects.

```cpp
#include <iostream>
#include <utility>

struct A {
    int id;

    A(int id) : id(id) {}

    // copy constructor
    A(A& other) : id(other.id) {}

    // move constructor
    A(A&& other) : id(other.id) {
        other.id = 0;
    }

    // deconstructor
    ~A() {
        std::cout << id << std::endl;
    }
};

int accept(A x) {
    return x.id;
} // the destructor of x is called after the return expression is evaluated

// Prints:
// 2
// 3
// 0
// 1
int main() {
    A x(1);
    A y(2);

    accept(std::move(y));

	A z(3);

    return 0;
}
```

In Rust, the drop order is similar to that of C++ (reverse order of
declaration). If additional specific details about the drop order are needed
(e.g., for writing unsafe code), the full rules for the drop order are described
in [the language
reference](https://doc.rust-lang.org/reference/destructors.html).

```rust
struct A {
    id: i32,
}

impl Drop for A {
    fn drop(&mut self) {
        println!("{}", self.id)
    }
}

fn accept(x: A) -> i32 {
    return x.id;
}

// Prints:
// 2
// 3
// 1
fn main() {
    let x = A { id: 1 };
    let y = A { id: 2 };

    accept(y);

    let z = A { id: 3 };
}
```

One particular difference between C++ and Rust is that after ownership of `y` is
moved into the function `acccept`, there is no additional object remaining, and
so there is no additional `Drop::drop` call (which in the C++ example prints `0`).

Rust's drop methods do run when leaving scope due to a panic, though not if the
panic occurs in a destructor that was called in response to an initial panic.

## Early cleanup and explicitly destroying values

In C++ you can explicitly destroy an object. This is mainly useful for
situations where placement new has been used to allocate the object at a
specific memory location, and so the destructor will not be implicitly called.

However, once the destructor has been explicitly called, [it may not be called
again, even implicitly](https://eel.is/c++draft/class.dtor#note-8). Thus the
destructor can't be used for early cleanup. Instead, either the class must be
designed with a separate cleanup method that releases the resources but leaves
the object in a state where the destructor can be called or the function using
the object must be structured so that the variable goes out of scope at the
desired time.

In Rust, values can be dropped early for early cleanup by using
[`std::mem::drop`](https://doc.rust-lang.org/std/mem/fn.drop.html). This works
because ([for non-`Copy`
types](/idioms/constructors/copy_and_move_constructors.md#trivially-copyable-types))
ownership of the object is actually transferred to `std::mem::drop` function,
and so `Drop::drop` is called at the end of `std::mem::drop`.

Thus, `std::mem::drop` can be used for early cleanup of resources without having
to restructure a function to force variables out of scope early.

For example, the following allocates a large vector on the heap, but explicitly
drops it before allocating a second large vector on the heap, reducing the
overall memory usage.

```rust
fn main() {
    let v = vec![0u32; 100000];
    // ... use v

    std::mem::drop(v);
    // can no longer use v here

    let v2 = vec![0u32; 100000];
    // ... use v2
}
```
