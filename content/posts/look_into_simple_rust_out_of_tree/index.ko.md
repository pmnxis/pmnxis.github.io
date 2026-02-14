---
title: "[Rust Driver] 예제 Rust Linux 드라이버를 빌드해보자"
date: 2022-10-02T21:05:52+09:00
draft: false
categories: ["Linux"]
tags: ["Rust", "Linux", "ARMv8A", "Rust Driver", "Korean_Article"]
---

> 10월 기준, Rust for Linux는 linux-next에 있으며 stable이 아닙니다.<br>
> 따라서 이 글은 Linux 6.1 stable이 나오기 전에 구식이 될 수 있습니다.

## modules, out-of-tree
커널 모듈을 개발하는 방법은 크게 두 가지가 있습니다. In-Of-Tree와 Out-Of-Tree 방식입니다. 이 글에서는 Out-Of-Tree 방식으로 Rust 커널 모듈을 만들어 보겠습니다.

## 시작하기 전에
### 커널이 `CONFIG_RUST=y`로 컴파일되었는지 확인하기

다음 명령어로 확인합니다.
```sh
zcat /proc/config.gz | grep -i CONFIG_RUST=y
```
결과가 `CONFIG_RUST=y`로 나오면 됩니다.

하지만 배포판 커널 이미지를 다운로드하거나 사전 설치된 경우에는 `/proc/config.gz`에서 확인할 수 없을 수도 있습니다.

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

### `$KDIR` 준비
`$KDIR`은 커널 소스의 경로입니다.<br>
이 글에서는 `CONFIG_RUST`로 부팅에 사용된 커널 소스의 경로를 의미합니다.

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

제 경우에는 `~/Develop/linux`입니다.
```sh
# /home/pmnxis/Develop/linux
export KDIR=$HOME/Develop/linux
```

## 코드 살펴보기

코드 [`rust_out_of_tree.rs`](https://github.com/Rust-for-Linux/rust-out-of-tree-module/blob/48c6c6477afc3e53a0d95db29a8facd6d67eff26/rust_out_of_tree.rs) 를 미리 살펴봅시다...

### 라이선스와 import

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
1~3번 줄은 파일의 라이선스 정보를 나타냅니다.
회사에서 코드를 작성하는 경우 `GPL-2.0` 대신 `SomeCompanyName`을 사용하거나 그대로 `GPL-2.0`을 유지합니다.
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

5번 줄은 이 코드에서 사용할 [Rust for Linux 라이브러리](https://rust-for-linux.github.io/docs/kernel/prelude/index.html)를 가져오는 것을 의미합니다.

다음 [예제](https://github.com/Rust-for-Linux/linux/blob/542379556669fee413ad45bc9611187e075b0d47/samples/trace_printk/trace-printk.c#L2-L4)에서 C로 작성된 모듈은 이렇게 include합니다.

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
8번 줄은 module trait의 구현체입니다. <br>
9번 줄은 모듈의 이름으로, C로 작성했을 때 *.ko의 name 필드에 해당합니다.
10~12번 줄은 아래의 C로 작성된 [예제](https://github.com/Rust-for-Linux/linux/blob/542379556669fee413ad45bc9611187e075b0d47/samples/trace_printk/trace-printk.c#L56-L58)와 동일한 목적의 필드입니다.

<!--- todo! fix range, hl_lines had some bug  -->
```c {linenos=table, linenostart=56, hl_lines=["1-59"]}
MODULE_AUTHOR("Steven Rostedt");
MODULE_DESCRIPTION("trace-printk");
MODULE_LICENSE("GPL");
```

`macro_rule! module`을 간단히 살펴보겠습니다. 자세한 내용은 여기를 참고하세요.

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

### 실제 구현
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
 동작을 추측해보면...
1) init (`insmod`?) 시, `Rust out-of-tree sample (init)` 텍스트를 어딘가에 출력
2) `vec<i32>[72, 108, 200]`이 `RustOutOfTree` 구조체와 함께 커널 메모리 공간에 저장됨
3) 모듈을 drop (`rmmod`?) 할 때, `[72, 108, 200]` 텍스트를 출력

여기서 주의 깊게 봐야 할 부분이 있습니다.
<!--- todo! fix range, hl_lines had some bug  -->
```rs {linenos=table, linenostart=23, hl_lines=[2]}
        let mut numbers = Vec::new();
        numbers.try_push(72)?;
```
24번 줄에서 `try_push`는 [`std::Vec`](https://doc.rust-lang.org/std/vec/struct.Vec.html)에는 존재하지 않습니다. Rust 커널 프로그래밍에서는 `std::Vec::push` 대신 [`try_push`](https://rust-for-linux.github.io/docs/kernel/prelude/struct.Vec.html#method.try_push)를 사용해야 합니다.

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
또한 20번 줄과 33번 줄에 `init`과 `drop` 함수가 있습니다. 이 코드는 `impl for` 패턴으로 해당 함수들을 구현하고 있습니다.
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

구현체와 그 철학에 대해서는 이후 글에서 설명하겠습니다.

## 코드 실행
### 빌드하기
```sh
make LLVM=1
```
Rust를 지원하는 커널 빌드가 LLVM으로 이루어졌으므로,<br>
커널 모듈도 LLVM으로 컴파일합니다.

### 모듈 설치
```sh
sudo insmod ./rust_out_of_tree.ko
```
컴파일 후 프로젝트 디렉토리 안에 `rust_out_of_tree.ko`가 생성됩니다.<br>
기존에 사용하던 것처럼 `insmod`로 모듈을 설치할 수 있습니다.

### 결과 확인
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
위 명령어로 실제 결과를 확인할 수 있습니다.

![Insepction Result](img/look_into_simple_rust_out_of_tree/inspect.png)
예상한 대로 `[72, 108, 200]`이 출력됩니다.

----

## 결론

이 간단한 커널 모듈에서 다음과 같은 내용을 정리할 수 있습니다.

### 요약
- 코드 상단에 `use kernel::prelude::*;`를 사용해야 합니다.
- `module!` 매크로로 설명을 정의하고 자신의 구조체를 커널 모듈에 등록합니다.
- `kernel::Module` 템플릿 함수 .... -WIP-
- 커널 프로그래밍에서는 [`std::Vec`](https://doc.rust-lang.org/std/vec/struct.Vec.html) 대신 [`alloc::vec::Vec`](https://rust-for-linux.github.io/docs/kernel/prelude/struct.Vec.html#method.try_push)를 사용합니다.
- `pr_info`는 `C`에서 사용하는 방식과 동일합니다.

----

## 참고
1) https://github.com/Rust-for-Linux/rust-out-of-tree-module
2) https://www.kernel.org/doc/html/latest/kbuild/modules.html
3) https://github.com/Rust-for-Linux/linux
4) https://rust-for-linux.github.io/docs/kernel/prelude/index.html
5) https://rust-for-linux.github.io/docs/kernel/prelude/macro.module.html
6) https://rust-for-linux.github.io/docs/kernel/prelude/struct.Vec.html
