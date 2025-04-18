# Encapsulation

In C++ the encapsulation boundary is the class. In Rust the encapsulation
boundary is the module, which may contain several types along with standalone
functions. In larger projects, the crate may also act as an encapsulation
boundary.

This difference means that in Rust one is more likely to have multiple tightly
coupled types that work together which are defined in one module and
encapsulated as a whole.

This section provides ways to translate between C++ and Rust's notions of
encapsulation in terms of both mechanism and concept.
