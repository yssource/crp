# Header files

One use of header files in C++ is to expose declarations that are defined in one
translation units to other translation units without requiring the duplication
of the declarations in multiple files. By convention, declarations that are not
included in the header are considered to be private to the defining translation
unit (though, to enforce this convention other mechanisms, such as [anonymous
namespaces](/idioms/encapsulation/anonymous_namespaces.md), are required).

```c++
// person.h
class Person {
    std::string name;
public:
    Person(std::string name) : name(name) {}
    const std::string& getName();
};

// person.cc
std::string& Person::getName() {
    return this.name;
}

// client.cc
#include "person.h"

int go() {
    Person p("Alice");
    std::string& name = p.getName();

    // ...

    return 0;
}
```

Rust does uses neither textually-included header files nor forward declarations.
Instead, Rust modules control visibility and linkage simultaneously and expose
public definitions for use by other modules. Using Rust modules, something like
the above definitions becomes

```rust,ignore
// person.rs
pub struct Person {
    name: String,
}

impl Person {
    pub fn new(name: String) -> Person {
        Person { name }
    }

    pub fn name(&self) -> &String {
        &self.name
    }
}

// client.rs
mod person;

use person::*;

fn go() {
    let p = Person::new("Alice");
    let name = p.name();

    //...
}
```

In the `person.rs`, the `Person` type is public by the `name` field is not. This
prevents both direct construction of values of the type (similar to private
members preventing aggregate initialization) and field access (including
preventing pattern matching). The static method `new` and method `name` are
exposed to clients of the module by the `pub` visibility declarations.

In the `client` module, the `mod` declaration defines the content of `person.rs`
as a submodule named `person`. The `use` declaration brings the contents of the
`person` module into scope.

## The essence of the difference

A C++ program is a collection of translation units. Header files are required to
make the providing of forward declarations of definitions from other translation
units manageable.

A Rust program is a tree of modules. Definitions in one module may access items
from other modules based on visibility declarations given in the definitions of
the module themselves.

## Submodules and additional visibility features

Modules and visibility declarations are more powerful than shown in the above
example. More details on how to use modules, `pub`, and `use` to achieve
encapsulation goals are described in the chapter on [private members and
friends](./idioms/encapsulation/private_members.md).
