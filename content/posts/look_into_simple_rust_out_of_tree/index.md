---
title: "[Rust Driver] Let's try build example rust linux driver."
date: 2022-10-02T21:05:52+09:00
draft: false
category: ["Linux"]
tags: ["Rust", "Linux", "ARMv8A", "Rust Driver", "English_Article"]
---

> In October, Rust for linux is under the linux-next, not stable<br>
> Thus this article would be out-of-date before Linux 6.1 stable comes.

## modules, out-of-tree
There are two main ways to develop kernel modules. In-Of-Tree and Out-Of-Tree. In this article, we're going to make the Out-Of-Tree method a Rust kernel module.

## Before we start
### Check your kernel has been compiled with `CONFIG_RUST=y`.

Check with following command.
```sh
zcat /proc/config.gz | grep -i CONFIG_RUST=y
```
The result comes with `CONFIG_RUST=y`.

But you may not check from `/proc/config.gz` when using distibution kernel image that downloaded or pre-installed.

<!--- Some emphasize block with HTML block  -->
<div class="flex px-4 py-2 mb-8 text-base rounded-md bg-primary-100 dark:bg-primary-900">
  <span class="flex items-center ltr:pr-3 rtl:pl-3 text-primary-400">
    {{< icon "triangle-exclamation" >}}
  </span>
  <span class="flex items-center justify-between grow dark:text-neutral-300">
    <span class="prose dark:prose-invert">Need some build & install rust support kernel see here.</span>
    <a href="https://pmnxis.github.io/posts/rust_for_linux_with_m1/" class="px-4 !text-neutral !no-underline rounded-md bg-primary-600 hover:!bg-primary-500 dark:bg-primary-800 dark:hover:!bg-primary-700">
      See details &rharu;
    </a>
  </span>
</div>
<!--- End of the block  -->

### Prepare `$KDIR`
`$KDIR` is path of kernel source.<br>
In this article path of kernel source that system used for boot with `CONFIG_RUST`.

<!--- Some emphasize block with HTML block  -->
<div class="flex px-4 py-2 mb-8 text-base rounded-md bg-primary-100 dark:bg-primary-900">
  <span class="flex items-center ltr:pr-3 rtl:pl-3 text-primary-400">
    {{< icon "triangle-exclamation" >}}
  </span>
  <span class="flex items-center justify-between grow dark:text-neutral-300">
    <span class="prose dark:prose-invert">KDIR and other kernel module descriptions</span>
    <a href="https://www.kernel.org/doc/html/latest/kbuild/modules.html#options" class="px-4 !text-neutral !no-underline rounded-md bg-primary-600 hover:!bg-primary-500 dark:bg-primary-800 dark:hover:!bg-primary-700">
      See details &rharu;
    </a>
  </span>
</div>
<!--- End of the block  -->

In my case it's `~/Develop/linux`
```sh
# /home/pmnxis/Develop/linux
export KDIR=$HOME/Develop/linux
```

## Looking in to code

Let's preview the code [`rust_out_of_tree.rs`](https://github.com/Rust-for-Linux/rust-out-of-tree-module/blob/48c6c6477afc3e53a0d95db29a8facd6d67eff26/rust_out_of_tree.rs) ...

### License and imports

```rs {linenos=table, hl_lines=["1-3"], linenostart=1}
// SPDX-License-Identifier: GPL-2.0

//! Rust out-of-tree sample

use kernel::prelude::*;

module! {
    type: RustOutOfTree,
    name: "rust_out_of_tree",
    author: "Rust for Linux Contributors",
    description: "Rust out-of-tree sample",
    license: "GPL",
}
```
Lines 1~3, show file's license information.
If you are write the code in company, `SomeCompanyName` instead `GPL-2.0` or just keep `GPL-2.0`.
<br><br>

```rs {linenos=table, hl_lines=[5], linenostart=1}
// SPDX-License-Identifier: GPL-2.0

//! Rust out-of-tree sample

use kernel::prelude::*;

module! {
    type: RustOutOfTree,
    name: "rust_out_of_tree",
    author: "Rust for Linux Contributors",
    description: "Rust out-of-tree sample",
    license: "GPL",
}
```

