# Pre-allocated buffers

There are situations where large quantities of data need to be returned from a
function that will be called repeatedly, so that incurring the copies involved
in returning by value or repeated heap allocations would be cost prohibitive.
Some of these situations include:

- performing file or network IO,
- communicating with graphics hardware,
- communicating with hardware on embedded systems, or
- implementing cryptography algorithms.

In these situations, C++ programs tend to pre-allocate buffers that are reused
for all calls. This also usually enables allocating the buffer on the stack,
rather than having to use dynamic storage.

The following example pre-allocates a buffer and reads a large file into it
within a loop.

<div class="comparison">

```cpp
#include <fstream>

int main() {
  std::ifstream file("/path/to/file");
  if (!file.is_open()) {
    return -1;
  }

  byte buf[1024];
  while (file.good()) {
    file.read(buf, sizeof buf);
    std::streamsize count = file.gcount();

    // use data in buf
  }

  return 0;
}
```

```rust,no_run
use std::fs::File;
use std::io::{BufReader, Read};

fn main() -> Result<(), std::io::Error> {
    let mut f = BufReader::new(File::open(
        "/path/to/file",
    )?);

    let mut buf = [0u8; 1024];

    loop {
        let count = f.read(&mut buf)?;
        if count == 0 {
            break;
        }

        // use data in buf
    }

    Ok(())
}
```

</div>

The major difference between the C++ program and the Rust program is that in the
Rust program the buffer must be initialized before it can be used. In most
cases, this one-time initialization cost is not significant. When it is, unsafe
Rust is required to avoid the initialization.

The technique for avoiding initialization makes use of
[`std::mem::MaybeUninit`](https://doc.rust-lang.org/std/mem/union.MaybeUninit.html).
[Examples of safe usage of
`MaybeUninit`](https://doc.rust-lang.org/std/mem/union.MaybeUninit.html#examples)
are given in the API documentation for the type.

The IO API in stable Rust does not include support for `MaybeUninit`. Instead,
there is a [new safe API being developed](#upcoming-changes-and-borrowedbuf)
that will enable avoiding initialization without requiring unsafe Rust in code
that uses the API.

If the callee might need to grow the provided buffer and dynamic allocation is
allowed, then a `&mut Vec<T>` can be used instead of `&mut [T]`. This is similar
to providing a `std::vector<T>&` in C++. To avoid unnecessary reallocation, the
vector can be created using `Vec::<T>::with_capacity(n)`.

## A note on reading files

While the examples here use IO to demonstrate re-using pre-allocated buffers,
there are higher-level interfaces available for reading from `File`s, both from
the [`Read`](https://doc.rust-lang.org/std/io/trait.Read.html) and
[`BufRead`](https://doc.rust-lang.org/std/io/trait.BufRead.html) traits, and
from convenience functions in
[`std::io`](https://doc.rust-lang.org/std/io/index.html#functions-1) and in
[`std::fs`](https://doc.rust-lang.org/std/fs/index.html#functions-1).

The techniques described here are useful, however, in other situations where a
reusable buffer is required, such as when interacting with hardware APIs, when
using existing C or C++ libraries, or when implementing algorithms that produce
larges amount of data in chunks, such as cryptography algorithms.

## Upcoming changes and `BorrowedBuf`

The Rust community is refining approaches to working with uninitialized buffers.
On the nightly branch of Rust, one can use
[`BorrowedBuf`](https://doc.rust-lang.org/std/io/struct.BorrowedBuf.html) to
achieve the same results as when using slices of `MaybeUninit`, but without
having to write any unsafe code. The IO APIs for avoiding unnecessary
initialization use `BorrowedBuf` instead of slices of `MaybeUninit`.
