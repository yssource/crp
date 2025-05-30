# Moved members

Moving values out of variables or fields in Rust is more explicit than it is in
C++. A value that might be moved with nothing left behind needs to be
represented using an `Option<Box<T>>` type in Rust, while in C++ it would just
be a `std::unique_ptr<T>`.

<div class="comparison">

```c++
#include <memory>

void readMailbox(std::unique_ptr<int> &mailbox,
                 std::mutex mailboxMutex) {
  std::lock_guard<std::mutex> guard(mailboxMutex);

  if (!mailbox) {
    return;
  }
  int x = *mailbox;
  mailbox = nullptr;
  // use x
}
```

```rust
use std::sync::Arc;
use std::sync::Mutex;

fn read(mailbox: Arc<Mutex<Option<i32>>>) {
    let Ok(mut x) = mailbox.lock() else {
        return;
    };
    let x = x.take();
    // use x
}
```

</div>

Additionally, when taking ownership of a value from within a mutable reference,
something has to be left in its place. This can be done using
[`std::mem::swap`](https://doc.rust-lang.org/std/mem/fn.swap.html), and many
container-like types have methods for making common ownership-swapping more
ergonomic, like
[`Option::take`](https://doc.rust-lang.org/std/option/enum.Option.html#method.take)
as seen in the earlier example,
[`Option::replace`](https://doc.rust-lang.org/std/option/enum.Option.html#method.replace)
or
[`Vec::swap`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.swap_remove).

## Deleting moved objects

Another common use of null pointers in modern C++ is as values for the members
of moved objects so that the destructor can still safely be called. E.g.,

```cpp
$#include <cstdlib>
$#include <cstring>
$
// widget.h
struct widget_t;
widget_t *alloc_widget();
void free_widget(widget_t*);
void copy_widget(widget_t* dst, widget_t* src);

// widget.cc
class Widget {
    widget_t* widget;
public:
$    Widget() : widget(alloc_widget()) {}
$
$    Widget(const Widget &other) : widget(alloc_widget()) {
$        copy_widget(widget, other.widget);
$    }
$
    Widget(Widget &&other) : widget(other.widget) {
        other.widget = nullptr;
    }

    ~Widget() {
        free_widget(widget);
    }
};
```

Rust's notion of moving objects does not involve leaving behind an object on
which a destructor will be called, and so this use of null does not have a
corresponding idiom. See the chapter on [copy and move
constructors](../constructors/copy_and_move_constructors.md) for more
details.
