# Private constructors

In C++ constructors for classes can be made private by declaring them private,
or by defining a class using `class` and using the default private visibility.

```c++
#include <string>

struct Person {
  std::string name;
  int age;

private:
  Person() = default;
};

int main() {
  // Person nobody; // <-- fails to compile, Person::Person() private
  // Person alice{"Alice", 42}; // <-- fails to compile since C++20
  return 0;
}
```

In Rust, constructors for structs are visible from wherever the type and all
fields are visible.

To achieve similar visibility restrictions as the above example, an additional
private field needs to be added to the struct in Rust. Because Rust supports
zero-sized types, the additional field can have no performance cost. The [unit
type](https://doc.rust-lang.org/std/primitive.unit.html) has zero size and can
be used for this purpose.


```rust
mod person {
    pub struct Person {
        pub name: String,
        pub age: i32,
        _private: (),
    }
}

use person::*;

fn main() {
    // field `_private` of struct `person::Person` is private
    // let alice = Person {
    //     name: "Alice".to_string(),
    //     age: 42,
    //     _private: (),
    // };

    // cannot construct `person::Person` with struct literal syntax due to private fields
    // let bob = Person {
    //     name: "Bob".to_string(),
    //     age: 55,
    // };
}
```

## Enums

Unlike C++ unions, but like `std::variant`, Rust enums do not have direct
control over the visibility of their variants or the fields of their variants.
In the following example, the `circle` variant of the `Shape` union is not
public, so it can only be accessed from within the definition of `Shape`, as it
is by the `make_circle` static method.

```c++
#include <iostream>

struct Triangle {
  double base;
  double height;
};

struct Circle {
  double radius;
};

union Shape {
  Triangle triangle;

private:
  Circle circle;

public:
  static Shape make_circle(double radius) {
    Shape s;
    s.circle = Circle(radius);
    return s;
  };
};

int main() {
  Shape triangle;
  triangle.triangle = Triangle{1.0, 2.0};
  Shape circle = Shape::make_circle(1.0);
  // circle.circle = Circle{1.0}; // fails to compile

  // std::cout << shape.circle.radius; // fails to compile
}
```

In Rust visibility modifiers cannot be applied to individual enum variants or
their fields.

```rust
mod shape {
    pub enum Shape {
        Triangle { base: f64, height: f64 },
        Circle { radius: f64 },
    }
}

use shape::*;

fn main() {
    let triangle = Shape::Triangle {
        base: 1.0,
        height: 2.0,
    };

    let circle = Shape::Circle { radius: 1.0 };

    match circle {
        Shape::Triangle { base, height } => {
            println!("Triangle: {}, {}", base, height);
        }
        Shape::Circle { radius } => {
            println!("Circle {}", radius);
        }
    }
}
```

Instead, to control construction of and pattern matching on the enum
implementation, one of two approaches can be taken. The first controls
construction of and access to the fields, but not inspection of which variant is
active.

```rust
mod shape {
    pub enum Shape {
        Triangle { base: f64, height: f64 },
        Circle { radius: f64 },
    }
}

use shape::*;

fn main() {
    let triangle = Shape::Triangle {
        base: 1.0,
        height: 2.0,
    };

    let circle = Shape::Circle { radius: 1.0 };

    match circle {
        Shape::Triangle { base, height } => {
            println!("Triangle: {}, {}", base, height);
        }
        Shape::Circle { radius } => {
            println!("Circle {}", radius);
        }
    }
}
```

The second places the enum in a struct with a private field, preventing both
construction and inspection from outside of the module.

```rust
mod shape {
    enum ShapeKind {
        Triangle { base: f64, height: f64 },
        Circle { radius: f64 },
    }

    pub struct Shape(ShapeKind);

    impl Shape {
        pub fn new_circle(radius: f64) -> Shape {
            Shape(ShapeKind::Circle { radius })
        }

        pub fn new_triangle(base: f64, height: f64) -> Shape {
            Shape(ShapeKind::Triangle { base, height })
        }

        pub fn print(&self) {
            match self.0 {
                ShapeKind::Triangle { base, height } => {
                    println!("Triangle: {}, {}", base, height);
                }
                ShapeKind::Circle { radius } => {
                    println!("Circle: {}", radius);
                }
            }
        }
    }
}

use shape::*;

fn main() {
    let triangle = Shape::new_triangle(1.0, 2.0);
    let circle = Shape::new_circle(1.0);

    // Does not compile because Shape has private fields.
    // match circle {
    //   Shape(_) -> {}
    // }

    circle.print();
}
```

If the purpose of making the variants private is to ensure that invariants are
met, then it can be useful to expose the implementing enum (`ShapeKind`) but not
the field of the wrapping struct (`Shape`), with the invariants only being
guaranteed when the wrapping struct is used.

```rust
mod shape {
    pub enum ShapeKind {
        Triangle { base: f64, height: f64 },
        Circle { radius: f64 },
    }

    pub struct Shape(ShapeKind);

    impl Shape {
        pub fn new(kind: ShapeKind) -> Shape {
            // ... check invariants ...
            Shape(kind)
        }

        pub fn get_kind(&self) -> &ShapeKind {
            &self.0
        }
    }
}

use shape::*;

fn main() {
    let triangle = Shape::new(ShapeKind::Triangle {
        base: 1.0,
        height: 2.0,
    });
    let circle = Shape::new(ShapeKind::Circle { radius: 1.0 });

    // Does not compile because Shape has private fields.
    // match circle {
    //   Shape(_) -> {}
    // }

    match circle.get_kind() {
        ShapeKind::Triangle { base, height } => {
            println!("Triangle: {}, {}", base, height);
        }
        ShapeKind::Circle { radius } => {
            println!("Circle: {}", radius);
        }
    }
}
```

The situation in Rust resembles the situation in C++ when using `std::variant`,
for which it is not possible to make the variants themselves private. Instead
either the constructors for the types that form the variants can be made private
or the variant can be wrapped in a class with appropriate visibility controls.
