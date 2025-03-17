# Default constructors

## Default constructors in C++

C++ has a special concept of default constructors to support several scenarios
where they are implicitly called. For example,

- Initialization of class members without explicit initialization,

  ```c++
  class A {
    int m;

  public:
    A() : m(42) {}
  }

  class S {
    A a;
  }
  ```

- Array declarations without explicit initialization of values,

  ```c++
  class A {
    int m;

  public:
    A() : m(42) {}
  }

  void f() {
    A many_a[3];
    // ...
  }
  ```

- Initialization of the base class object when no other constructor is specified,

  ```c++
  class Base {
  }

  class Derived : Base {
    // default constructor implicitly defined
  }
  ```

- Container element initialization (until C++11) (one element is default
  constructed, and the elements are copy constructed from that initial element):

  ```c++
  #include <vector>

  class A {
    int m;

  public:
    A() : m(42) {}
  }

  void f() {
    std::vector<A> v(3);
    // ...
  }
  ```

- Local variable definitions without explicit initialization,

  ```c++
  class A {
    int m;

  public:
    A() : m(42) {}
  }

  void f() {
    A a;
    // ...
  }
  ```

## Equivalents in Rust

Rust does not have a notion of a default constructor in the same way as in C++.
Some of the uses cases are achieved via a different mechanism or with different
conventions, and others do not apply to Rust.

If a structure has a useful default value (such as would be constructed by a
default constructor in C++), then the type should provide
[both](https://rust-lang.github.io/api-guidelines/interoperability.html?highlight=default#types-eagerly-implement-common-traits-c-common-traits)
a `new` method that takes no arguments and an implementation of the [`Default`
trait](https://doc.rust-lang.org/std/default/trait.Default.html).

```rust
struct A {
   m: u32,
}

impl A {
    pub const fn new() -> Self {
        Self { m: 42 }
    }
}

impl Default for A {
    fn default() -> Self {
        Self::new()
    }
}
```

If all of the members of the structure have an implementation of `Default`, then
an implementation for the structure can be provided by the compiler.

```rust
#[derive(Default)]
struct A {
    m: u32,
}
```

This is equivalent to writing

```rust
struct A {
    m: u32,
}

impl Default for A {
    fn default() -> Self {
        A {
            m: Default::default()
        }
    }
}
```

The default value for the base integer and floating point types is zero.

The `Default` trait can be used to achieve many of the same things that are done
using the default constructor in C++. For example, to initialize a subset of
members of a struct to their default values one can use the [struct update
syntax](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax):

```rust
#[derive(Default)]
struct B {
    a: A,
    b: u32,
    c: u32,
    d: f32,
}

impl B {
    pub fn with_c(c: u32) -> Self {
        B {
            c,
            ..Default::default()
        }
    }
}
```

The trait is also used to support various standard library functions, such as:

- the [implementation of the `Default` trait for the `Box`
  type](https://doc.rust-lang.org/std/boxed/struct.Box.html#impl-Debug-for-Box%3CT,+A%3E),
  which enables the construction of something similar to a `unique_ptr`,
  initialized with a default value.

  ```rust
  fn make_boxed() -> Box<A> {
      Default::default()
  }
  ```

  This is similar to the following in C++:

  ```c++
  #include <memory>

  template <class T> std::unique_ptr<T> default_unique_ptr() {
    return std::unique_ptr<T>(new T());
  }
  ```

- the [implementation of
  `Option::unwrap_or_default`](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or_default),
  which makes getting a default value when the `Option` does not contain a value
  more convenient.

  ```rust
  fn go(x: Option<A>) {
      let a: A = x.unwrap_or_default();
      // ...
  }
  ```

  This is similar to the following in C++:

  ```c++
  #include <optional>

  template <class T> T unwrap_or_default(std::optional<T> x) {
    if (x) {
      return *x;
    } else {
      return T();
    }
  }
  ```

  or in C++ since C++23:

  ```c++
  #include <optional>

  template <class T> T unwrap_or_default(std::optional<T> x) {
    return x.or_else([] { return T(); });
  }
  ```

## Situations with no equivalent

Initialization of collections in Rust requires that a value be provided, in the
same way as in C++ since C++11. Also, much like how in C++ the copy constructor
is used to produce the elements of the vector, in Rust the `Clone`
implementation is used. See the chapter on [copy
constructors](copy_constructors.md) for more details.

```rust
fn foo() {
    let v = vec![A::new(); 3];
    // ...
}
```

Additionally, local variables in Rust must always be explicitly initialized
before use.

```rust
fn foo() {
    let a: A; // this is NOT initialized with Default::<A>::default()
    // ...
}
```

Since Rust structures do not have inheritance, there is not a direct equivalent
to using the default constructor for initializing a base class. However, see the
chapter on [implementation reuse](/idoms/implementation-reuse.html) or the
section on [traits in the Rust
book](https://doc.rust-lang.org/book/ch10-02-traits.html) for alternatives.
