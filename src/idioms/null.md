# Null (`nullptr`)

## Null members
### Moved member variables

One common use of null pointers in modern C++ is as values for the members of
moved objects so that the destructor can still safely be called. E.g.,

```c++,hidelines=#
# #include <cstdlib>
# #include <cstring>
#
// widget.h
struct widget_t;
widget_t *alloc_widget();
void free_widget(widget_t*);
void copy_widget(widget_t* dst, widget_t* src);

// widget.cc
class Widget {
    widget_t* widget;
public:
#    Widget() : widget(alloc_widget()) {}
#
#    Widget(const Widget &other) : widget(alloc_widget()) {
#        copy_widget(widget, other.widget);
#    }
#
    Widget(Widget &&other) : widget(other.widget) {
        other.widget = nullptr;
    }

    ~Widget() {
        free_widget(widget);
    }
};
```

Rust's notion of moving objects does not involve leaving behind an object on
which a destructor will be called, and so this use of null does not have a
corresponding idiom. See the chapter on [copy and move
constructors](/idioms/constructors/copy_and_move_constructors.md) for more
details.

### Partially constructed objects

A second common use is for partially constructed objects, e.g., in codebases
that make initialization a separate step from construction.

```c++
#include <string>
#include <memory>

struct Movie {
    std::string title;
};

class Person {
	std::shared_ptr<Movie> favorite_movie;
public:
    Person() : favorite_movie(nullptr) {}

    Person& init(std::shared_ptr<Movie> favorite_movie) {
        this->favorite_movie = favorite_movie;

        return *this;
    }
};
```

This pattern may be used for one of two reasons:

- A desire to incrementally initialize the object using some kind of builder
  pattern. This usually involves returning a reference to the object so that the
  initialization methods can be chained together, as `init` does in the above
  example.
- A desire to re-use allocated objects, in which case the class usually also has
  some kind of `clear()` method to prepare the object to be reused. This is most
  common in embedded programming as an alternative to dynamic allocation.

In Rust, the normal approach to the first reason is to use a builder pattern.
This involves defining a second type to represent the partially constructed
object, where each field in the original object of type `T` has type `Option<T>`
in the builder. For example,

```rust
struct Person {
    age: i32,
    name: String,
}

struct PersonBuilder {
    age: Option<i32>,
    name: Option<String>,
}

impl PersonBuilder {
    fn new() -> PersonBuilder {
        PersonBuilder { age: None, name: None }
    }

    fn age(&mut self, age: i32) -> &mut Self {
        self.age = Some(age);
        self
    }

    fn name(&mut self, name: String) -> &mut Self {
        self.name = Some(name);
        self
    }

    fn build(&self) -> Option<Person> {
        Some(Person {
            age: self.age?,
            name: match self.name {
                Some(ref name) => name.clone(),
                None => return None,
            },
        })
    }
}
```

This pattern is sufficiently common that there are libraries to support it, such
as the [`derive_builder` crate](https://crates.io/crates/derive_builder). Using
that crate, the above example would just be

```rust,ignore
#[derive(Builder)]
struct Person {
    age: i32,
    name: String,
}
```

and the resulting API would include additional features, such as the `build`
method returning a `Result` with an informative error, rather than just `None`,
when not all of required fields are set.

If there is a reasonable default value for the object, then instead of the
builder pattern, the `Default` trait can be implemented, and [values can be
constructed based on a default
value](/idioms/constructors/default_constructors.md#equivalents-in-rust).

### Sentinel values

TODO

## Nullable arguments

### Empty array

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

### Default values
null to indicate default value should be used -> use Default::default() or Option

### Scratch buffers
null to indicate use of internal scratch space -> ...

## Nullable return values

null to indicate failure -> option or result (see error handling chapter)
null to indicate success -> option<()> or result<()> (see error handling chapter)

