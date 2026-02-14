---
title: "Apple Silicon MacOS에서 Rust For Linux 개발 환경 구축하기"
date: 2022-10-01
draft: false
categories: ["Linux"]
tags: ["Rust", "Linux", "ARMv8A", "Korean_Article"]
---

> 10월 1일 기준, Rust for Linux는 linux-next에 있으며 stable이 아닙니다.<br>
> 따라서 이 글은 Linux 6.1 stable이 나오기 전에 구식이 될 수 있습니다.

> 이 글에서는 https://github.com/Rust-for-Linux/linux/tree/for-next/rust 를 사용합니다.


![Building_Rust_For_Linux_In_M1_Mac](img/build_in_m1vm/build_rust_linux_in_m1.png)

## 소개
 현재 Apple Silicon Mac 시리즈는 일반 데스크톱급 워크스테이션 성능을 갖추면서 어디서든 구매할 수 있는 유일한 ARM 워크스테이션입니다.
물론 32GB 이상의 메모리와 최소 8개의 빅 코어를 갖춘 Apple Silicon이어야 합니다.

참고로 이 글은 https://github.com/Rust-for-Linux/linux/blob/rust/Documentation/rust/quick-start.rst 를 참고하여 작성되었습니다.

## VM 하이퍼바이저 소프트웨어 선택
 - UTM : 무료 / 오픈소스, QEMU 기반. 가끔 까다로움.
 - VM Fusion Tech Preview : 현재 무료 / 클로즈드소스, 무난함.
 - Parallels : 유료 / 클로즈드소스, 취향이 아님 (죄송합니다).

[Asahi Linux](https://asahilinux.org/)로 작업하는 옵션도 있습니다. 하지만 이 글에서는 네이티브 Asahi Linux 환경은 고려하지 않습니다.

제 경우에는 VM Fusion을 선택했습니다.

## 가상 머신 구성
- Debian 11 : https://cdimage.debian.org/debian-cd/current/arm64/iso-dvd/ <br>
    !! 정상 동작 확인 완료.

- Ubuntu : <br>
    `aarch64` Ubuntu apt 저장소에서 `clang`과 다른 gcc 빌드 도구의 버전 불일치로 인해 apt가 깨지는 이슈가 있었습니다.
    하지만 Ubuntu로 시도해볼 수는 있습니다.

- Arch Linux : https://gitlab.archlinux.org/tpowa/archboot/-/wikis/Archboot-Homepage#aarch64-architecture <br>
    아직 테스트하지 않았습니다. 다만 M1 Mac Mini에서 Asahi Linux로는 테스트했습니다.


### Debian / Ubuntu 패키지 요구사항

```sh
# Install build-requirements for kernel compile with LLVM.
apt install clang git llvm-dev libclang-dev build-essential \
bc kmod cpio flex libncurses5-dev libelf-dev libssl-dev \
dwarves bison lld curl
```
### Asahi Linux 패키지 요구사항

```sh
pacman -S base-devel cpio lld llvm llvm-libs bc libdwarf
```

### Rust 준비

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
***   Your version:     1.64.0
***   Expected version: 1.62.0
***
Rust is available!
```
위와 같은 결과가 나오면 준비 완료입니다.

## `menuconfig`로 Linux 소스코드 설정하기
```
make ARCH=arm64 defconfig
make menuconfig
```

## `GCC plugins` 비활성화
![Menuconfig Rust Support](img/build_in_m1vm/disable_gcc_plugins.png)
`General architecture-dependent options -> GCC plugins`
현재(6.1 rc*) 시점에서 `RUST_CONFIG` 설정을 위해 `GCC_PLUGINS` 설정을 비활성화해야 합니다.
반드시 비활성화하세요.

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


## 가상 머신에서 컴파일 및 설치

![Build time](img/build_in_m1vm/build_make_LLVM.png)

```sh
# -j4 for 4 core virtual machine, -j2 for 2 core, -j1 for single core.
make LLVM=1 -j4
```
위 명령어로 빌드합니다. 가상 머신에 할당된 코어 수를 고려하여 job 수를 설정해야 합니다. (`-j#`)

빌드 중에 일부 플래그에 대해 질문이 나올 수 있습니다. 저는 기본값을 선택했습니다.

시간이 꽤 걸립니다 (라즈베리 파이 4보다는 훨씬 낫습니다만), 제 환경(VM 4코어, 8GB)에서 13~14분 정도 소요되었습니다.

빌드 후 다음 명령어로 설치합니다.
```sh
# should be under the root permision.
make modules_install
make install
update-grub
```
![make install rust-kernel](img/build_in_m1vm/make_install.png)

완료되었습니다! <br>
재부팅 후 커널이 정상적으로 동작하는지 확인합니다.


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

### 간단한 컴파일 속도 비교
| 머신 / 환경 | 컴파일 시간 |
| --------------------- | -----------: |
| M1 Max 가상 머신 (4코어 8GB RAM, aarch64 debian11) | 16분 |
| M1 Asahi Linux (4P+4E 코어 16GB RAM MacMini, 6.1.0-rc6-asahi) | 11분 |
| AMD Ryzen 5950x 네이티브 (16코어 32스레드, 64GB, x86_64)  | 3분  |
| AMD Threadripper Pro 5975wx 네이티브 (32코어 64스레드, 256GB, x86_64)  | 2분  |
