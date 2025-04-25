# Pure virtual classes, interfaces, and dynamic dispatch

Pure virtual classes are used to model interfaces when objects meeting the
interface will be used with dynamic dispatch to resolve the invoked method.
In Rust the interface is given by a *trait*, which is then implemented for the
types that support that trait. Programs can then be written over *trait objects*
that use that trait as their base type.

For example, this program shows how to translate a `Shape` virtual class in C++ into a `Shape` trait in Rust,
along with corresponding `Triangle` and `Rectangle` implementations of `Shape`.

<div class="comparison">

```cpp
$#include <iostream>
$#include <memory>
$
struct Shape {
  Shape() {};
  virtual ~Shape() {};
  virtual double area() = 0;
};

struct Triangle : Shape {
  double base;
  double height;

  Triangle(double base, double height) : 
    base(base), height(height) {}

  double area() override { return 0.5 * base * height; }
};

struct Rectangle : Shape {
  double width;
  double height;

  Rectangle(double width, double height) : 
    width(width), height(height) {}

  double area() override { return width * height; }
};

int main() {
  Shape &&triangle = Triangle{1.0, 1.0};
  std::unique_ptr<Shape> rectangle = 
    std::make_unique<Rectangle>(1.0, 1.0);

  std::cout << triangle.area() << std::endl;
  std::cout << rectangle->area() << std::endl;
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
  fn area(&self) -> f64 { 0.5 * self.base * self.height }
}

struct Rectangle {
  width: f64,
  height: f64,
}

impl Shape for Rectangle {
  fn area(&self) -> f64 { self.width * self.height }
}

fn main() {
  let triangle: &dyn Shape = 
    &Triangle { base: 1.0, height: 1.0 };
  let rectangle: Box<dyn Shape> = 
    Box::new(Rectangle { width: 1.0, height: 1.0 });

  println!("{}", triangle.area());
  println!("{}", rectangle.area());
}
```

</div>

In C++, to use an object via a pure virtual class, this requires accessing the object via a pointer. 
Rust requires a similar kind of indirection. In the example above, one trait object is a reference
(with type `&dyn Shape`) and the other is a heap-allocated owned value (with
type `Box<dyn Shape>`). These correspond to the reference and `std::unique_ptr`
used in the above C++ example, respectively.

In Rust, not all traits can be used as the base trait for trait objects. The most
commonly encountered restriction is that traits that require knowledge of the
object's size via a `Sized` supertrait are not `dyn`-compatible. There are
[additional
restrictions](https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility).
