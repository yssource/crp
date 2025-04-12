# Tagged unions and `std::variant`

## C-style tagged unions

Because unions cannot be used for type punning in C++, when they are used it is
usually with a tag to discriminate between which variant of the union is active.

```c++
enum Tag { Rectangle, Triangle };

struct Shape {
  Tag tag;
  union Value {
    struct {
      double width;
      double height;
    } rectangle;
    struct {
      double base;
      double height;
    } triangle;
  } value;

  double area() {
    switch (this->tag) {
    case Rectangle: {
      return this->value.rectangle.width * this->value.rectangle.height;
    }
    case Triangle: {
      return 0.5 * this->value.triangle.base * this->value.triangle.height;
    }
    }
  }
};
```

Rust's equivalent to union types are always tagged. They are a generalization of
Rust enums, where additional data may be associated with the enum variants.

```rust
enum Shape {
    Rectangle { width: f64, height: f64 },
    Triangle { base: f64, height: f64 },
}

impl Shape {
    fn area(&self) -> f64 {
        match self {
            Shape::Rectangle { width, height } => width * height,
            Shape::Triangle { base, height } => 0.5 * base * height,
        }
    }
}
```
## Accessing the value without checking the discriminant

Unlike with C-style unions, Rust always requires matching on the discriminant
before accessing the values. If the variant is already known, e.g., due to an
earlier check, then the code can usually be refactored to encode the knowledge
in the type so that the second check (and corresponding error handling) can be
omitted.

A C++ program like the following requires more restructuring of the types to
achieve the same goal in Rust.

```c++
#include <ranges>
#include <vector>

std::vector<Shape> get_shapes() {
  return std::vector<Shape>{
      Shape{Triangle, {.triangle = {1.0, 1.0}}},
      Shape{Triangle, {.triangle = {1.0, 1.0}}},
      Shape{Rectangle, {.rectangle = {1.0, 1.0}}},
  };
}

std::vector<Shape> get_shapes();

int main() {
  std::vector<Shape> shapes = get_shapes();

  auto is_triangle = [](Shape shape) { return shape.tag == Triangle; };

  // Create an iterator that only sees the triangles. (std::views::filter is
  // from C++20, but the same effect can be acheived with a custom iterator.)
  auto triangles = shapes | std::views::filter(is_triangle);

  double total_base = 0.0;
  for (auto &triangle : triangles) {
    // Skip checking the tag because we know we have only triangles.
    total_base += triangle.value.triangle.base;
  }

  return 0;
}
```

The corresponding Rust program requires defining separate types for each variant
of the `Shape` enum so that the fact that all of the value are of a given type
can be expressed in the type system by having an array of `Triangle` instead of
an array of `Shape`.

```rust
struct Rectangle { width: f64, height: f64 }
struct  Triangle { base: f64, height: f64 }

enum Shape {
    Rectangle(Rectangle),
    Triangle(Triangle),
}


fn get_shapes() -> Vec<Shape> {
    vec![
        Shape::Triangle(Triangle {
            base: 1.0,
            height: 1.0,
        }),
        Shape::Triangle(Triangle {
            base: 1.0,
            height: 1.0,
        }),
        Shape::Rectangle(Rectangle {
            width: 1.0,
            height: 1.0,
        }),
    ]
}

fn main() {
    let shapes = get_shapes();

    // Create an iterator that only sees the triangles.
    let triangles = shapes
        .iter()
        // Keep only the triangles
        .filter_map(|shape| match shape {
            Shape::Triangle(t) => Some(t),
            _ => None,
        });

    let mut total_base = 0.0;
    for triangle in triangles {
        total_base += triangle.base;
    }
}
```

This kind of use is common enough that the variants are often designed to have
their own types from the start.

## `std::variant` (since C++17)

In more modern C++, `std::variant` is more similar in usage to Rust.

```c++
#include <variant>

struct Rectangle {
  double width;
  double height;
};

struct Triangle {
  double base;
  double height;
};

using Shape = std::variant<Rectangle, Triangle>;

double area(const Shape &shape) {
  return std::visit(
      [](auto &&arg) -> double {
        using T = std::decay_t<decltype(arg)>;
        if constexpr (std::is_same_v<T, Rectangle>) {
          return arg.width * arg.height;
        } else if constexpr (std::is_same_v<T, Triangle>) {
          return 0.5 * arg.base * arg.height;
        }
      },
      shape);
}
```

Because Rust doesn't depend on templates for this language feature, error
messages when a variant is missed or when a new variant is added are easier to
read, which removes one of the barriers to using tagged unions more frequently.
Compare the errors in C++ (using gcc) and Rust when the `Triangle` case is
omitted.

```c++
#include <variant>

struct Rectangle {
  double width;
  double height;
};

struct Triangle {
  double base;
  double height;
};

using Shape = std::variant<Rectangle, Triangle>;

double area(const Shape &shape) {
  return std::visit(
      [](auto &&arg) -> double {
        using T = std::decay_t<decltype(arg)>;
        if constexpr (std::is_same_v<T, Rectangle>) {
          return arg.width * arg.height;
        }
      },
      shape);
}
```

