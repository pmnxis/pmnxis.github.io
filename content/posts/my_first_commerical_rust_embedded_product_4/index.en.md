---
title: "Developing a Mass-Produced Rust Embedded Product - 4 Leveraging Build Scripts"
date: 2023-11-13T21:00:00+09:00
draft: false
categories: ["My Frist Mass Production With Rust Embedded"]
tags: ["회고록", "Rust", "Embedded", "Korean_Article"]
---

{{< alert >}}
**WARN!** This article is still a work in progress. Content may change at any time.
{{< /alert >}}

![constcat](feature.jpg)

## Before Reading
This post is a continuation of the previous post [Part 3: Leave It to Compile Time](/posts/my_first_commerical_rust_embedded_product_3), but focuses on `build.rs` rather than const fn/trait/impl.

If you are not familiar enough with Rust syntax, I recommend reading the #Studying-Rust section in the previous post [Part 2: Study Methods and Key Characteristics](/posts/my_first_commerical_rust_embedded_product_2/#러스트-공부). I also recommend the excellent article by Ki-O Kim, [A Typical C Developer's Journey into Rust: A Guide for C Developers Entering Rust](https://wikidocs.net/book/12811).

## Build Scripts (`build.rs`)
Just as C and C++ have Makefiles, Rust has Cargo. Cargo provides `build.rs`, which allows you to easily write scripts that run before compilation begins.

