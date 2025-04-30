# Pure virtual classes, interfaces, and dynamic dispatch

Pure virtual classes are used to model interfaces when objects meeting the
interface will be used with dynamic dispatch to resolve the invoked method.
In Rust the interface is given by a *trait*, which is then implemented for the
types that support that trait. Programs can then be written over *trait objects*
that use that trait as their base type.

The following example defines an interface, two implementations of that
interface, and a function that takes an argument that satisfies the interface.
In C++ the interface is defined with an abstract class where all of the methods
are pure virtual methods, and in Rust the interface is defined with a trait.

In both languages the function (`printArea` in C++ and `print_area` in Rust)
invokes a method using dynamic dispatch.

<div class="comparison">

```cpp
#include <iostream>
#include <memory>

struct Shape {
  Shape() {};
  virtual ~Shape() {};
  virtual double area() = 0;
};

struct Triangle : Shape {
  double base;
  double height;

  Triangle(double base, double height)
      : base(base), height(height) {}

  double area() override {
    return 0.5 * base * height;
  }
};

struct Rectangle : Shape {
  double width;
  double height;

  Rectangle(double width, double height)
      : width(width), height(height) {}

  double area() override {
    return width * height;
  }
};

void printArea(Shape &shape) {
  std::cout << shape.area() << std::endl;
}

int main() {
  Triangle triangle = Triangle{1.0, 1.0};

  printArea(triangle);

  std::unique_ptr<Shape> shape;
  if (true) {
    shape = std::make_unique<Rectangle>(1.0, 1.0);
  } else {
    shape = std::make_unique<Triangle>(
        std::move(triangle));
  }

  printArea(*shape);
}
```

```rust
trait Shape {
    fn area(&self) -> f64;
}

struct Triangle {
    base: f64,
    height: f64,
}

impl Shape for Triangle {
    fn area(&self) -> f64 {
        0.5 * self.base * self.height
    }
}

struct Rectangle {
    width: f64,
    height: f64,
}

impl Shape for Rectangle {
    fn area(&self) -> f64 {
        self.width * self.height
    }
}

fn print_area(shape: &dyn Shape) {
    println!("{}", shape.area());
}

fn main() {
    let triangle = Triangle {
        base: 1.0,
        height: 1.0,
    };

    print_area(&triangle);

    let shape: Box<dyn Shape> = if true {
        Box::new(Rectangle {
            width: 1.0,
            height: 1.0,
        })
    } else {
        Box::new(triangle)
    };

    print_area(shape.as_ref());
}
```

</div>

## Indirection, object slicing, and Rust trait object types

In C++, using an object via an abstract base class is done by accessing the
object via a pointer or a reference, supported by a vtable.

Rust also requires the indirection. The type `dyn Shape` in the above example is
the type of a trait object for the `Shape` trait. A trait object includes a
vtable along with the underlying value.

The indirection is enforced by the type `dyn Shape` not implementing the `Sized`
marker trait, preventing it from being used in contexts that require knowing the
size of a type statically.

```rust
# trait Shape {
#     fn area(&self) -> f64;
# }
#
# struct Triangle {
#     base: f64,
#     height: f64,
# }
#
# impl Shape for Triangle {
#     fn area(&self) -> f64 {
#         0.5 * self.base * self.height
#     }
# }
#
fn main() {
    // Local variables must have a known size.
    // let v: dyn Shape = Triangle { base: 1.0, height: 1.0 };

    // References always have a known size.
    let shape: &dyn Shape = &Triangle {
        base: 1.0,
        height: 1.0,
    };
    // Boxes also always have a known size.
    let boxed_shape: Box<dyn Shape> = Box::new(Triangle {
        base: 1.0,
        height: 1.0,
    });

    // Types like Option<T> the value of type T directly, and so also need to
    // know the size of T.
    // let v: Option<dyn Shape> = Some(Triangle { base: 1.0, height: 1.0 });
}

// Parameter types must have a known size.
// fn print_area(shape: dyn Shape) { }
fn print_area(shape: &dyn Shape) {}
```

The forbidden uses correspond to situations in which object slicing would occur
in C++.

## Limitations of trait objects in Rust

In Rust, not all traits can be used as the base trait for trait objects. The most
commonly encountered restriction is that traits that require knowledge of the
object's size via a `Sized` supertrait are not `dyn`-compatible. There are
[additional
restrictions](https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility).

## Trait objects and lifetimes

Objects which are used with dynamic dispatch may contain pointers or references
to other objects. In C++ the lifetimes of those references must be tracked
manually by the programmer.

Rust checks the bounds on the lifetimes of references that the trait objects may
contain. If the bounds are not given explicitly, they are determined according
to the [lifetime elision
rules](https://doc.rust-lang.org/reference/lifetime-elision.html#r-lifetime-elision.trait-object).

Usually the elision rules pick the correct lifetime bound. Sometimes, the rules
result in surprising error messages from the compiler. In those situations or
when the compiler cannot determine which lifetime bound to assign, the bound may
be given manually. The following example shows explicitly what the inferred
lifetimes are for a structure storing a trait object and for the `print_area`
function.

```rust
# trait Shape {
#     fn area(&self) -> f64;
# }
#
# struct Triangle {
#     base: f64,
#     height: f64,
# }
#
# impl Shape for Triangle {
#     fn area(&self) -> f64 {
#         0.5 * self.base * self.height
#     }
# }
#
struct Scaled {
    scale: f64,
    // 'static is the lifetime that would be inferred by the lifetime elision
    // rule [lifetime-elision.trait-object.default].
    shape: Box<dyn Shape + 'static>,
}

impl Shape for Scaled {
    fn area(&self) -> f64 {
        self.scale * self.shape.area()
    }
}

// These are the lifetimes that would be inferred by the lifetime elision rule
// [lifetime-elision.function.implicit-lifetime-parameters] for the reference
// and [lifetime-elision.trait-object.containing-type-unique] for the trait
// bound.
fn print_area<'a>(shape: &'a (dyn Shape + 'a)) {
    println!("{}", shape.area());
}

fn main() {
    let triangle = Triangle {
        base: 1.0,
        height: 1.0,
    };
    print_area(&triangle);

    let scaled_triangle = Scaled {
        scale: 2.0,
        shape: Box::new(triangle),
    };
    print_area(&scaled_triangle);
}
```

{{#quiz pure_virtual_classes.toml}}