# Summary

[How to use this book](./how_to_use_this_book.md)

# Idioms

- [Data modeling](./idioms/data_modeling.md)
  - [Enums](./idioms/data_modeling/enums.md)
  - [Tagged unions and `std::variant`](./idioms/data_modeling/tagged_unions.md)
  - [Pure virtual classes, interfaces, and dynamic dispatch](./idioms/data_modeling/pure_virtual_classes.md)
  - [Concepts, interfaces, and static dispatch](./idioms/data_modeling/concepts.md)
  - [Inheritance and implementation reuse](./idioms/data_modeling/inheritance_and_reuse.md)
  - [Template classes, functions, and methods](./idioms/data_modeling/templates.md)
  - [Template specialization](./idioms/data_modeling/template_specialization.md)
- [Null (`nullptr`)](./idioms/null.md)
  - [Zero-length arrays](./idioms/null/zero_length_arrays.md)
  - [Sentinel values](./idioms/null/sentinel_values.md)
  - [Moved members](./idioms/null/moved_members.md)
- [Encapsulation](./idioms/encapsulation.md)
  - [Header files](./idioms/encapsulation/headers.md)
  - [Anonymous namespaces and `static`](./idioms/encapsulation/anonymous_namespaces.md)
  - [Private members and friends](./idioms/encapsulation/private_and_friends.md)
  - [Private constructors](./idioms/encapsulation/private_constructors.md)
  - [Setter and getter methods](./idioms/encapsulation/setters_and_getters.md)
- [Primitive types: `int`, `float`, `size_t`, etc.]()
- [Overloading]()
- [Constructors](./idioms/constructors.md)
  - [Default constructors](./idioms/constructors/default_constructors.md)
  - [Copy and move constructors](./idioms/constructors/copy_and_move_constructors.md)
  - [Rule of three/five/zero](./idioms/constructors/rule_of_three_five_zero.md)
  - [Separate construction and initialization](./idioms/constructors/partial_initialzation.md)
- [Destructors and resource cleanup](./idioms/destructors.md)
- [RTTI]()
- [Iterators]()
- [Out parameters]()
  - [Multiple return values](./idioms/out_params/multiple_return.md)
  - [Optional return values](./idioms/out_params/optional_return.md)
  - [Pre-allocated buffers](./idioms/out_params/pre-allocated_buffers.md)
- [Exceptions and error handling](./idioms/exceptions.md)
- [Function objects, lambdas, and closures]()
- [Object identity](./idioms/object_identity.md)
- [Varargs]()
- [Attributes]()
- [STL type equivalents]()
- [Scratch buffers]()

# Patterns

- [Visitor pattern and double dispatch]()
- [Pointer-to-implementation (PImpl)]()
- [Curiously recurring template pattern (CRTP)]()
- [X macros]()

# Libraries

# Optimizations

- [NRVO, RVO, and placement new]()

# Tooling

- [Build systems (CMake)]()
- [Unit tests]()
