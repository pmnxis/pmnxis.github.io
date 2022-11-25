---
title: "Rust For Linux Development Environment with AppleSilicon MacOS"
date: 2022-10-01
draft: false
category: "Linux"
tags: ["Rust", "Linux", "ARMv8A", "English_Article"]
---

> In October 1, Rust for linux is under the linux-next, not stable<br>
> Thus this article would be out-of-date before Linux 6.1 stable comes.

> This article play with https://github.com/Rust-for-Linux/linux/tree/for-next/rust


![Building_Rust_For_Linux_In_M1_Mac](img/build_in_m1vm/build_rust_linux_in_m1.png)

## Introduction
 Currently Apple Silicon mac series is only one ARM workstation that have powerful performance as normal desktop class workstation and can purchase anywhere.
Of course if you have 32GB or bigger memory and least 8 big cores of apple silicon.

Btw, this article is in reference to https://github.com/Rust-for-Linux/linux/blob/rust/Documentation/rust/quick-start.rst .

## VM hypervisor software selection.
 - UTM : Free / OpenSource, QEMU based Sometimes tricky.
 - VM Fusion Tech Preview : Free for now / ClosedSource, Moderate
 - Parallels : Non-Free / ClosedSource, not my taste (sorry).

There's some option working with [Asahi Linux](https://asahilinux.org/). But in this article is not consider native asahi linux environment.

In my case, I was chosen VM Fusion.

## Virtual Machine Configuration
- Debian 11 : https://cdimage.debian.org/debian-cd/current/arm64/iso-dvd/ <br>
    !! Checked working well.

- Ubuntu : <br>
    There were some issue `clang` and other gcc build tools version mismatch than broken apt things in `aarch64` ubuntu apt repo.
    But you can try with ubuntu.

- Arch Linux : https://gitlab.archlinux.org/tpowa/archboot/-/wikis/Archboot-Homepage#aarch64-architecture <br>
    I didn't tested yet. But tested with Asahi linux with M1 Mac Mini


### Debian / Ubuntu Package Requirements

```sh
# Install build-requirements for kernel compile with LLVM.
apt install clang git llvm-dev libclang-dev build-essential \
bc kmod cpio flex libncurses5-dev libelf-dev libssl-dev \
dwarves bison lld curl
```
### Asahi Linux Package Requirements

```sh
pacman -S base-devel cpio lld llvm llvm-libs bc libdwarf
```

### Ready for rust

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
***   Your version:     1.64.0
***   Expected version: 1.62.0
***
Rust is available!
```
Than if you get result like this it's good to go

## Configure linux source code with `menuconfig`
```
make ARCH=arm64 defconfig
make menuconfig
```

## Disable `GCC plugins`
![Menuconfig Rust Support](img/build_in_m1vm/disable_gcc_plugins.png)
`General architecture-dependent options -> GCC plugins`
For now (6.1 rc*), `GCC_PLUGINS` config should be disabled for `RUST_CONFIG` config.
Be sure disable it.

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


## Compile and install it in virtual machine.

![Build time](img/build_in_m1vm/build_make_LLVM.png)

```sh
# -j4 for 4 core virtual machine, -j2 for 2 core, -j1 for single core.
make LLVM=1 -j4
```
Build with following command. You need to set number of job considering assigned number of cores for virtual machine. (`-j#`)

Also while you build it, it will ask some flag. I just select default in my case.

It takes lot of time (don't worry much better than raspberry pi 4), 13~14 minuites takes in my environment (VM 4core, 8GB)

After than, install via following command
```sh
# should be under the root permision.
make modules_install
make install
update-grub
```
![make install rust-kernel](img/build_in_m1vm/make_install.png)

It's done!. <br>
Reboot program and then check the kernel working well.


```
Linux lambda-next 6.0.0-rc7-175589-g542379556669 #2 SMP PREEMPT Sun Oct 2 19:02:32 KST 2022 aarch64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Oct  2 18:20:21 2022 from 192.168.99.1
pmnxis@lambda-next:~$ uname -r
6.0.0-rc7-175589-g542379556669
```

### Simple compile speed comparation.
| Machine / Environment | Compile time |
| --------------------- | -----------: |
| M1 Max Virtual Machine (4 core 8GB RAM with aarch64 debian11) | 16 minutes |
| M1 Asahi Linux (4P+4E core 16GB RAM MacMini with 6.1.0-rc6-asahi) | 11 minutes |
| AMD Ryzen 5950x Native (16 core 32 thread, 64GB with x86_64)  | 3 minutes  |
| AMD Threadripper Pro 5975wx Native (32 core 64 thread, 256GB with x86_64)  | 2 minutes  |
