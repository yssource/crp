# Copy and move constructors

In both C++ and Rust you will rarely have to write copy or move constructors (or
their Rust equivalents) by hand. In C++ this is because the implicit definitions
are good enough for most purposes, especially if you make use of smart pointers.
(I.e., if you follow [the rule of
zero](https://en.cppreference.com/w/cpp/language/rule_of_three).) In Rust this
is because move semantics are the default, and the automatically derived
implementations of the `Clone` and `Copy` traits are good enough for most
purposes.

For the following C++ classes, the implicitly defined copy and move constructors
are sufficient.

```c++
#include <string>
#include <memory>

struct Age {
  unsigned int years;

  Age(unsigned int years) : years(years) {}

  // copy and move constructors and destructor implicilty declared and defined
};

struct Person {
  Age age;
  std::string name;
  std::shared_ptr<Person> best_friend;

  Person(Age age, std::string name, std::shared_ptr<Person> best_friend)
      : age(age), name(name), best_friend(best_friend) {}

  // copy and move constructors and destructor implicilty declared and defined
};
```

The equivalent in Rust is

```Rust
use std::rc::Rc;

#[derive(Clone, Copy)]
struct Age {
    years: u32,
}

#[derive(Clone)]
struct Person {
    age: Age,
    name: String,
    best_friend: Rc<Person>,
}
```

On the other hand, the following example require a user-defined copy and move
constructor because it manages a resource (a pointer acquired from a C library).

```c++
#include <cstdlib>
#include <cstring>

// widget.h
struct widget_t;
widget_t *alloc_widget();
void free_widget(widget_t*);
void copy_widget(widget_t* dst, widget_t* src);

// widget.cc
class Widget {
    widget_t* widget;
public:
    Widget() : widget(alloc_widget()) {}

    Widget(const Widget &other) : widget(alloc_widget()) {
        copy_widget(widget, other.widget);
    }

    Widget(Widget &&other) : widget(other.widget) {
        other.widget = nullptr;
    }

    ~Widget() {
        free_widget(widget);
    }
};
```

The equivalent in Rust is:

```rust
mod widget_ffi {
    // Models an opaque type. See https://doc.rust-lang.org/nomicon/ffi.html#representing-opaque-structs
    #[repr(C)]
    pub struct CWidget {
        _data: [u8; 0],
        _marker: core::marker::PhantomData<(*mut u8, core::marker::PhantomPinned)>,
    }

    extern "C" {
        pub fn make_widget() -> *mut CWidget;
        pub fn copy_widget(dst: *mut CWidget, src: *mut CWidget);
        pub fn free_widget(ptr: *mut CWidget);
    }
}

use widget_ffi::*;

struct Widget {
    widget: *mut CWidget,
}

impl Widget {
    fn new() -> Self {
        Widget {
            widget: unsafe { make_widget() },
        }
    }
}

impl Clone for Widget {
    fn clone(&self) -> Self {
        let widget = unsafe { make_widget() };
        unsafe {
            copy_widget(widget, self.widget);
        }
        Widget { widget }
    }
}

impl Drop for Widget {
    fn drop(&mut self) {
        unsafe { free_widget(self.widget) };
    }
}
```

Just as with how in C++ it is uncommon to need user-defined implementations for
copy and move constructors or user-defined implementations for destructors, in
Rust it is rare to need to implement the `Clone` and `Drop` traits by hand.

There is one exception to this. If the type has type parameters, you might want
to implement `Clone` (and `Copy`) manually even if the clone should be done
field-by-field. See the [standard library documentation of
`Clone`](https://doc.rust-lang.org/std/clone/trait.Clone.html#how-can-i-implement-clone)
and [of
`Copy`](https://doc.rust-lang.org/std/marker/trait.Copy.html#how-can-i-implement-copy)
for details.

## Trivially copyable types

Rust indicates whether types are trivially copyable with the `Copy` trait. Just
as with trivially copyable types in C++, types that implement `Copy` in Rust can
be copied bit-for-bit. Rust requires explicit calls to the `clone` method to
make copies of values of types that do not implement `Copy`.

Rust will prevent you from implementing `Copy` for a type if any of its fields
are not `Copy`, but will not prevent you from implementing `Copy` for types
which should not be copied bit-for-bit due to the intended meaning of the type
(which is usually reflected by a user-defined `Clone` implementation). Rust will
also not allow you to implement both `Copy` and `Drop` for the same type (which
matches the C++ standard's requirement that trivially copyable types not
implement a user-defined destructor).

Notice, for example that in the first Rust example, `Age` implements the `Copy`
trait but `Person` does not. This is because neither `std::String` nor
`Rc<Person>` implement `Copy`. They do not implement `Copy` because they own
data that lives on the heap, and so are not trivially copyable.

## Move constructors

In Rust, all types support move semantics by default, and custom move semantics
cannot be (and do not need to be) defined. This is because what "move" means in
Rust is not the same as it is in C++. In Rust, moving a value means changing
what owns the value. In particular, there is no "old" object to be destructed
after a move, because the compiler will prevent you from using a variable whose
value has been moved.

```rust
#[derive(Clone)]
struct Buffer {
    len: usize,
    buffer: Box<[u8]>,
}

impl Buffer {
    fn new(len: usize) {
        let other_buffer: Box<[u8]> = vec![0; self.len].into_boxed_slice();
    }
}

impl Clone for Buffer {
    fn clone(&self) -> Self {
        Buffer {
            buffer_len: self.buffer_len,
            buffer: self.buffer.clone(),
        }
    }
}
```

## Assignment operators

Rust does not have a copy or move assignment operator. Instead, assignment
either moves (by transferring ownership), explicitly clones and then moves, or
implicitly copies and then moves.

```rust
fn go() {
    let x = Box::<u32>::new(5);
    let y = x; // moves
    let z = y.clone(); // explicitly clones and then moves the clone
    let w = *y; // implicitly copies the content of the Box and then moves the copy
}
```

For situations where something like a user-defined copy assignment could avoid
allocations, the `Clone` trait has an additional method called `clone_from`.
The method is usually automatically defined, but can be overridden when
implementing the `Clone` trait.

The method is not used for normal assignments, but can be explicitly used in
situations where the performance of the assignment is significant and would be
improved.

```rust
fn go(x: &Vec<u32>) {
    let y = vec![0; x.len()];
    // ...
    y.clone_from(&x);
    // ...
}
```

## Performance concerns and `Copy`

The decision to implement `Copy` should be based on the semantics of the type,
not on performance. If you are worried about the size of objects being copied,
then use a reference (`&T` or `&mut T`) or put it on the heap (`Box<T>`). These
situations correspond to passing by reference or using a `shared_ptr` in C++.
