# Separate construction and initialization

A second common use is for partially constructed objects, e.g., in codebases
that make initialization a separate step from construction.

```cpp
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
