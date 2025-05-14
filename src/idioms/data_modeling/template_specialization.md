# Template specialization

Template specialization in C++ makes it possible for a template entity to have
different implementations for different parameters. Most STL implementations
make use of this to, for example, provide a [space-efficient representation of
`std::vector<bool>`](https://en.cppreference.com/w/cpp/container/vector_bool).

In Rust it is not possible to specialize a generic implementation in this way.
This is because unlike C++ templates, Rust's generics are not a mechanism for
metaprogramming.

Instead it is more common in Rust to define traits for interfaces and to
implement generic functions over those interfaces, enabling clients to select
their choice of representation by using a different concrete type. This is more
practical to do in Rust than in C++ because generics not being a metaprogramming
facility means that generic entities can be type checked locally, making them
easier to define.

Additionally, because generic functions can only interact with generic values in
ways defined by the trait bounds, it is easier to test generic implementations.
In particular, code testing a generic implementation only has to consider the
possible behaviors of the given trait.

For a comparison, consider the following programs.

<div class="comparison">

```cpp
template <totally_ordered T>
T max(const T &x, const T &y) {
  return (x > y) ? x : y;
}

template <>
int max(const int &x, const int &y) {
  return (x > y) ? x + 1 : y + 1;
}
```

```rust
fn max<'a, T: Ord>(x: &'a T, y: &'a T) -> &'a T {
    if x > y {
        x
    } else {
        y
    }
}
```

</div>

In the Rust program, _parametricity_ means that (assuming safe Rust) from the
type alone one can tell that if the function returns, it must return exactly one
of `x` or `y`. This is because the trait bound `Ord` doesn't give any way to
construct new values of type `T`, and the use of references doesn't give any way
for the function to store one of `x` or `y` from an earlier call to return in a
later call.

In the C++ program, a call to `max` with `int` as the template parameter will
give a distinctly different result than with any other parameter because of the
template specialization enabling the behavior of the function to vary based on
the type.

The trade-off is that in Rust specialized implementations are harder to use
because they must have different names, but that they are easier to write
because it is easier to write generic code while being confident about its
correctness.

## Niche optimization

There are several cases where the Rust compiler will perform optimizations to
achieve more efficient representations. Those situations are all ones where the
efficiency gains do not otherwise change the observable behavior of the code.

[The most common case is with the `Option`
type](https://doc.rust-lang.org/std/option/index.html#representation). When
`Option` is used with a type where the compiler can tell that there are unused
values, one f those unused values will be used to represent the `None` case, so
that `Option<T>` will not require an extra word of memory to indicate the
discriminant of the enum.

This optimization is applied to reference types (`&` and `&mut`), since
references cannot be null. It is also applied to `NonNull<T>`, which represents
a non-null pointer to a value of type `T`, and to `NonZeroU8` and other non-zero
integral types. The optimization for the reference case is what makes
`Option<&T>` and `Option<&mut T>` safer equivalents to using non-owning
observation pointers in C++.
