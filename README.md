# `enum_c_helper`

Opinionated helpers for dealing with `#[repr(C)]` enums

Expects enum discriminants in a contiguous range, uses `unsafe`, and tries to use declarative macros whenever possible.

## tl;dr

Given the following:

```rust
# use enum_c_helper::enum_c_helper;
enum_c_helper! {
    #[repr(C, u8)]
    enum ExampleEnum {
        VariantOne,
        VariantTwo(u8, u16),
        VariantThree { x: u32, y: u32 },
    }
}
```

The following output is generated:

```rust
# use enum_c_helper::*;
#[repr(C, u8)]
enum ExampleEnum {
    VariantOne,
    VariantTwo(u8, u16),
    VariantThree { x: u32, y: u32 },
}

#[repr(u8)]
#[derive(Clone, Copy, PartialEq, Eq, PartialOrd, Ord, Debug, Hash)]
enum ExampleEnumDiscriminant {
    VariantOne,
    VariantTwo,
    VariantThree,
}

unsafe impl EnumHelped for ExampleEnum {
    type DiscriminantTy = ExampleEnumDiscriminant;
}

unsafe impl EnumDiscriminant for ExampleEnumDiscriminant {
    const NUM_VARIANTS: u128 = 3;
    type ReprTy = u8;
    type FancyTy = ExampleEnum;
}
```

This is useful so that you can:

```rust
# use enum_c_helper::*;
# enum_c_helper! {
#     #[repr(C, u8)]
#     enum ExampleEnum {
#         VariantOne,
#         VariantTwo(u8, u16),
#         VariantThree { x: u32, y: u32 },
#     }
# }
fn thing_as_u8(x: ExampleEnum) -> u8 {
    x.as_int()
}
fn thing_as_u32(x: ExampleEnum) -> u32 {
    x.as_any_int()
}
fn thing_kind(x: u8) -> ExampleEnumDiscriminant {
    <_>::from_int(x)
}
fn how_many() {
    println!("There are {} variants", ExampleEnum::NUM_VARIANTS);
}
```
