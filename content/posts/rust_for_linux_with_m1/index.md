---
title: "[DRAFT] Rust For Linux Development Environment with AppleSilicon MacOS"
date: 2022-10-01
draft: false
category: "Linux"
tags: ["Rust", "Linux", "ARMv8A", "English_Article", "Draft"]
---

>> Currently Draft

> In October 1, Rust for linux is under the linux-next, not stable<br>
> Thus this article would be out-of-date before Linux 6.1 stable comes.

> This article play with https://github.com/Rust-for-Linux/linux/tree/for-next/rust

## Introduction
 Currently Apple Silicon mac series is only one ARM workstation that have powerful performance as normal desktop class workstation and can purchase anywhere.
Of course if you have 32GB or bigger memory and least 8 big cores of apple silicon.

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
    I didn't tested yet.


### Debian Requirements
```sh
apt install build-essential
```
-TBD-

## Check `RUST_AVAILABLE`
- TBD


## menuconfig



## compile and save