Line 5 means, bring  [rust for linux library](https://rust-for-linux.github.io/docs/kernel/prelude/index.html)  for this code.

In following [example](https://github.com/Rust-for-Linux/linux/blob/542379556669fee413ad45bc9611187e075b0d47/samples/trace_printk/trace-printk.c#L2-L4) module written in C were include like this.

<!--- todo! fix range, hl_lines had some bug  -->
```c {linenos=table, hl_lines=["1-4"], linenostart=2}
#include <linux/module.h>
#include <linux/kthread.h>
#include <linux/irq_work.h>
```
<br><br>

```rs {linenos=table, hl_lines=["7-12"], linenostart=1}
// SPDX-License-Identifier: GPL-2.0

//! Rust out-of-tree sample

use kernel::prelude::*;

module! {
    type: RustOutOfTree,
    name: "rust_out_of_tree",
    author: "Rust for Linux Contributors",
    description: "Rust out-of-tree sample",
    license: "GPL",
}
```
Line 8, implement of the module trait. <br>
Line 9, name of the module, if we written c, it's the name of *.ko name field.
Line 10~12, those fields are simillar with below the [example](https://github.com/Rust-for-Linux/linux/blob/542379556669fee413ad45bc9611187e075b0d47/samples/trace_printk/trace-printk.c#L56-L58) written in c. Those fields are same purpose.

<!--- todo! fix range, hl_lines had some bug  -->
```c {linenos=table, linenostart=56, hl_lines=["1-59"]}
MODULE_AUTHOR("Steven Rostedt");
MODULE_DESCRIPTION("trace-printk");
MODULE_LICENSE("GPL");
```

We preview `macro_rule! module` shortly. You can see detail here.

<!--- Some emphasize block with HTML block  -->
<div class="flex px-4 py-2 mb-8 text-base rounded-md bg-primary-100 dark:bg-primary-900">
  <span class="flex items-center ltr:pr-3 rtl:pl-3 text-primary-400">
    {{< icon "triangle-exclamation" >}}
  </span>
  <span class="flex items-center justify-between grow dark:text-neutral-300">
    <span class="prose dark:prose-invert">Details for <code id="layout">module!</code></span>
    <a href="https://rust-for-linux.github.io/docs/kernel/prelude/macro.module.html#supported-argument-types" class="px-4 !text-neutral !no-underline rounded-md bg-primary-600 hover:!bg-primary-500 dark:bg-primary-800 dark:hover:!bg-primary-700">
      See details &rharu;
    </a>
  </span>
</div>
<!--- End of the block  -->

### Actual implements
```rs {linenos=table, linenostart=15}
struct RustOutOfTree {
    numbers: Vec<i32>,
}

impl kernel::Module for RustOutOfTree {
    fn init(_name: &'static CStr, _module: &'static ThisModule) -> Result<Self> {
        pr_info!("Rust out-of-tree sample (init)\n");

        let mut numbers = Vec::new();
        numbers.try_push(72)?;
        numbers.try_push(108)?;
        numbers.try_push(200)?;

        Ok(RustOutOfTree { numbers })
    }
}

impl Drop for RustOutOfTree {
    fn drop(&mut self) {
        pr_info!("My numbers are {:?}\n", self.numbers);
        pr_info!("Rust out-of-tree sample (exit)\n");
    }
}
```
 I just guess working as ...
1) On init (`insmod`?), print out somewhere with text `Rust out-of-tree sample (init)`
2) `vec<i32>[72, 108, 200]` is stored some kernel memory space with struct `RustOutOfTree`.
3) When drop the module (`rmmod`?), will print out with text `[72, 108, 200]`. 

