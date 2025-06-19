# Tests

This chapter gives a small example of testing in Rust by comparison to
[Boost.Test](https://github.com/boostorg/test) for C++. A more thorough guide to
testing in Rust is available as part of the [Rust
Book](https://doc.rust-lang.org/book/ch11-00-testing.html), including
information on [documentation
testing](https://doc.rust-lang.org/stable/book/ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests).
Details on the built-in support for testing in
[rustc](https://doc.rust-lang.org/rustc/tests/index.html) and
[Cargo](https://doc.rust-lang.org/cargo/commands/cargo-test.html) are available
in their respective manuals.

## Unit tests

When using C++ test frameworks such as Boost.Test, tests are defined as
functions which are registered with the framework. The tests are compiled
against a test driver provided by the framework.

In Rust, testing is handled similarly. Unlike with C++, the test registration
mechanism and test driver are both provided by Rust itself.

Additionally, the way that tests are organized is different. In C++, unit tests
are defined in separate files from the units under test. In Rust, they are
typically defined in the same file, usually in a `test` sub-module, with
inclusion of the module controlled by the `test` feature flag via the
`#[cfg(test)]` annotation.

The example below defines a small class and tests for that class. In C++ this
involves creating three separate files: a header file for the interface, the
implementation of the interface, and the test driver. In Rust, this is all done
in a single file.

<div class="comparison">

```cpp
// counter.h
#ifndef COUNTER_H
#define COUNTER_H

class Counter {
  unsigned int count;

public:
  Counter();
  unsigned int get();
  void increment();
};

#endif

// counter.cc
#include "counter.h"
Counter::Counter() : count(0) {}

unsigned int Counter::get() {
  return count;
}

void Counter::increment() {
  ++count;
}

// test_main.cc
#define BOOST_TEST_MODULE my_tests
#include <boost/test/included/unit_test.hpp>
#include "counter.h"

BOOST_AUTO_TEST_CASE(test_counter_initialize) {
  Counter c;
  BOOST_TEST(c.get() == 0);
}

BOOST_AUTO_TEST_CASE(test_counter_increment) {
  Counter c;
  c.increment();
  BOOST_TEST(c.get() == 1);
}
```

```rust
// counter.rs
pub struct Counter(u32);

impl Counter {
    pub fn new() -> Counter {
        Counter(0)
    }

    pub fn get(&self) -> u32 {
        self.0
    }

    pub fn increment(&mut self) {
        self.0 += 1;
    }
}

#[cfg(test)]
mod test {
    use super::*;

    #[test]
    fn counter_initialize() {
        let c = Counter::new();
        assert_eq!(0, c.get());
    }

    #[test]
    fn counter_increment() {
        let mut c = Counter::new();
        c.increment();
        assert_eq!(1, c.get());
    }
}
```

</div>

Defining unit tests within the module being tested has the benefit of making
internals of the module under test visible to the test code. This makes it
possible to unit test internal components without exposing them to the rest of
the program, as one would have to in C++ by including declarations in the header
file and by declaring test fixtures as `friend`s.

Running the tests in C++ involves linking against `boost_unit_test_framework`.
In Rust, the tests can be run with `cargo test`. If not using Cargo, the tests
can be compiled by passing the flag `--test` to rustc, and running the
executable produced.

## Integration tests

Integration tests as supported by Cargo, are still defined outside of the module
and outside of the crate, purely in terms of the exposed API.

See the [Rust
Book](https://doc.rust-lang.org/book/ch11-03-test-organization.html#integration-tests)
for details on how to organize integration tests for Rust programs.
