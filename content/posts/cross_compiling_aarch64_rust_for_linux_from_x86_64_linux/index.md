---
title: "Cross compiling aarch64(arm64) rust for linux from x86_64 machine"
date: 2022-10-04T02:09:31+09:00
draft: false
category: ["Linux"]
tags: ["Cross compile", "Rust", "Linux", "ARMv8A", "English_Article"]
---

> In October 6, Rust for linux is under the linux-next, not stable<br>
> Thus this article would be out-of-date before Linux 6.1 stable comes.

> Current linux 6.1 rc1 doesn't contain rust for linux with ARM64.
> Thus this article play with https://github.com/Rust-for-Linux/linux/tree/for-next/rust


![Building_Rust_For_Linux_with_cross_compile](img/cross_compiling_aarch64_rust_for_linux_from_x86_64_linux/cat_12.png)

## Introduction
 This article describes cross-compiling rust for linux on x86_64 debian. There is still not enough computing power to build arm64 native kernel except for Apple Silicon.


Btw, this article is in reference to these links
- https://github.com/Rust-for-Linux/linux/blob/rust/Documentation/rust/quick-start.rst
- https://docs.kernel.org/kbuild/llvm.html#cross-compiling

### Debian / Ubuntu Package Requirements

```sh
# Install build-requirements for kernel compile with LLVM.
# Biggest difference to native build is
# crossbuild-essential-arm64 needed to build` for arm64
apt install clang git llvm-dev libclang-dev build-essential \
bc kmod cpio flex libncurses5-dev libelf-dev libssl-dev \
dwarves bison lld curl crossbuild-essential-arm64
```
Before build kernel, we need to install some packages.

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup default 1.62
rustup component add rust-src
# rustfmt and clippy is need for later developing and debugging.
rustup component add rustfmt 
rustup component add clippy
```
Install rust with curl. You can select just default options.
Also current rust for linux working with 1.62.
Some native compile is working well with recent version (1.64 tested, but cross compile not working).


### Arch Package Requirements
 - TBD


## Clone linux from Rust-For-Linux 
![Check working branch](img/build_in_m1vm/check_working_branch.png)
_State of current rust-for-linux, they are under 6.0 RC_

```sh
# In my case use `Develop` as worksapce, you can replace this word. 
mkdir -p ~/Develop
cd ~/Develop
git clone https://github.com/Rust-for-Linux/linux.git -b rust
```
clone like this.

## Necessary some rust scripts in `Rust-For-Linux`
In cloned linux directory.
```sh
git clone --recurse-submodules \
        --branch $(scripts/min-tool-version.sh rustc) \
        https://github.com/rust-lang/rust \
        $(rustc --print sysroot)/lib/rustlib/src/rust
```
This work clone `rustlib` repository in your rust toolchain directory.

```sh
cargo install --locked --version $(scripts/min-tool-version.sh bindgen) bindgen
```
This work need to bind existing c code to rust code.
s
## Check `RUST_AVAILABLE`
```sh
cd ~/Develop/linux
make LLVM=1 rustavailable
```

```
$ make LLVM=1 rustavailable
***
*** Rust compiler 'rustc' is too new. This may or may not work.
***   Your version:     1.62.1
***   Expected version: 1.62.0
***
Rust is available!
```
Than if you get result like this it's good to go
(1.62.1 was fine to cross compile, but if you consider best fit, run `rustup default 1.62.0`.)

## Configure linux source code with `menuconfig`
```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig
```
### `General setup -> Rust support`
![Menuconfig Rust Support](img/build_in_m1vm/menuconfig_rust_in_m1vm_check_rust_support.png)
In `General setup -> Rust support` , Enable this <br>
If you don't see the flag, double-check that the `make LLVM=1 rustavailable` process was successful. <br>

<!--- Some emphasize block with HTML block  -->
<div class="flex px-4 py-2 mb-8 text-base rounded-md bg-primary-100 dark:bg-primary-900">
  <span class="flex items-center ltr:pr-3 rtl:pl-3 text-primary-400">
    {{< icon "triangle-exclamation" >}}
  </span>
  <span class="flex items-center justify-between grow dark:text-neutral-300">
    <span class="prose dark:prose-invert">For a detailed mailing thread on <code id="layout">CONFIG_RUST</code> see here.</span>
    <a href="https://lore.kernel.org/lkml/20220317181032.15436-17-ojeda@kernel.org/" class="px-4 !text-neutral !no-underline rounded-md bg-primary-600 hover:!bg-primary-500 dark:bg-primary-800 dark:hover:!bg-primary-700">
      See details &rharu;
    </a>
  </span>
</div>
<!--- End of the block  -->


### `Kernel hacking -> Sample kernel code`
For easy to develop rust kernel code we need some examples.
You can get them with following menus.

|    |    |
| -- | -- |
| ![Menuconfig Rust Sample1](img/build_in_m1vm/menuconfig_rust_in_m1vm_example_add2.png) | ![Menuconfig Rust Sample2](img/build_in_m1vm/menuconfig_rust_in_m1vm_example_add1.png) |

In `Kernel hacking -> Sample kernel code` , enable it (not all of them..) when you interest.
I don't recommend you enable them when you write own driver. Because there's some possibility make system slow or make unwanted log in dmesg. <br>
In particular, the netflitter example outputs too many dmesg, so it is recommended that you disable it unless you are studying the `netfilter` example.
![example too many dmesg log from netfilter example](img/build_in_m1vm/too_many_log.png)<br>

### `Kernel hacking -> Rust hacking`
For debug rust kernel code or driver, need to enable some debug options.

![Menuconfig Rust Debug](img/build_in_m1vm/menuconfig_rust_in_m1vm_debug.png)

In `Kernel hacking -> Rust hacking` , enables it and inside menus.


## Cross compile

![Build time](img/cross_compiling_aarch64_rust_for_linux_from_x86_64_linux/cross_with_j32.png)

```sh
# -j4 for 4 core virtual machine, -j2 for 2 core, -j1 for single core.
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 -j32
```
Build with following command. You need to set number of job considering assigned number of cores for virtual machine. (`-j#`)

Also while you build it, it will ask some flag. I just select default in my case.

### Simple compile speed comparation.
| Machine / Environment | Compile time |
| --------------------- | -----------: |
| M1 Max Virtual Machine (4 core 8GB RAM with aarch64 debian11) | 16 minutes |
| M1 Asahi Linux (4P+4E core 16GB RAM MacMini with 6.1.0-rc6-asahi) | 11 minutes |
| AMD Ryzen 5950x Native (16 core 32 thread, 64GB with x86_64)  | 3 minutes  |
| AMD Threadripper Pro 5975wx Native (32 core 64 thread, 256GB with x86_64)  | 2 minutes  |

## Install cross compiled kernel to arm64 virtual machine
- TBD, will update asap.

## Install cross compiled kernel to raspberry pi
- TBD, will update asap.
