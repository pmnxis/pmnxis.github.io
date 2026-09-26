---
title: "Bringing Embassy to C-SKY MCUs That Rust Did Not Support"
date: 2026-09-26T00:00:00+09:00
draft: true
categories: ["Embedded"]
tags: ["Rust", "C-SKY", "Embassy", "LLVM", "QEMU", "English_Article"]
description: "Building a C-SKY bare-metal Rust target, LLD, QEMU, and Embassy stack and validating it on a WinnerMicro W806"
showHero: true
heroStyle: "background"
---

> 🌐 [한국어 아티클](/ko/posts/csky-rust-embassy/) | [日本語アーティクル](/ja/posts/csky-rust-embassy/)

> This article covers only the public C-SKY architecture, a generic QEMU
> environment, and validation on the commercially available HLK-W806-KIT.

Espressif's ESP32 series uses Xtensa LX and RISC-V, while GigaDevice's GD32
series uses Arm Cortex-M and RISC-V. Less well known are the C-SKY-based MCUs
found in some wireless and appliance products for the Chinese domestic market.

<style>
img[src$="rust-shigure-ui-ferris.webp"] {
  display: block;
  width: 60%;
  height: auto;
  margin: 1.25rem auto;
}
</style>

![Ferris examining a simple wireless or appliance product](rust-shigure-ui-ferris.webp)

<p style="text-align: center;"><em>Could a simple product like this contain an 8051 or C-SKY MCU?</em></p>

Rust already has the Tier 3 `csky-unknown-linux-gnuabiv2` target, and LLVM has a
C-SKY code generator. That path, however, builds Linux programs. An MCU
bare-metal environment including startup code, vector tables, interrupts,
linker scripts, and `core` was not available.

This article describes building a `csky-unknown-none-elfabiv2` environment,
producing firmware with Rust and LLVM/LLD without a GCC linker, and running the
Embassy executor, timers, and `embassy_sync::Channel` on QEMU's CK801, CK802,
CK803, CK804, and CK805 profiles and on a real WinnerMicro W806.

## Results first

As of September 2026, I have verified the following:

- Patched Rust/LLVM produces bare-metal Rust binaries for CK801, CK802, CK803,
  CK804, and CK805.
- Patched LLD links little-endian ELF32 for C-SKY, without using a GCC
  toolchain `ld` for the final link.
- Generic C-SKY QEMU runs workloads covering reset, memory initialization,
  IRQs, compiler builtins, and Embassy wake/timer/queue behavior.
- CK804 passes queue-capacity-eight saturation, repeated cancellation,
  simultaneous deadlines, IRQ priority, and 64-bit timebase wrap checks.
- Standard Embassy Channel ping/pong tasks run in QEMU and on the HLK-W806-KIT.
- A 1 ms TIM0 IRQ drives the Embassy time driver on W806, and the real board
  wakes from `wait32` instead of busy-spinning.

This is not yet a finished target upstreamed to rustc. The official rustup
`rust-lld` does not contain this C-SKY bare-metal work either. These results use
a project-local compiler and patch series.

## C-SKY and CK80x

C-SKY is a 32-bit RISC ISA. This work focuses on the ABIv2 CK80x MCU family.
CK801 and CK802 are small-MCU profiles; the available instructions and CPU
configuration change through CK803, CK804, and CK805. These numbers should not
be mapped one-to-one to Armv6-M or Armv7-M because the ISA, ABI, exception
entry, and interrupt-controller models differ.

