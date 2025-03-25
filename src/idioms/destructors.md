# Destructors

In C++, destructors are defined by providing a special member function. To
achieve the equivalent in Rust, implement the [`Drop`
trait](https://doc.rust-lang.org/std/ops/trait.Drop.html).

For an example, see [the example in the chapter on copy and move
constructors](constructors/copy_and_move_constructors.html#user-defined-constructors).

## Lifetimes and destructors