For detailed documentation on `build.rs`, see: [The Cargo Book - Build Scripts](https://doc.rust-lang.org/cargo/reference/build-scripts.html)

`build.rs` runs before compiling the source code in the `src` directory or external libraries. Importantly, even if you are developing code for `no_std`, `build.rs` can use libraries available on the host OS, including `no_std` and the `alloc` crate.

A separate program in `main.rs` must create const data at compile time, but there are cases where the functions and methods you need are not available in const form.

### Fetching Strings with `build.rs` and `env!()`

Let us practice fetching a string (`&str`) at compile time using `build.rs` and the `env!()` macro.

![build.rs project dir](./build_rs_path.png)

The project structure looks like this. `build.rs` is not in the src directory; it goes in the root directory of the project.

#### Cargo.toml Example
```toml
# End of Cargo.toml
[build-dependencies]
chrono = "0.4.31"
```

As mentioned above, `build.rs` can use libraries available for OS targets.
If there is a crate you want to use, add it under `build-dependencies` below `dependencies`.
Here we have added the `chrono` crate for our example.

#### build.rs Example
```rs
// build.rs
fn main() {
    let now = chrono::Utc::now().to_rfc3339();
    println!("cargo:rustc-env=GIVEN_BUILD_TIME={}", now);
}
```

When Cargo compiles and runs `build.rs`, it uses the chrono library to get the current time in UTC and converts it to a `String`.

Then it passes it to rustc (the compiler) as the "GIVEN_BUILD_TIME" environment variable via `println!`.

#### src/main.rs Example
```rs
// src/main.rs
const BUILD_TIME: &str = env!("GIVEN_BUILD_TIME");

fn main() {
    // if you working with firmware environment,
    // use defmt::info!(..) instead of println!(..)
    println!("passed build time is {}", BUILD_TIME);
    // "passed build time is 2023-11-14T06:14:02.366899222+00:00"
}
```

In `src/main.rs`, the "GIVEN_BUILD_TIME" environment variable is retrieved as a `const &str` using the built-in `env!(..)` macro.

The exact module path of `env!(..)` is `core::env!(..)`, and `core::env!(..)` fetches environment variables at compile time.
If you are developing for an OS target and want to get runtime environment variables or execution arguments rather than compile-time ones, you should use the items in the `std::env` module.
Since you may end up using both when working with `build.rs`, be careful not to confuse them.

For detailed documentation on the `env!()` macro, see: https://doc.rust-lang.org/core/macro.env.html

## Creating Fixed-Length Binary Data
### Fetching with `include_bytes`
`core::include_bytes` can load a file from the project directory as binary data at compile time.
https://doc.rust-lang.org/core/macro.include_bytes.html

#### Creating a Dummy Binary
```sh
echo -n "\x00\x01\x02\x03" >> ./src/stuff0123.bin
```
Create a dummy binary for our exercise. It is `[0x00, 0x01, 0x02, 0x03]`, totaling 4 bytes, and is placed in the same location as `main.rs`.

In this post we create the dummy binary from the terminal, but it can also be done from build.rs.

#### src/main.rs Example
```rs
const DATA0123: &[u8; 4] = include_bytes!("stuff0123.bin");

fn main() {
    println!("binary data is {:?}", DATA0123);
}
```
In `src/main.rs`, the binary created earlier is fetched at compile time as `const &[u8; 4]`.
Note that unlike fetching a `&str` above, you must specify the number of elements like `[u8; N]`.

To fetch data without worrying about the number of elements, you need to use macros. This is covered later in this post.

## Fetching Variable-Length Binary Data
| `build.rs`| `src/**.rs`           |
| :-------: | :-------------------: |
| `String`  | `const &str`          |
| `Vec<u8>` | `const [u8; N]`       |

In `build.rs`, data has variable-length characteristics, but when compiling for the target inside `src/**.rs`, you need the help of proc_macro.
However, if the data size changes variably with each compilation and is not strictly controlled, it can be dangerous, so keep this in mind when using it.


### Fetching Variable-Length Binary Data with proc-macro
There is also an approach of saving data to a tmp file and using `include_bytes!` with `proc_macro`, but the method introduced here uses `env!` together with `proc_macro`.

To explain proc_macro: in the past with C, you would use `##` to concatenate tokens to generate code, or use compiler-specific features for black magic. Proc_macro brings in the concepts of tokens and parsers to enable reasonably stable metaprogramming through a macro system.

In Korean it is called procedural macros (jeolchahyeong maekeullo). If you master proc_macro and procedural macros, you can achieve excellent metaprogramming. I will write about this in more detail in a future post.

For the official detailed explanation of proc_macro, see: [Procedural Macros](https://doc.rust-lang.org/beta/reference/procedural-macros.html)

#### Overall Flow
{{<mermaid>}}
stateDiagram-v2
    state "Total flow" as total_flow
    state "data prepare" as data_prepare
    state "hex::encode(..)" as hex_encode
    state "hex encoded" as hex_encoded
    state "passing to env" as passing_to
    state "passing from env" as passing_from
    state "proc_macro" as proc_macro
    state "decoding at build time" as decoding
    state "decoded data" as decoded_data
    state "code generation" as code_generation
    state "src/**.rs" as src_rs
    state "const NAME: [u8; N] = [...]" as const_u8_n

    state total_flow {
        [*] --> build.rs
        state build.rs {
            [*] --> data_prepare
            data_prepare --> hex_encode
            hex_encode --> hex_encoded
            hex_encoded --> passing_to
        }
        --
        state src_rs {
            passing_from --> proc_macro
            state proc_macro {
                [*] --> decoding
                decoding --> decoded_data
                decoded_data --> code_generation
            }
            proc_macro --> const_u8_n
        }
    }
{{< /mermaid >}}

The overall flow is as follows:
1. Prepare the data to inject in build.rs.
2. Encode it as hex and pass it as an environment variable to rustc.
3. In src/**.rs, receive the environment variable and pass it to a function created with proc_macro.
4. The macro function decodes the data at compile time and turns it into an array of the original data.
5. Generate const data as a code declaration with the specified name and the array length.

#### proc_macro Code for Decoding at Compile Time
To perform steps 4 and 5 described above, the following code is needed:

```rs
fn slice_to_auto_sized (
    arr_name: String,
    input: &[u8],
) -> TokenStream {
    format!(
        "const {}: [u8; {}] = [{}];",
        arr_name,
        input.len(),
        input.iter().join(", ")
    )
    .parse::<proc_macro2::TokenStream>()
    .expect("Failed to parse array")
    .into()
}

struct NameAndEnvInput {
    arr_name: syn::LitStr,
    _comma0: Token![,],
    env_var: syn::LitStr,
}

impl Parse for NameAndEnvInput {
    fn parse(input: syn::parse::ParseStream) -> syn::Result<Self> {
        Ok(Self {
            arr_name: input.parse()?,
            _comma0: input.parse()?,
            env_var: input.parse()?,
        })
    }
}

#[proc_macro]
pub fn c(inputs: TokenStream) -> TokenStream {
    let inputs = parse_macro_input!(inputs as NameAndEnvInput);
    slice_to_auto_sized(
        inputs.link_section_name.value(),
        inputs.arr_name.value(),
        hex::decode(std::env::var(inputs.env_var.value()).expect("This env not found"))
            .expect("Can't decode hex")
            .as_slice(),
    )
}
```
As an example, assume that `slice_to_auto_sized!(SOME_DATA, GIVEN_ENV);` is declared in `main.rs`.

In `const_from_hex_env`, it takes three tokens matching the structure of the `NameAndEnvInput` struct: "SOME_DATA", a comma (,), and "GIVEN_ENV".
These tokens are passed to `slice_to_auto_sized` at the top, which generates the code to be produced.

- _The above code can be found in the [forked env-to-array commit](https://github.com/pmnxis/env-to-array/commit/782c2b265d8a23653321d163ac5cea96c04bc85d). The code there has been modified to create dummy sections for the linker, so it is slightly different._
- _This code is a slightly modified fork of the [env-to-array](https://crates.io/crates/env_to_array) crate._


#### Data Generation Example for Hex Encoding
```rs
fn main {
    // https://github.com/pmnxis/billmock-mptool/blob/master/otp-proof-of-concept/build.rs
    // (abbreviated)
    let fingerprint = MpFingerprint {
        firmware_fingerprint: FirmwareFingerprint {
            model_name: main_package.name.clone(), // reference package name temporary
            model_ver: feature_based_model_ver,
            firmware_ver: main_package.version.to_string(),
            firmware_git_hash: format!("{}", commit_hash),
        },
    };

    // cargo objdump --release -- -s --section .mp_fingerprint
    println!(
        "cargo:rustc-env=MP_FINGERPRINT_TOML_HEX={}",
        fingerprint.to_hex_string(),
    );
}
```
When encoding in hex style, you can use [hex](https://crates.io/crates/hex) and encode with `hex::encode()`.

The code above creates package information and a git hash in TOML format, then hex-encodes it.

## Practical Applications

### Application - Fetching EUC-KR Strings at Compile Time
```rs
use encoding_rs::EUC_KR;

fn main() {
    // encoding_rs::Encoding::encode(..) is not const fn
    let ret: &[u8] = EUC_KR.encode("안녕하세요").0.to_vec().as_slice();
}
```

While developing [billmock-app-rs](https://github.com/pmnxis/billmock-app-rs), I encountered several problems when creating an NDA financial institution protocol communication library.
I needed to hold EUC-KR strings as const. However, the EUC-KR string encoding library [encoding-rs](https://crates.io/crates/encoding) does not provide const functions.

Since EUC-KR is ultimately a character encoding, under the assumption that the strings are predetermined, it is binary data that can be generated at compile time.

By combining the techniques introduced above, I wrote code in the financial institution protocol communication library that converts EUC-KR strings to `const [u8; N]` at compile time.

### Application - Dummy ELF Header
During firmware development, when creating an MP Tool (Mass Production Tool) for flashing firmware in bulk, I inserted a dummy header into the ELF to prevent mistakes and record hardware version information. Since this dummy ELF header is data that is not actually loaded into flash, using variable-length data was no problem at all.

The overall structure is similar to what was described above in ["Fetching Variable-Length Binary Data with proc-macro"](#fetching-variable-length-binary-data).

Related commits are as follows:

- https://github.com/pmnxis/billmock-app-rs/pull/42/files
- https://github.com/pmnxis/env-to-array/commit/782c2b265d8a23653321d163ac5cea96c04bc85d

## Wrapping Up
This post is a continuation of the previous post [Part 3: Leave It to Compile Time](/posts/my_first_commerical_rust_embedded_product_3), but focused on `build.rs` rather than const fn/trait/impl.
I covered how to handle fixed-length and variable-length data through `build.rs` and listed practical usage examples.

The topic of proc_macro (procedural macros) was too lengthy to explain in the middle of this post, so I plan to cover it in a future article.