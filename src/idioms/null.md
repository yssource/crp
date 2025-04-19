# Null (nullptr)

This section covers idiomatic uses of `nullptr` in C++ and how to achieve the
same results in Rust.

Some uses of `nullptr` in C++ don't arise in the first place in Rust because of
other language differences. For example, [moved objects don't leave anything
behind that needs to be destroyed](/idioms/null/moved_members.md). Therefore
there is no to use `nullptr` as a as placeholder for a moved pointer that can
have `delete` or `free` called on it.

Other uses are replaced by `Option`, which in safe Rust requires checking for
the empty case before accessing the contained value. This use is common enough
that [Rust has an
optimization](https://doc.rust-lang.org/std/option/index.html#representation)
for when `Option` is used with a reference (`&` or `&mut ref`), `Box`
(equivalent of `unique_ptr`), and `NonNull` (a non-null raw pointer).
