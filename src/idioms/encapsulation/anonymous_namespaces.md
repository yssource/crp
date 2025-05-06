# Anonymous namespaces and `static`

Anonymous namespaces in C++ are used to avoid symbol collisions between
different translation units. Such collisions violate [the one definition
rule](https://timsong-cpp.github.io/cppwp/n4950/basic.def.odr#14) and result in
undefined behavior (which at best manifests as linking errors).

For example, without the use of anonymous namespaces, the following would result
in undefined behavior (and no linking error, due to the use of `inline` producing
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

Rust avoids the linkage problem by controlling linkage and visibility
simultaneously, with declarations always also being definitions. Instead of
translation units, programs are structured in terms of
[modules](/idioms/encapsulation/headers.md), which provide both namespaces and
visibility controls over definitions, enabling the Rust compiler to guarantee
that symbol collision issues cannot happen.

The following Rust program achieves the same goal as the C++ program above, in
terms of avoiding the collision of the two functions while making them available
for use within the defining files.

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

Additionally,

1. Unlike C++ namespaces, Rust modules (which provide namespacing as well as
   visibility controls) can only be defined once, and this is checked by the
   compiler.
2. Each file [defines a module which has to be explicitly included in the module
   hierarchy](https://doc.rust-lang.org/stable/book/ch07-05-separating-modules-into-different-files.html).
2. Modules from Rust crates (libraries) are always qualified with some root
   module name, so they cannot conflict. If they would conflict, [the root
   module name must be replaced with some user-chosen
   name](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#renaming-dependencies-in-cargotoml).

## Caveats about C interoperability

When using libraries not managed by Rust, the usual problems can occur if there are symbol collisions in the
object files. This can arise when using C or C++ static or dynamic libraries. 
It can also arise when using Rust static or dynamic libraries built for use in C or
C++ programs.

Rust provides [`#[unsafe(no_mangle)]`](https://doc.rust-lang.org/reference/abi.html#the-no_mangle-attribute) to bypass name mangling
in order to produce functions that can be easily
referred to from C or C++. This can also cause undefined behavior due to name collision.

{{#quiz anonymous_namespaces.toml}}
