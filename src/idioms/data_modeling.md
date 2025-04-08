# Data modeling

In C++ the mechanisms available for data modeling are classes, enums, and
unions.

Rust, on the other hand, uses records and algebraic data types (ADTs).

One major difference between C++ and Rust that arises from this is that C++
types are open (i.e., you can define a new variant of a type by defining a
subclass), while Rust types are closed (that the set of variants for a Rust type
are fixed at the time of definition). The other side of this trade-off is that
in C++ the set of interfaces that are supported by a type is closed, while in
Rust the set of interfaces is open. In both languages, there are ways to work
around the restrictions of the other (e.g., the adapter pattern in C++).

This section gives examples of common constructions used when programming in C++
data and how to achieve the same effects using Rust's features.
