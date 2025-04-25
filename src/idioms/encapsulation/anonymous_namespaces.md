# Anonymous namespaces and `static`

Anonymous namespaces are used in C++ to prevent symbols defined in one
translation unit from colliding with those defined in another, which would
result in a linking error at best and possibly undefined behavior at runtime.

For example, without the use of anonymous namespaces, the following would result
in undefined behavior (and no linking error, due to the use of inline producing
weak symbols in the object files).

```cpp
/// a.cc
namespace {
    inline void common_function_name() {
        // ...
    }
}

/// b.cc
namespace {
    inline void common_function_name() {
        // ...
    }
}
```

C++ static declarations are also used to achieve the same goal by making it so that
a declaration has internal linkage (and so is not visible outside of the
translation unit).

Thus, the following Rust program achieves the same goal as the C++ program above
in terms of avoiding the collision of the two functions while making them
available for use within the defining files.

```rust
// a.rs
# mod a {
fn common_function_name() {
    // ...
}
# }

// b.rs
# mod b {
fn common_function_name() {
    // ...
}
# }
```

Rust avoids the linkage problem by controlling linkage and visibility
simultaneously, with declarations always also being definitions. Instead of
translation units, programs are structured in terms of
[modules](/idioms/encapsulation/headers.md), which provide both namespaces and
visibility controls over definitions, enabling the Rust compiler to guarantee
that symbol collision issues cannot happen.

Additionally,

1. Unlike C++ namespaces, Rust modules (which provide namespacing as well as
   visibility controls) can only be defined once, and this is checked by the
   compiler.
2. Each file [defines a module which has to be explicitly included in the module
   hierarchy](https://doc.rust-lang.org/stable/book/ch07-05-separating-modules-into-different-files.html).
2. Modules from Rust crates (libraries) are always qualified with some root
   module name, so they cannot conflict. If the would conflict, [the root module
   name must be replaced with some user-chosen
   name](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#renaming-dependencies-in-cargotoml).

## A caveat about using C and C++ libraries

When using libraries not managed by Rust, such as C or C++ static or dynamic
libraries or when producing Rust static or dynamic libraries to be used by C or
C++ programs, the usual problems can occur if there are symbol collisions in the
object files.
