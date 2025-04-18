# Setter and getter methods

Setters and getters work similarly in C++ and Rust, but are used less frequently
in Rust.

It would not be unusual to see the following representation of a two-dimensional
vector in C++, which hides its implementation and provides setters and getters
to access the fields. This choice would typically be made in case a
representation change (such as using polar instead of rectangular coordinates)
needed to be made later without breaking clients.

```c++
class Vec2 {
  double x;
  double y;

public:
  Vec2(double x, double y) : x(x), y(y) {}
  double getX() { return x; }
  double getY() { return y; }

  // ... vector operations ...
};
```

On the other hand, in Rust such a type would almost always be defined with
public fields.

```rust
pub struct Vec2 {
    pub x: f64,
    pub y: f64,
}

impl Vec2 {
    // ... vector operations ...
}
```

Some reasons for the difference are:

- Ergonomics: Public members make it possible to use pattern matching.
- Transparency of performance: A change in representation would dramatically
  change the costs involved with the getters. Exposing the representation makes
  the cost change visible.
- Control over mutability: Static lifetime checking of mutable references
  removes concerns of unintended mutation of the value through Rust's equivalent
  of observation pointers.

## Types with invariants and newtypes

When types need to preserve invariants but the benefits of exposing fields are
desired, a newtype pattern can be used. A wrapping "newtype" struct that
represents the data with an invariant is defined and access to the fields of the
underlying struct is provided by via a non-`mut` reference.

```rust
mod vec2 {
    #[derive(Debug, Clone, Copy)]
    pub struct Vec2 {
        pub x: f64,
        pub y: f64,
    }

    /// Represents a 2-vector that has magnitude 1.
    #[derive(Debug, Clone, Copy)]
    pub struct Normalized(Vec2); // note the private field

    fn sqrt_approx_zero(x: f64) -> bool {
        x < 0.001
    }

    impl Normalized {
        pub fn from_vec2(v: Vec2) -> Option<Normalized> {
            if sqrt_approx_zero(v.x * v.x + v.y * v.x - 1.0) {
                Some(Normalized(v))
            } else {
                None
            }
        }

        // The getter provides a reference to the underlying Vec2 value
        // permitting mutation.
        pub fn get(&self) -> &Vec2 {
            &self.0
        }
    }
}
```
