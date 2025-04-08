# Summary

[How to use this book](./how_to_use_this_book.md)

# Idioms

- [Data modeling](./idioms/data_modeling.md)
  - [Enums](./idioms/data_modeling/enums.md)
  - [Tagged unions and `std::variant`](./idioms/data_modeling/tagged_unions.md)
  - [Pure virtual classes, interfaces, and dynamic dispatch](./idioms/data_modeling/pure_virtual_classes.md)
  - [Concepts, interfaces, and static dispatch](./idioms/data_modeling/concepts.md)
  - [Abstract classes and implementation reuse]()
  - [Modeling shallow type hierarchies]()
  - [Template classes, functions, and methods]()
  - [Template specialization]()
  - [Static and dynamic dispatch (virtual methods)]()
- [Null (`nullptr`)]()
  - [Zero-length arrays](./idioms/null/zero_length_arrays.md)
  - [Sentinel values]()
  - [Moved members](./idioms/null/moved_members.md)
- [Encapsulation]()
  - [Header files](./idioms/encapsulation/headers.md)
  - [Anonymous namespaces and `static`](./idioms/encapsulation/anonymous_namespaces.md)
  - [Private members and friends]()
  - [Returning const references]()
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
- [X macros]()

# Libraries

# Optimizations

- [NRVO, RVO, and placement new]()

# Tooling

- [Build systems (CMake)]()
