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

class Widget {
    size_t len;
    char* buffer;
public:
    Widget(size_t len) : len(len), buffer((char*)std::calloc(sizeof(char), len)) {}

    Widget(const Widget &other) : len(other.len), buffer((char*)std::malloc(len)) {
        std::memcpy(buffer, other.buffer, len);
    }

    Widget(Widget &&other) : len(other.len), buffer(other.buffer) {
        other.len = 0;
        other.buffer = NULL;
    }

    ~Widget() {
        std::free(buffer);
    }
};
```

The equivalent in Rust is somewhat noisier because it requires the use of
`unsafe` due to handling raw pointers.

```rust
use std::alloc::{Layout, System};

struct Widget {
    len: usize,
    buffer: *mut u8,
}

impl Widget {
    fn layout(len: usize) -> Layout {
        Layout::from_size_align(len, 8).expect("Bad length.")
    }

    pub fn new(len: usize) -> Self {
        let buffer = unsafe {
            let system = System::default();
            let layout = Self::layout(len);
            system.alloc_zeroed(layout)
        };
        Widget { len, buffer }
    }
}

impl Clone for Widget {
    pub fn clone(&self) -> Self {
        let buffer = unsafe {
            let system = System::default();
            let layout = Self::layout(self.len);
            system.alloc(layout)
        };
        unsafe {
            std::ptr::copy_nonoverlapping(self.buffer, buffer, self.len);
        };
        Widget { len: self.len, buffer }
    }
}

impl Drop for  Widget {
    pub fn drop(&mut self) {
        unsafe {
            let system = System::default();
            let layout = Self::layout(self.len);
            system.dealloc(buffer, layout);
        }
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

## Performance concerns and `Copy`

The decision to implement `Copy` should be based on the semantics of the type,
not on performance. If you are worried about the size of objects being copied,
then use a reference (`&T` or `&mut T`) or put it on the heap (`Box<T>`). These
situations correspond to passing by reference or using a `shared_ptr` in C++.
