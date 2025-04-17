# Enums

In C++, enums are often used to model fixed set alternatives, especially when
    each of those enumerators maps to a specific integer value, such as is needed
when working with hardware, system calls, or protocol implementations.

For example, the various modes for a GPIO pin could be modeled as an enum, which
would restrict methods using the mode to valid values.

```c++
#include <cstdint>

enum Pin : uint8_t {
    Pin1 = 0x01,
    Pin2 = 0x02,
    Pin3 = 0x04
};

enum Mode : uint8_t {
    Output            = 0x03,
    Pullup            = 0x04,
    Analog            = 0x27
    // ...
};

void lowLevelSetPin(uint8_t pin, uint8_t mode);

void setPinMode(Pin pin, Mode mode) {
    lowLevelSetPin(pin, mode);
}
```

While Rust enums are [more general](./idioms/data_modeling/tagged_unions.md),
they also can be used for this sort of modeling.

```rust
#[repr(u8)]
#[derive(Clone, Copy)]
enum Pin {
    Pin1 = 0x01,
    Pin2 = 0x02,
    Pin3 = 0x04,
}

#[repr(u8)]
#[derive(Clone, Copy)]
enum Mode {
    Output = 0x03,
    Pullup = 0x04,
    Analog = 0x27,
    // ...
}

extern "C" {
    fn low_level_set_pin(pin: u8, mode: u8);
}

fn set_pin_mode(pin: Pin, mode: Mode) {
    unsafe { low_level_set_pin(pin as u8, mode as u8) };
}
```

The `#[repr(u8)]` attribute ensures that the representation of the enum is the
same as a byte (like declaring the underlying type of an enum in C++). The enum
values can then be freely converted to the underlying type with the `as`.

In C++ the standard way to convert from an integer to an enum is a static cast.
However, this [requires that the user check the validity of the cast
themselves](https://www.open-std.org/jtc1/sc22/wg21/docs/cwg_defects.html#1766).
Often the conversion is done by a function that checks that the value to convert
is a valid enum value.

```c++
Pin cast_to_pin(uint8_t pin) {
    return static_cast<Pin>(pin);
}

struct InvalidPin {
    uint8_t pin;
};

Pin to_pin(uint8_t pin) {
    switch (pin) {
	case 0x1: { return Pin1; }
	case 0x2: { return Pin2; }
    case 0x4: { return Pin3; }
	default: { throw InvalidPin{pin}; }
    }
}
```

In Rust the standard way to perform the conversion is to implement the `TryFrom`
trait for the type.

```rust
# #[repr(u8)]
# #[derive(Clone, Copy)]
# enum Pin {
#     Pin1 = 0x01,
#     Pin2 = 0x02,
#     Pin3 = 0x04,
# }
#
# #[repr(u8)]
# #[derive(Clone, Copy)]
# enum Mode {
#     Output = 0x03,
#     Pullup = 0x04,
#     Analog = 0x27,
#     // ...
# }

use std::convert::TryFrom;

struct InvalidPin(u8);

impl TryFrom<u8> for Pin {
    type Error = InvalidPin;

    fn try_from(value: u8) -> Result<Self, Self::Error> {
       match value {
           0x01 => Ok(Pin::Pin1),
           0x02 => Ok(Pin::Pin2),
           0x04 => Ok(Pin::Pin3),
           pin => Err(InvalidPin(pin)),
       }
    }
}
```

See [Exceptions and error handling](/idioms/exceptions.md) for examples of how
to ergonomically handle the result of `try_from`.

If low-level performance is more of a concern than memory safety,
`std::mem::transmute` is analogous to a C++ reinterpret cast, but requires
unsafe Rust because its use can result in undefined behavior. Uses of
`std::mem::transmute` for this purpose should not be hidden behind an interface
that can be called from safe Rust unless the interface can actually guarantee
that the call will never happen with an invalid value.