```text
$ g++ -o example example.cc
example.cc: In instantiation of ‘area(const Shape&)::<lambda(auto:27&&)> [with auto:27 = const Triangle&]’:
/usr/include/c++/14.2.1/bits/invoke.h:61:36:   required from ‘constexpr _Res std::__invoke_impl(__invoke_other, _Fn&&, _Args&& ...) [with _Res = double; _Fn = area(const Shape&)::<lambda(auto:27&&)>; _Args = {const Triangle&}]’
   61 |     { return std::forward<_Fn>(__f)(std::forward<_Args>(__args)...); }
      |              ~~~~~~~~~~~~~~~~~~~~~~^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
/usr/include/c++/14.2.1/bits/invoke.h:96:40:   required from ‘constexpr typename std::__invoke_result<_Functor, _ArgTypes>::type std::__invoke(_Callable&&, _Args&& ...) [with _Callable = area(const Shape&)::<lambda(auto:27&&)>; _Args = {const Triangle&}; typename __invoke_result<_Functor, _ArgTypes>::type = double]’
   96 |       return std::__invoke_impl<__type>(__tag{}, std::forward<_Callable>(__fn),
      |              ~~~~~~~~~~~~~~~~~~~~~~~~~~^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   97 |                                         std::forward<_Args>(__args)...);
      |                                         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
/usr/include/c++/14.2.1/variant:1060:24:   required from ‘static constexpr decltype(auto) std::__detail::__variant::__gen_vtable_impl<std::__detail::__variant::_Multi_array<_Result_type (*)(_Visitor, _Variants ...)>, std::integer_sequence<long unsigned int, __indices ...> >::__visit_invoke(_Visitor&&, _Variants ...) [with _Result_type = std::__detail::__variant::__deduce_visit_result<double>; _Visitor = area(const Shape&)::<lambda(auto:27&&)>&&; _Variants = {const std::variant<Rectangle, Triangle>&}; long unsigned int ...__indices = {1}]’
 1060 |           return std::__invoke(std::forward<_Visitor>(__visitor),
      |                  ~~~~~~~~~~~~~^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 1061 |               __element_by_index_or_cookie<__indices>(
      |               ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 1062 |                 std::forward<_Variants>(__vars))...);
      |                 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
/usr/include/c++/14.2.1/variant:1820:5:   required from ‘constexpr decltype(auto) std::__do_visit(_Visitor&&, _Variants&& ...) [with _Result_type = __detail::__variant::__deduce_visit_result<double>; _Visitor = area(const Shape&)::<lambda(auto:27&&)>; _Variants = {const variant<Rectangle, Triangle>&}]’
 1820 |                   _GLIBCXX_VISIT_CASE(1)
      |                   ^~~~~~~~~~~~~~~~~~~
/usr/include/c++/14.2.1/variant:1882:34:   required from ‘constexpr std::__detail::__variant::__visit_result_t<_Visitor, _Variants ...> std::visit(_Visitor&&, _Variants&& ...) [with _Visitor = area(const Shape&)::<lambda(auto:27&&)>; _Variants = {const variant<Rectangle, Triangle>&}; __detail::__variant::__visit_result_t<_Visitor, _Variants ...> = double]’
 1882 |             return std::__do_visit<_Tag>(
      |                    ~~~~~~~~~~~~~~~~~~~~~^
 1883 |               std::forward<_Visitor>(__visitor),
      |               ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 1884 |               static_cast<_Vp>(__variants)...);
      |               ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
example.cc:17:20:   required from here
   17 |   return std::visit(
      |          ~~~~~~~~~~^
   18 |       [](auto &&arg) -> double {
      |       ~~~~~~~~~~~~~~~~~~~~~~~~~~
   19 |         using T = std::decay_t<decltype(arg)>;
      |         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   20 |         if constexpr (std::is_same_v<T, Rectangle>) {
      |         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   21 |           return arg.width * arg.height;
      |           ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   22 |         }
      |         ~
   23 |       },
      |       ~~
   24 |       shape);
      |       ~~~~~~
example.cc:23:7: error: no return statement in ‘constexpr’ function returning non-void
   23 |       },
      |       ^
example.cc: In lambda function:
example.cc:23:7: warning: control reaches end of non-void function [-Wreturn-type]
```

```rust,ignore
enum Shape {
    Rectangle { width: f64, height: f64 },
    Triangle { base: f64, height: f64 },
}

impl Shape {
    fn area(&self) -> f64 {
        match self {
            Shape::Rectangle { width, height } => width * height,
        }
    }
}
```

```text
error[E0004]: non-exhaustive patterns: `&Shape::Triangle { .. }` not covered
 --> example.rs:8:15
  |
8 |         match self {
  |               ^^^^ pattern `&Shape::Triangle { .. }` not covered
  |
note: `Shape` defined here
 --> example.rs:1:6
  |
1 | enum Shape {
  |      ^^^^^
2 |     Rectangle { width: f64, height: f64 },
3 |     Triangle { base: f64, height: f64 },
  |     -------- not covered
  = note: the matched value is of type `&Shape`
help: ensure that all possible cases are being handled by adding a match arm with a wildcard pattern or an explicit pattern as shown
  |
9 ~             Shape::Rectangle { width, height } => width * height,
10~             &Shape::Triangle { .. } => todo!(),
  |

error: aborting due to 1 previous error

For more information about this error, try `rustc --explain E0004`.
```