By the way, we need to keep on eyes here.
<!--- todo! fix range, hl_lines had some bug  -->
```rs {linenos=table, linenostart=23, hl_lines=[2]}
        let mut numbers = Vec::new();
        numbers.try_push(72)?;
```
In line 24, `try_push` is not exsting in [`std::Vec`](https://doc.rust-lang.org/std/vec/struct.Vec.html). In rust kernel programming, need to use [`try_push`](https://rust-for-linux.github.io/docs/kernel/prelude/struct.Vec.html#method.try_push) instead `std::Vec::push`.

<!--- Some emphasize block with HTML block  -->
<div class="flex px-4 py-2 mb-8 text-base rounded-md bg-primary-100 dark:bg-primary-900">
  <span class="flex items-center ltr:pr-3 rtl:pl-3 text-primary-400">
    {{< icon "triangle-exclamation" >}}
  </span>
  <span class="flex items-center justify-between grow dark:text-neutral-300">
    <span class="prose dark:prose-invert">Details for <code id="layout">alloc::vec::Vec</code></span>
    <a href="https://rust-for-linux.github.io/docs/kernel/prelude/struct.Vec.html" class="px-4 !text-neutral !no-underline rounded-md bg-primary-600 hover:!bg-primary-500 dark:bg-primary-800 dark:hover:!bg-primary-700">
      See details &rharu;
    </a>
  </span>
</div>
<!--- End of the block  -->

<br>
Also there's some `init` and `drop` functions in line 20 and 33. The code covers those function with `impl for` pattern.
<!--- Some emphasize block with HTML block  -->
<div class="flex px-4 py-2 mb-8 text-base rounded-md bg-primary-100 dark:bg-primary-900">
  <span class="flex items-center ltr:pr-3 rtl:pl-3 text-primary-400">
    {{< icon "triangle-exclamation" >}}
  </span>
  <span class="flex items-center justify-between grow dark:text-neutral-300">
    <span class="prose dark:prose-invert">Details for Implementation in rust</span>
    <a href="https://doc.rust-lang.org/rust-by-example/generics/impl.html" class="px-4 !text-neutral !no-underline rounded-md bg-primary-600 hover:!bg-primary-500 dark:bg-primary-800 dark:hover:!bg-primary-700">
      See details &rharu;
    </a>
  </span>
</div>
<!--- End of the block  -->

I will explain about implementation and it's philosophy later article.

## Run code
### Build it
```sh
make LLVM=1
```
My rust acceptable kernel build were buiten with LLVM.<br>
So I compile the kernel module with LLVM.

### Install module
```sh
sudo insmod ./rust_out_of_tree.ko
```
After compile, we can there's `rust_out_of_tree.ko` inside of project directory.<br>
We can install module with `insmod` that normally used before.

### Inspect result
```sh
# do `sudo rmmod rust_out_of_tree` if you already install the module`

# clear all of dmesg log 
sudo dmesg -C

# install the module
sudo insmod ./rust_out_of_tree.ko

# see log
dmesg

# uninstall the module
sudo rmmod rust_out_of_tree

# check log again.
dmesg
```
We can check the inspect actual result with above commands.

![Insepction Result](img/look_into_simple_rust_out_of_tree/inspect.png)
As we guess it prints with `[72, 108, 200]`.

----

## Conclusion

We can summary from this simple kernel module.

### Summary
- Need to use `use kernel::prelude::*;` on top of code.
- `module!` macro to define some description and board my own struct to the kernel module.
- `kernel::Module` templete functions .... -WIP-
- In kernel programming, use [`alloc::vec::Vec`](https://rust-for-linux.github.io/docs/kernel/prelude/struct.Vec.html#method.try_push) instead [`std::Vec`](https://doc.rust-lang.org/std/vec/struct.Vec.html).
- `pr_info` is just same as way to write with `C`.

----

## Reference
1) https://github.com/Rust-for-Linux/rust-out-of-tree-module
2) https://www.kernel.org/doc/html/latest/kbuild/modules.html
3) https://github.com/Rust-for-Linux/linux
4) https://rust-for-linux.github.io/docs/kernel/prelude/index.html
5) https://rust-for-linux.github.io/docs/kernel/prelude/macro.module.html
6) https://rust-for-linux.github.io/docs/kernel/prelude/struct.Vec.html