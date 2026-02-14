---
title: "x86_64 머신에서 aarch64(arm64) Rust for Linux 크로스 컴파일하기"
date: 2022-10-04T02:09:31+09:00
draft: false
categories: ["Linux"]
tags: ["Cross compile", "Rust", "Linux", "ARMv8A", "Korean_Article"]
---

> 10월 6일 기준, Rust for Linux는 linux-next에 있으며 stable이 아닙니다.<br>
> 따라서 이 글은 Linux 6.1 stable이 나오기 전에 구식이 될 수 있습니다.

> 현재 Linux 6.1 rc1에는 ARM64용 Rust for Linux가 포함되어 있지 않습니다.
> 따라서 이 글에서는 https://github.com/Rust-for-Linux/linux/tree/for-next/rust 를 사용합니다.


![Building_Rust_For_Linux_with_cross_compile](img/cross_compiling_aarch64_rust_for_linux_from_x86_64_linux/cat_12.png)

## 소개
 이 글에서는 x86_64 debian 환경에서 Rust for Linux를 크로스 컴파일하는 방법을 설명합니다. Apple Silicon을 제외하면 아직 arm64 네이티브 커널을 빌드할 만한 충분한 컴퓨팅 파워가 없습니다.


참고로 이 글은 다음 링크를 참고하여 작성되었습니다.
- https://github.com/Rust-for-Linux/linux/blob/rust/Documentation/rust/quick-start.rst
- https://docs.kernel.org/kbuild/llvm.html#cross-compiling

### Debian / Ubuntu 패키지 요구사항

```sh
# Install build-requirements for kernel compile with LLVM.
# Biggest difference to native build is
# crossbuild-essential-arm64 needed to build` for arm64
apt install clang git llvm-dev libclang-dev build-essential \
bc kmod cpio flex libncurses5-dev libelf-dev libssl-dev \
dwarves bison lld curl crossbuild-essential-arm64
```
커널 빌드 전에 필요한 패키지를 설치해야 합니다.

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup default 1.62
rustup component add rust-src
# rustfmt and clippy is need for later developing and debugging.
rustup component add rustfmt
rustup component add clippy
```
curl로 Rust를 설치합니다. 기본 옵션을 선택하면 됩니다.
또한 현재 Rust for Linux는 1.62 버전에서 동작합니다.
최신 버전(1.64 테스트 완료)에서도 네이티브 컴파일은 잘 동작하지만, 크로스 컴파일은 동작하지 않습니다.


### Arch 패키지 요구사항
 - TBD


## Rust-For-Linux에서 Linux 클론하기
![Check working branch](img/build_in_m1vm/check_working_branch.png)
_현재 Rust-for-Linux의 상태, 6.0 RC 단계_

```sh
# In my case use `Develop` as worksapce, you can replace this word.
mkdir -p ~/Develop
cd ~/Develop
git clone https://github.com/Rust-for-Linux/linux.git -b rust
```
위와 같이 클론합니다.

## `Rust-For-Linux`에 필요한 Rust 스크립트
클론한 Linux 디렉토리에서 실행합니다.
```sh
git clone --recurse-submodules \
        --branch $(scripts/min-tool-version.sh rustc) \
        https://github.com/rust-lang/rust \
        $(rustc --print sysroot)/lib/rustlib/src/rust
```
이 작업은 `rustlib` 저장소를 Rust 툴체인 디렉토리에 클론합니다.

```sh
cargo install --locked --version $(scripts/min-tool-version.sh bindgen) bindgen
```
이 작업은 기존 C 코드를 Rust 코드에 바인딩하기 위해 필요합니다.

## `RUST_AVAILABLE` 확인
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
위와 같은 결과가 나오면 준비 완료입니다.
(1.62.1에서도 크로스 컴파일이 잘 되었지만, 최적의 호환을 위해서는 `rustup default 1.62.0`을 실행하세요.)

## `menuconfig`로 Linux 소스코드 설정하기
```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig
```
### `General setup -> Rust support`
![Menuconfig Rust Support](img/build_in_m1vm/menuconfig_rust_in_m1vm_check_rust_support.png)
`General setup -> Rust support`에서 이 옵션을 활성화합니다. <br>
만약 해당 플래그가 보이지 않으면 `make LLVM=1 rustavailable` 과정이 성공적으로 완료되었는지 다시 확인하세요. <br>

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
Rust 커널 코드를 쉽게 개발하기 위해 예제가 필요합니다.
다음 메뉴에서 예제를 활성화할 수 있습니다.

|    |    |
| -- | -- |
| ![Menuconfig Rust Sample1](img/build_in_m1vm/menuconfig_rust_in_m1vm_example_add2.png) | ![Menuconfig Rust Sample2](img/build_in_m1vm/menuconfig_rust_in_m1vm_example_add1.png) |

`Kernel hacking -> Sample kernel code`에서 관심 있는 항목을 활성화합니다 (전부 다 활성화할 필요는 없습니다).
직접 드라이버를 작성할 때는 활성화하지 않는 것을 권장합니다. 시스템이 느려지거나 dmesg에 불필요한 로그가 남을 수 있기 때문입니다. <br>
특히 netfilter 예제는 너무 많은 dmesg를 출력하므로, `netfilter` 예제를 공부하는 경우가 아니라면 비활성화하는 것을 권장합니다.
![example too many dmesg log from netfilter example](img/build_in_m1vm/too_many_log.png)<br>

### `Kernel hacking -> Rust hacking`
Rust 커널 코드나 드라이버를 디버깅하려면 일부 디버그 옵션을 활성화해야 합니다.

![Menuconfig Rust Debug](img/build_in_m1vm/menuconfig_rust_in_m1vm_debug.png)

`Kernel hacking -> Rust hacking`에서 해당 메뉴와 하위 메뉴를 활성화합니다.


## 크로스 컴파일

![Build time](img/cross_compiling_aarch64_rust_for_linux_from_x86_64_linux/cross_with_j32.png)

```sh
# -j4 for 4 core virtual machine, -j2 for 2 core, -j1 for single core.
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- LLVM=1 -j32
```
위 명령어로 빌드합니다. 가상 머신에 할당된 코어 수를 고려하여 job 수를 설정해야 합니다. (`-j#`)

빌드 중에 일부 플래그에 대해 질문이 나올 수 있습니다. 저는 기본값을 선택했습니다.

### 간단한 컴파일 속도 비교
| 머신 / 환경 | 컴파일 시간 |
| --------------------- | -----------: |
| M1 Max 가상 머신 (4코어 8GB RAM, aarch64 debian11) | 16분 |
| M1 Asahi Linux (4P+4E 코어 16GB RAM MacMini, 6.1.0-rc6-asahi) | 11분 |
| AMD Ryzen 5950x 네이티브 (16코어 32스레드, 64GB, x86_64)  | 3분  |
| AMD Threadripper Pro 5975wx 네이티브 (32코어 64스레드, 256GB, x86_64)  | 2분  |

## 크로스 컴파일된 커널을 arm64 가상 머신에 설치하기
- TBD, 곧 업데이트 예정입니다.

## 크로스 컴파일된 커널을 라즈베리 파이에 설치하기
- TBD, 곧 업데이트 예정입니다.