For instruction-set, register, and exception-model details, I referred to the
[`C-SKY Architecture User Guide`](https://github.com/c-sky/csky-doc/blob/master/CSKY%20Architecture%20user_guide.pdf).

The following differences matter especially to an MCU runtime:

- Exception entry and return must follow the PSR/EPSR/EPC and CPU/compiler
  interrupt-frame rules. `ipush` and `ipop` are ISA instructions used in this
  context handling. The public W806 SDK's
  [`vectors.S`](https://github.com/IOsetting/wm-sdk-w806/blob/cb56742abb1b3eb4e0472213e43e0d6028dbb5ba/platform/arch/xt804/bsp/vectors.S#L55-L82)
  saves general registers and EPSR/EPC on trap entry, while its
  [`tspend_handler`](https://github.com/IOsetting/wm-sdk-w806/blob/cb56742abb1b3eb4e0472213e43e0d6028dbb5ba/platform/component/FreeRTOS/portable/xt804/cpu_task_sw.S#L119-L150)
  saves and restores task context including EPSR/EPC.
- The architecture's base exception vectors and a SoC's total vector count
  are separate concerns. A real MCU may expose 48 or 64 entries.
- The VBR (Vector Base Register) location and whether ROM vectors are copied
  into RAM are chip/runtime policies.
- CPU interrupt enable and the SoC interrupt controller's enable, pending,
  priority, and wake-enable states are separate.

Public boards include the W800 Arduino, W803 Pico, HLK-W800, and HLK-W806.
Their pinouts and flash layouts differ, but they are not each a separate Rust
ISA target. I chose the readily available W806 on the HLK-W806-KIT—an
XT804/CK804EF-family device—for hardware validation.

## Why more work was needed despite the LLVM backend

“LLVM has a C-SKY backend” does not mean “Rust bare-metal firmware can be
completed.” The latter requires every layer in this path:

{{<mermaid>}}
flowchart TB
    APP["1. Application / runtime<br/><br/>Cortex-M: Embassy + cortex-m + cortex-m-rt<br/>C-SKY: C-SKY-enabled Embassy + csky + csky-rt"]
    TARGET["2. Target / base libraries<br/><br/>Cortex-M: built-in rustc target + distributed core<br/>C-SKY: custom target JSON + build-std core / compiler_builtins"]
    TOOLS["3. Codegen / linking<br/><br/>Cortex-M: distributed rustc / LLVM Arm + rust-lld<br/>C-SKY: project-local rustc / patched LLVM + patched ld.lld"]

    APP --> TARGET --> TOOLS
{{< /mermaid >}}

1. **Application and runtime**  
   On Cortex-M, Embassy runs on [`cortex-m`](https://crates.io/crates/cortex-m)
   and [`cortex-m-rt`](https://crates.io/crates/cortex-m-rt). For C-SKY, the
   corresponding roles are filled by C-SKY-enabled Embassy and
   [`csky`](https://crates.io/crates/csky)/[`csky-rt`](https://crates.io/crates/csky-rt).
   In particular, the runtime must implement reset entry,
   memory initialization, the vector table, and interrupt entry/return for the
   architecture.

2. **Target and base libraries**  
   Common Cortex-M target information is built into rustc, and rustup ships
   `core` for those targets. Because a C-SKY bare-metal target is not yet an
   official distribution, a custom target JSON supplies CPU and ABI properties,
   and `build-std` builds `core` and `compiler_builtins` from source.

   References:

   - https://github.com/pmnxis/csky-rust-buildroot/blob/main/config/csky-unknown-none-elfabiv2-ck804.json
   - https://github.com/pmnxis/csky-rust-buildroot/blob/main/scripts/build-example.sh#L38-L43
   - https://github.com/pmnxis/csky-rust-buildroot/blob/main/docs/patches.md#role-of-the-installed-rust-components

3. **Code generation and linking**  
   rustc does not compile Rust through Clang; it uses LLVM libraries as its
   code-generation backend. Cortex-M can use the distributed rustc LLVM Arm
   backend and `rust-lld`. The current C-SKY setup creates objects with the
   patched LLVM C-SKY backend in a project-local rustc, then combines them into
   a bare-metal ELF with patched `ld.lld` and `link.x`. Clang is the C/C++ front
   end in the same LLVM ecosystem and does not participate in this Rust path.

This research uses two LLVM trees:

1. LLVM bundled with the Rust source handles C-SKY codegen for rustc.
2. A standalone LLVM tree builds `ld.lld`, `llvm-readelf`, `llvm-objdump`, and
   related tools.

The custom target JSON passes CPU, ISA feature, endianness, pointer-width, and
atomic capabilities to rustc. Since rustup has no `core` for the custom target,
it is built with `-Z build-std=core,compiler_builtins`. `no_std` means not using
`std`; it does not mean that `core` is unnecessary.

### LLVM/LLD additions

The applied order and patch files are in
[`patches/llvm`](https://github.com/pmnxis/csky-rust-buildroot/tree/main/patches/llvm).

- assembler `.ltorg`/`.pool` literal-pool directives
- a little-endian ELF32 C-SKY LLD backend and initial relocations
- range and alignment checks for PC-relative IMM16 relocations
- naturally aligned 8/16/32-bit atomic load/store codegen

The last item directly affects Embassy. A lack of compare-and-swap on CK80x is
not the same as being unable to perform ordinary atomic loads and stores. The
target declares `max-atomic-width = 32` and `atomic-cas = false`, while LLVM
generates the supported atomic loads/stores. This made it possible to use the
standard `embassy_sync::Channel` without adding a C-SKY-only queue to Embassy.

LLD dynamic linking, TLS/GOT/PLT, ELF attribute merging, compatible-CPU
`e_flags` merging, long-branch thunks, and big-endian regressions are not yet
complete. Static bare-metal MCU ELF was validated first.

## [`csky`](https://crates.io/crates/csky) and [`csky-rt`](https://crates.io/crates/csky-rt)

Like [`cortex-m`](https://crates.io/crates/cortex-m) and
[`cortex-m-rt`](https://crates.io/crates/cortex-m-rt) in the Arm ecosystem,
the implementation is split into two layers.

The [`csky`](https://crates.io/crates/csky) crate provides CPU-level functionality:

- architectural-register APIs for PSR, VBR, SP, EPSR, EPC, CPUID, and others
- interrupt masking and a critical-section backend
- compiler and hardware barriers
- low-level instructions such as `nop` and `wait`

[`csky-rt`](https://crates.io/crates/csky-rt) handles the path before the program
reaches main and interrupt entry:

- reset handler, stack, `.data` copying, and `.bss` initialization
- the `#[csky_rt::entry]` Rust entry macro
- 32/48/64-entry vector layouts and a default handler
- VBR policy for moving flash vectors into RAM
- an assembly wrapper that transfers control to a Rust IRQ handler

W806 clock, GPIO, UART, TIM, and VIC registers belong in a SoC HAL/BSP. The
generic runtime lets a user choose vector size and VBR policy but does not
embed a particular product's memory map.

### IRQ frames depend on CPU features

C-SKY `ipush` alone does not preserve every register required by a Rust call.
The wrapper additionally saves r15 and, on CK804E/EF with high registers,
r18–r31. Conversely, that same frame must not be used unconditionally on CPUs
that do not have those registers. Nested IRQ frames therefore need validation
for each CPU profile.

※ *The W806 is a CK804EF, but the current target disables LLVM's FPU instruction feature.*

## Embassy executor and idle

Embassy is a Rust embedded ecosystem that brings `async`/`await` to MCUs. It
fits the goal of dividing timer and protocol state machines into tasks with
explicit await boundaries.

The main change is confined to the architecture-specific part of
`embassy-executor`. `embassy-time`, `embassy-time-driver`,
`embassy-time-queue-utils`, and `embassy-sync` remain unchanged where possible.
The Channel issue was solved by fixing LLVM atomic support instead of placing a
temporary C-SKY implementation in Embassy.

Idle behavior turned out to be a deeper issue. After establishing correctness
with a spin loop, I applied `csky::asm::wait()`. Executing `wait32` inside a
critical section with interrupts masked prevents wakeup, while sleeping just
after enabling interrupts creates a check-then-sleep boundary race.

The current implementation uses this sequence:

1. Mask interrupts and recheck the executor work flag.
2. If there is no work, wait with adjacent `psrset ie` → `wait32`.
3. W806 uses a recurring 1 ms IRQ, so even a short boundary race advances on
   the next tick.

{{<mermaid>}}
flowchart LR
    CHECK["1. Mask interrupts<br/>recheck work flag"] --> WORK{"Work pending?"}
    WORK -- "yes" --> RUN["Run executor"]
    WORK -- "no" --> WAIT["2. psrset ie → wait32<br/>enable interrupts, then wait"]
    WAIT --> IRQ["3. Current or next 1 ms IRQ<br/>wake even after boundary race"]
    IRQ --> RUN
{{< /mermaid >}}

## Why QEMU came first

The QEMU patch order and the purpose of each patch are documented in
[`patches/qemu`](https://github.com/pmnxis/csky-rust-buildroot/tree/main/patches/qemu).

With hardware alone it is difficult to separate interrupt-frame, compiler
codegen, and peripheral-configuration faults. QEMU makes it possible to build
small ELFs and repeatedly inspect register, exception, and memory state.

After reviewing the early C-SKY QEMU 6.x code, I moved to the XUANTIE QEMU 9
branch and improved interrupt-controller pending masks, priorities, IRQ
lifecycle, and EPSR preservation.

The public buildroot QEMU is not a board model that duplicates a commercial
MCU. It validates the compiler/runtime/executor with a generic CPU, interrupt
controller, timer, UART, and semihosting. It contains no product peripheral,
ROM, or application protocol. Passing QEMU therefore does not automatically
prove a particular device's clock, IRQ, or peripheral compatibility.

### What was validated

- reset, stack, `.data`, `.bss`, and vector base
- critical sections, repeated IRQs, and callee-saved register canaries
- 256-iteration IRQ stress and `compiler_builtins`
- Embassy task wake, timer ordering, and cancel/re-arm
- completion of a ninth task after saturating a queue of capacity eight
- same-deadline alarms and differing interrupt priorities
- deadlines immediately before and after counter wrap and 64-bit Embassy `Instant`

The workload matrix ran on five profiles: CK801, CK802, CK803, CK804, and CK805.

LLVM code and QEMU results still disagree on 64-bit division/remainder that
uses CK802 E2 instructions. A temporary `-e2,+e1` workaround is not considered
completed CK802 E2 support.

※ *E1 and E2 are names used by the C-SKY ABIv2 toolchain for extension groups
added above the base ISA. E1 is the first group associated with CK801; E2 is
the next group associated with CK802. In LLVM, E2 includes E1. ***-e2,+e1***
keeps the CK802 CPU selection but disables E2-specific instruction generation,
leaving instructions through E1 enabled. It neither changes the CPU to CK801
nor changes ELF ABI v2 to v1.*

## Bringing up the HLK-W806-KIT

![The HLK-W806-KIT used for this validation](mcu-board-dc.webp)

_The WinnerMicro W806-based HLK-W806-KIT used for hardware validation._

The relevant W806 configuration is:

```text
CPU              XT804 / CK804EF family
application      0x0801_0400
vector RAM/VBR   0x2000_0000, 64 entries
UART0            115200 baud
Embassy tick     APB TIM0, IRQ 30, 1 ms recurring
idle             C-SKY wait32
```

When the timer initially failed, I checked CORET, VIC, priority, and PSR one by
one. I first proved that the TIM0 IRQ and ping/pong worked while busy-spinning;
after applying `wait32`, execution again stopped at the first wait.

The cause was that W806 VIC interrupt enable and wake enable are separate
registers. Enabling IRQ30 only in ISER delivers the IRQ while the CPU is
running, but cannot wake a CPU in `wait32`. After setting the IRQ30 bit
`0x4000_0000` in VIC `IWER0` (`0xe000_e140`), every tick woke the CPU and polled
the executor.

The hardware produced this output:

```text
HLK-W806 Embassy multi-task started
ping: received=0, sent=1, elapsed_ms=250
pong: received=1, sent=2, elapsed_ms=1000
ping: received=2, sent=3, elapsed_ms=250
pong: received=3, sent=4, elapsed_ms=1000
```

The example exchanges `DummyCounter { value: i32, time: Instant }` through a
standard Embassy Channel. Ping awaits 250 ms and pong awaits 1,000 ms. This
output is therefore evidence that all of the following layers worked together:

```text
TIM0 hardware IRQ
  → csky-rt interrupt entry/frame
  → Embassy time driver and timer queue
  → C-SKY executor
  → embassy_sync::Channel
  → ping/pong futures
```

## Reproducible `csky-rust-buildroot`

I packaged this work as `csky-rust-buildroot`. It is not Linux Buildroot; it is
a source buildroot and example workspace that uses Cargo `xtask` to reproduce
the compiler, emulator, and examples.

```sh
# Install dependencies for the host distribution.
cargo xtask deps

# Fetch the pinned Rust/LLVM, standalone LLVM, QEMU, and Embassy, then patch them.
cargo xtask fetch

# Build the local rustc, LLVM/LLD tools, and QEMU.
cargo xtask tools

# Run the basic timer example in QEMU.
cargo xtask run

# Run the standard Embassy Channel multi-task example.
EXAMPLE=embassy-multi-task RUN_SECONDS=15 cargo xtask run

# Decode QEMU transport defmt frames live.
DEFMT_LOG=trace RUN_SECONDS=15 cargo xtask run-defmt

# Flash the HLK-W806-KIT and follow its UART log.
./scripts/run-w806.sh multi-task ttyUSB0 --manual-reset
```

This is longer than `cargo run --example` because it does not merely build one
package. The project-local rustc, custom target, `build-std`, patched LLD, image
generation, and QEMU or flashing must be selected as one consistent set. Host
Cargo is the orchestrator, while `.local/rust/bin/rustc` performs codegen and
`.local/bin/ld.lld` performs the link.

This work was developed only on Linux hosts. Windows and macOS were not tested
separately.

## Remaining work

- Determine how CK801/CK802 CPU-profile selections and E1/E2 feature overrides
  affect the actual supported instruction set (insufficient information today)
- an atomic idle/wakeup rule applicable to one-shot IRQs
- IRQ context save/restore for CK804EF with the FPU enabled
- more LLD relocations, attributes, thunks, and endian coverage
- an upstream and rustup distribution path for rustc/LLVM/LLD
- clock, flash, IRQ, and peripheral validation on public C-SKY MCUs beyond W806

Success on QEMU and W806 must not be generalized to every C-SKY MCU. CPU
features, vector count, interrupt controller, wake registers, ROM contracts,
and memory maps can vary by SoC.

## Closing

The starting assumption was, “LLVM has a C-SKY backend, so perhaps one target
JSON is enough.” In practice, codegen, the linker, runtime, vector/IRQ ABI,
atomics, executor, emulator, and SoC wake mechanisms all had to be understood,
and substantial work was required to connect them into one continuous path.

It is now possible to create C-SKY bare-metal ELF with Rust and LLVM/LLD without
a GCC linker, regression-test several CPU profiles in QEMU, and run a
`wait32`-based Embassy executor, hardware timer, and standard Channel on a real
W806.

This foundation can test and analyze C-SKY binaries. Dedicated scratch
registers in the virtual QEMU environment can also inject external state and
inputs, enabling firmware behavior and features to be tested under emulation.
I expect this to be useful for CI/CD regression tests and C-SKY firmware
research. Validation on more commercially available C-SKY MCUs is the next
goal.

## Related material

- C-SKY Rust buildroot and examples: <https://github.com/pmnxis/csky-rust-buildroot>
- LLVM/LLD patch series: <https://github.com/pmnxis/csky-rust-buildroot/tree/main/patches/llvm>
- QEMU patch series: <https://github.com/pmnxis/csky-rust-buildroot/tree/main/patches/qemu>
- Embassy fork integration notes: <https://github.com/pmnxis/csky-rust-buildroot/tree/main/patches/embassy>
- [`csky` architecture crate](https://crates.io/crates/csky): [source](https://github.com/pmnxis/csky)
- [`csky-rt` runtime crate](https://crates.io/crates/csky-rt): [source](https://github.com/pmnxis/csky-rt)
- Experimental C-SKY Embassy branch: <https://github.com/pmnxis/embassy/tree/feature/csky-experimental>
- C-SKY architecture documentation: <https://github.com/c-sky/csky-doc>
- XUANTIE QEMU: <https://github.com/XUANTIE-RV/qemu>
- Embassy: <https://github.com/embassy-rs/embassy>
- WinnerMicro W800 documentation: <https://doc.winnermicro.net/w800/en/latest/>
