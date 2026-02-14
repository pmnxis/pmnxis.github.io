---
title: "Developing a Mass-Produced Rust Embedded Product - 3 Leave It to Compile Time"
date: 2023-11-12T21:00:00+09:00
draft: false
categories: ["My Frist Mass Production With Rust Embedded"]
tags: ["회고록", "Rust", "Embedded", "Korean_Article"]
---

{{< alert >}}
**WARN!** This article is still a work in progress. Content may change at any time.
{{< /alert >}}

![constcat](feature.jpg)

## Introduction
Before reading this post, if you are not familiar enough with Rust syntax, I recommend reading the #Studying-Rust section in the previous post [Part 2: Study Methods and Key Characteristics](/posts/my_first_commerical_rust_embedded_product_2/#러스트-공부). I also recommend the excellent article by Ki-O Kim, [A Typical C Developer's Journey into Rust: A Guide for C Developers Entering Rust](https://wikidocs.net/book/12811).

## Leave It to Compile Time
A typical function will use a constant (immutable value) directly if the result can be predicted at compile time.

However, if storing the value as a constant is deemed inefficient, it remains as a function in the instruction stream.

In embedded systems, especially firmware running on low-cost MCUs, RAM is very limited. (The MCU I used, `STM32G030C8`, has only 8KiB.)

To leave complex computations as constant-like data at compile time and load them into the Flash region (.text or .rodata), I occasionally departed from typical Rust programming practices and wrote code with this in mind.

### `const fn`
Unlike a regular `fn`, a `const fn` guarantees that constant-like data is obtained at compile time.

```rs
const fn const_add(a: i32, b: i32) -> i32 {
    a + b
}

const fn const_add_round_up(a: i32, b: i32) -> i32 {
    let added = const_add(a, b); // Functions inside must also be const
    (added / 10) * 10
}
```

However, there are many constraints. Functions within a `const fn` scope must also be `const fn`, and other operations must also be computable at compile time.

Conversely, a `const fn` can be used inside a regular `fn` or `async fn` scope.

### `const impl`
Making a regular function `const` is straightforward, but the moment you turn it into a method for a struct, you run into difficult problems.
This topic will be covered in the `const trait` section below.

For now, let us look at the simplest case: defining a default for a specific struct.

```rs
pub enum Player {
    Undefined = 0,
    Player1 = 1,
    Player2 = 2,
}

impl Player {
    pub const fn default() -> Self {
        Self::Undefined
    }
}
```

The code ends up being very simple, but by doing this, the caller of `default()` can obtain a constant at compile time.
For more complex cases, you can supply more complex values. As mentioned above, as long as the arguments are not dynamic, the function will not remain in function form, so you are guaranteed to skip branching, stack back-and-forth, and the process of backing up and restoring registers.

### `const trait`
Now we arrive at the much-anticipated `const trait`, which at the time of writing (November 2023) is a **nightly** feature.
From here, I will also describe why `const fn / impl / trait` is still an RFC under discussion in Rust.

#### Conflicts with Existing Traits
Did you notice anything odd about the `pub const fn default()` mentioned above?

The issue is that a [`Default` trait](https://doc.rust-lang.org/core/default/trait.Default.html) already exists separately.

However, the definition of the [`Default` trait](https://doc.rust-lang.org/core/default/trait.Default.html) is not in `const` form.

It is obvious that the `core::default::Default` trait impl for many structs is trivially simple.
But no matter how simple it looks by eye, in this case it cannot be used inside a `const fn`.

#### The Case of `Into/From`
So far there has been little need to abstract through the `Default` trait using `const`, but surprisingly, [Into/From](https://doc.rust-lang.org/core/convert/trait.Into.html) turned out to be the real problem.

For now, by modifying the trait definition of [`core::default::From`](https://doc.rust-lang.org/src/core/convert/mod.rs.html#704-728) into `ConstInto` and `ConstFrom` compatible with const traits, you can use them as follows:
  > - [ConstConvert definition used in the product](https://github.com/pmnxis/billmock-app-rs/blob/0.3.1/src/types/const_convert.rs)

**Example of using const-style Into/From**
```rs
pub struct UnpackedQuad4Bits {
    pub b0: u8,
    pub b1: u8,
    pub b2: u8,
    pub b3: u8,
}

#[derive(PartialEq)]
pub struct PackedQuad4Bits {
    inner: u16,
}

impl const ConstFrom<UnpackedQuad4Bits> for PackedQuad4Bits {
    fn const_from(value: UnpackedQuad4Bits) -> Self {
        PackedQuad4Bits {
            inner: ((value.b0 as u16 & 0xF) << 12)
                | ((value.b1 as u16 & 0xF) << 8)
                | ((value.b2 as u16 & 0xF) << 4)
                | (value.b3 as u16 & 0xF),
        }
    }
}

#[test]
fn test() {
        assert_eq!(
        PackedQuad4Bits::const_from(UnpackedQuad4Bits {
            b0: 0x0, b1: 0x1, b2: 0x2, b3: 0x3 }),
        PackedQuad4Bits { inner: 0x0123 }
    );
}
```

When you create ConstInto / ConstFrom in const form, you cannot use them directly via `.into()` as with regular Into/From. You need to call the preceding const method again from the Into/From trait definition, but you can define the `into` conversion at compile time.

#### `const trait` as a Nightly Feature
`const trait` is still a nightly feature. In my personal experience, I had to change feature flags every time I slightly updated the nightly compiler version.
Nevertheless, it is a very necessary feature in certain cases. (Of course, you can get by using only `const fn` without it.)

When defining a `const trait`, you need to add `#[const_trait]` before the trait definition and declare `#![feature(const_trait_impl)]` in `lib.rs` or `main.rs`.

{{< alert >}}
**todo!** Write about the history of discussions around `const trait`
{{< /alert >}}

- [Tracking issue for RFC 2632, impl const Trait for Ty and ~const (tilde const) syntax](https://github.com/rust-lang/rust/issues/67792)

-------------------------------
{{< alert icon="fire" cardColor="#4f5a63" iconColor="#1d3557" textColor="#f1faee" >}}
Check out other posts in this series:
[Developing a Mass-Produced Rust Embedded Product](/categories/my-frist-mass-production-with-rust-embedded/)
{{< /alert >}}