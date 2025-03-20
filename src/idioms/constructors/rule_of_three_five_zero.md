# Rule of three/five/zero

## Rule of three

In C++ the rule of three is a rule of thumb that if a class has a user-defined
destructor, copy constructor or copy assignment operator, it probably should
have all three.

The corresponding rule for Rust is that if a type has a user-defined `Clone` or
`Drop` implementation, it probably needs both or to have no `Clone`
implementation at all.

This is for the same reason as the rule of three in C++: if a type has a
user-defined implementation for `Clone` or `Drop`, it is probably because the
type manages a resource, and both `Clone` and `Drop` will need to take special
actions for the resource.

## Rule of five

The rule of five in C++ states that if move semantics are needed for a type with
a user-defined copy constructor or copy assignment operator, then a user-defined
move constructor and move assignment should also be provided, because no
implicit move constructor or move assignment operator will be generated.

In Rust, this rule does not apply because of the [difference in move semantics
between C++ and Rust.](copy_and_move_constructors.md#move-constructors)

## Rule of zero

The rule of zero states that classes with user-defined copy/move constructors,
assignment operators, and destructors should deal only with ownership, and other
classes should not have those constructors or destructors. In practice, most
classes should make use of types from the STL (`shared_ptr`, `vector`, etc.) for
dealing with ownership concerns so that the implicitly defined copy and move
constructors are sufficient.

In Rust, the same is true...

TODO Move the following to a separate chapter on type equivalents and link to it
here.

The following table maps the ownership-managing classes from C++ to equivalents
types in Rust:

| C++ name                                  | C++ type             | Rust type                                               |
|-------------------------------------------|----------------------|---------------------------------------------------------|
| Owned in-place                            | `T`                  | `T`                                                     |
| Single ownership on heap                  | `std::unique_ptr<T>` | `std::box::Box<T>`                                      |
| Shared ownership on heap                  | `std::shared_ptr<T>` | `std::rc::Rc<T> or std::sync::Arc<T>`                   |
| Shared ownership on heap of mutable value | `std::shared_ptr<T>` | `std::rc::Rc<RefCell<T>> or std::sync::Arc<RefCell<T>>` |
| Const observation pointer                 | `const *T`           | `&T`                                                    |
| Mutable observation pointer               | `*T`                 | `&mut T`                                                |
| Const reference                           | `const &T`           | `&T`                                                    |
| Mutable reference                         | `&T`                 | `&mut T`                                                |

There are also types that manage ownership while also providing other
functionality...

In some cases, multiple types might be needed to achieve the same effect. A C++
`shared_ptr` allows all of the sharing owners to mutate the object, even if
there is more than one owner. In Rust, this is not allowed. Instead, the [type
`Rc<RefCell<T>>` must be
used](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html#having-multiple-owners-of-mutable-data-by-combining-rct-and-refcellt).
