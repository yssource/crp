# Curiously recurring template pattern (CRTP)

The C++ [curiously recurring template
pattern](https://en.cppreference.com/w/cpp/language/crtp) is used to make the
concrete type of the derived class available in the definition of methods
defined in the base class.

## Sharing implementations with static polymorphism

The basic use of the CRTP is for reducing redundancy in implementations that
make use of static polymorphism. In this use case, the `this` pointer is cast to
the type provided by the template parameter so that methods from the derived
class can be called. This enables methods implemented in the base class to call
methods in the derived class without having to declare them virtual, avoiding
the cost of dynamic dispatch.

In the following example, `Triangle` and `Square` have a common implementation
of `twiceArea` without the need for dynamic dispatch. This use case is addressed
in Rust using default trait methods.

<div class="comparison">

```cpp
#include <iostream>

template <typename T>
struct Shape {
  // This implementation is shared and can call
  // the area method from derived classes without
  // declaring it virtual.
  double twiceArea() {
    return 2.0 * static_cast<T *>(this)->area();
  }
};

struct Triangle : public Shape<Triangle> {
  double base;
  double height;

  Triangle(double base, double height)
      : base(base), height(height) {}

  double area() {
    return 0.5 * base * height;
  }
};

struct Square : public Shape<Square> {
  double side;

  Square(double side) : side(side) {}

  double area() {
    return side * side;
  }
};

int main() {
  Triangle triangle{2.0, 1.0};
  Square square{2.0};

  std::cout << triangle.twiceArea() << std::endl;
  std::cout << square.twiceArea() << std::endl;
}
```

```rust
trait Shape {
    fn area(&self) -> f64;

    fn twice_area(&self) -> f64 {
        2.0 * self.area()
    }
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

struct Square {
    side: f64,
}

impl Shape for Square {
    fn area(&self) -> f64 {
        self.side * self.side
    }
}

fn main() {
    let triangle = Triangle {
        base: 2.0,
        height: 1.0,
    };
    let square = Square { side: 2.0 };
    println!("{}", triangle.twice_area());
    println!("{}", square.twice_area());
}
```

</div>

The reason why nothing additional needs to be done for the default method to
invoke area statically in Rust is that calls to methods on `self` are always
resolved statically in Rust. This is possible because [Rust does not have
inheritance between concrete
types](../idioms/data_modeling/inheritance_and_reuse.md). Despite being defined in
the trait, the default method is actually implemented as part of the
implementing struct.

## Method chaining

Another common use for the CRTP is for implementing method chaining when an
implementation of a method to be chained is provided by a base class.

In C++ the template parameter is used to ensure that the type returned from the
shared function is that of the derived class, so that further methods defined in
the derived class can be called on it. The template parameter is also used to
call a method on the derived type without declaring the method as virtual.

In Rust the template parameter is not required because the `Self` type is
available in traits to refer to the type of the implementing struct.

<div class="comparison">

```cpp
#include <iostream>
#include <span>
#include <string>
#include <vector>

// D is the type of the derived class
template <typename D>
struct Combinable {
  D combineWith(D &d);

  // concat is implemented in the base class, but
  // operates on values of the derived class.
  D concat(std::span<D> vec) {
    D acc(*static_cast<D *>(this));

    for (D &v : vec) {
      acc = acc.combineWith(v);
    }

    return acc;
  }
};

struct Sum : Combinable<Sum> {
  int sum;

  Sum(int sum) : sum(sum) {}

  Sum combineWith(Sum s) {
    return Sum(sum + s.sum);
  }

  // Sum includes an additional method that can be
  // chained.
  Sum mult(int n) {
    return Sum(sum * n);
  }
};

int main() {
  Sum s(0);
  std::vector<Sum> v{1, 2, 3, 4};
  Sum x = s.concat(v)
              // Even though concat is part of the
              // base class, it returns a value of
              // the implementing class, making it
              // possible to chain methods
              // specific to that class.
              .mult(2)
              .combineWith(5);
  std::cout << x.sum << std::endl;
}
```

```rust
// No generic type is required: Self already
// refers to implementing type.
trait Combinable {
    fn combine_with(&self, other: &Self) -> Self;

    // concat has a default implementation in
    // terms of Self.
    fn concat(&self, others: &[Self]) -> Self
    where
        Self: Clone,
    {
        let mut acc = self.clone();

        for v in others {
            acc = acc.combine_with(v);
        }
        acc
    }
}

#[derive(Clone)]
struct Sum(i32);

impl Sum {
    // Sum includes an additional method that can be
    // chained.
    fn mult(&self, n: i32) -> Self {
        Self(self.0 * n)
    }
}

impl Combinable for Sum {
    fn combine_with(&self, other: &Self) -> Self {
        Self(self.0 + other.0)
    }
}

fn main() {
    let s = Sum(0);
    let v = vec![Sum(1), Sum(2), Sum(3), Sum(4)];
    let x = s
        .concat(&v)
        // Even though concat is part of the
        // trait, it returns a value of the
        // implementing type, making it possible
        // to chain methods specific to that type.
        .mult(2)
        .combine_with(&Sum(5));
    println!("{}", x.0)
}
```

</div>

Again, the reason why `Self` can refer to the implementing type is that [Rust
does not have inheritance between concrete
types](../idioms/data_modeling/inheritance_and_reuse.md). This contrasts with C++
where a value may be used at any number of types which are concrete, and so it
would not be clear which type something like `Self` should refer to.

{{#quiz crtp.toml}}
