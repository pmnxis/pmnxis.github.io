---
title: "Rustが未対応だったC-SKY MCUでEmbassyを動かすまで"
date: 2026-09-26T00:00:00+09:00
draft: false
categories: ["Embedded"]
tags: ["Rust", "C-SKY", "Embassy", "LLVM", "QEMU", "Japanese_Article"]
description: "C-SKY bare-metal Rustターゲット、LLD、QEMU、Embassyを構築し、WinnerMicro W806実機で検証した記録"
summary: "C-SKY MCU向けのbare-metal RustおよびEmbassy環境を構築し、検証した過程を紹介する。"
showHero: true
heroStyle: "background"
---

> 🌐 [한국어 아티클](/ko/posts/csky-rust-embassy/) | [English Article](/en/posts/csky-rust-embassy/)

> 本記事では、公開されているC-SKYアーキテクチャ、汎用QEMU環境、そして
> 市販のHLK-W806-KITで検証した内容だけを扱う。

EspressifのESP32シリーズはXtensa LXとRISC-Vを、GigaDeviceのGD32シリーズは
Arm Cortex-MとRISC-Vを使用している。それらほど知られてはいないが、中国国内向けの
無線・家電用MCUにはC-SKY ISAを採用した製品もある。

<style>
img[src$="rust-shigure-ui-ferris.webp"] {
  display: block;
  width: 60%;
  height: auto;
  margin: 1.25rem auto;
}
</style>

![簡単な無線・家電製品を調べるFerris](rust-shigure-ui-ferris.webp)

<p style="text-align: center;"><em>このような簡単な製品なら、8051やC-SKYが入っているのではないだろうか？</em></p>

RustにはすでにTier 3の`csky-unknown-linux-gnuabiv2`ターゲットがあり、LLVMにも
C-SKYコードジェネレータがある。しかし、これはLinuxプログラムを作るための経路だ。
スタートアップコード、ベクタテーブル、割り込み、リンカスクリプト、`core`を含む
MCU bare-metal環境は用意されていなかった。

本記事では、`csky-unknown-none-elfabiv2`環境を構築し、GCCリンカを使わずに
RustとLLVM/LLDでファームウェアをビルドし、QEMUのCK801・CK802・CK803・CK804・
CK805プロファイルと実機のWinnerMicro W806でEmbassy executor、timer、
`embassy_sync::Channel`を動かすまでの過程をまとめる。

## まず結果から

2026年9月時点で確認できた範囲は次のとおりだ。

- パッチを適用したRust/LLVMで、CK801、CK802、CK803、CK804、CK805向けの
  bare-metal Rustバイナリを生成できることを確認した。
- パッチを適用したLLDでC-SKY little-endian ELF32をリンクし、最終リンクに
  GCC toolchainの`ld`を使用しないことを確認した。
- 汎用C-SKY QEMUで、reset、メモリ初期化、IRQ、compiler builtins、
  Embassyのwake/timer/queueを含むworkloadを実行できることを確認した。
- CK804でqueue容量8の飽和、繰り返しcancel、同時deadline、IRQ priority、
  64-bit timebase wrapを確認した。
- 標準Embassy Channelのping/pong taskがQEMUとHLK-W806-KITで動作した。
- W806の1 ms TIM0 IRQでEmbassy time driverを駆動し、CPUがbusy-spinせず
  `wait32`から起床することを実機で確認した。

ただし、まだrustcにupstreamされた完成済みターゲットではない。公式rustupの
`rust-lld`にも、このC-SKY bare-metal対応は含まれていない。以下の結果は
プロジェクトローカルのcompilerとpatch seriesを使用した実験結果である。

## C-SKYとCK80x

C-SKYは32-bit RISC ISAである。本作業はABIv2のCK80x MCUファミリを中心に進めた。
CK801/CK802は小規模MCU向けのプロファイルであり、CK803、CK804、CK805へ進むに
つれて命令とCPU構成が変わる。ただし、これらの番号をArmv6-MやArmv7-Mへ一対一で
対応させてはいけない。ISA、ABI、exception entry、interrupt controllerのモデルが
異なるためだ。

命令セット、register、exception modelの詳細については、
[`C-SKY Architecture User Guide`](https://github.com/c-sky/csky-doc/blob/master/CSKY%20Architecture%20user_guide.pdf)
を参照した。

MCU runtimeの観点では、特に次の違いが重要だった。

- 例外への進入と復帰では、PSR/EPSR/EPCおよびCPU/compilerのinterrupt-frame規則に
  従う必要がある。`ipush`/`ipop`は、このcontext処理に使われるISA命令だ。公開W806
  SDKの[`vectors.S`](https://github.com/IOsetting/wm-sdk-w806/blob/cb56742abb1b3eb4e0472213e43e0d6028dbb5ba/platform/arch/xt804/bsp/vectors.S#L55-L82)
  はtrap進入時に汎用registerとEPSR/EPCを保存し、
  [`tspend_handler`](https://github.com/IOsetting/wm-sdk-w806/blob/cb56742abb1b3eb4e0472213e43e0d6028dbb5ba/platform/component/FreeRTOS/portable/xt804/cpu_task_sw.S#L119-L150)
  はtask contextでEPSR/EPCを含む状態を保存・復元する。
- アーキテクチャの基本exception vector数とSoCが提供する全vector数は区別する必要が
  ある。実際のMCUは48または64 entryを持つ場合がある。
- VBR（Vector Base Register）の位置と、ROM vectorをRAMへ移すかどうかは
  chip/runtimeの方針である。
- CPUのinterrupt enableと、SoC interrupt controllerのenable、pending、priority、
  wake enableは別々の状態である。

公開ボードにはW800 Arduino、W803 Pico、HLK-W800、HLK-W806などがある。ボードごとに
pinとflash配置は異なるが、それぞれが別のRust ISA targetというわけではない。
ここでは比較的入手しやすいHLK-W806-KITのW806、つまりXT804/CK804EF系を実機検証用に
選択した。

## LLVM backendがあるのに、なぜ追加作業が必要だったのか

「LLVMにC-SKY backendがある」と「Rust bare-metalファームウェアを完成できる」は
同じ意味ではない。後者には、次の経路がすべて必要になる。

{{<mermaid>}}
flowchart TB
    APP["1. アプリケーション / runtime<br/><br/>Cortex-M: Embassy + cortex-m + cortex-m-rt<br/>C-SKY: C-SKY対応Embassy + csky + csky-rt"]
    TARGET["2. target / 基本ライブラリ<br/><br/>Cortex-M: rustc内蔵target + 配布済みcore<br/>C-SKY: custom target JSON + build-std core / compiler_builtins"]
    TOOLS["3. codegen / link<br/><br/>Cortex-M: 配布rustc / LLVM Arm + rust-lld<br/>C-SKY: project-local rustc / patched LLVM + patched ld.lld"]

    APP --> TARGET --> TOOLS
{{< /mermaid >}}

1. **アプリケーションとruntime**  
   Cortex-MではEmbassyが[`cortex-m`](https://crates.io/crates/cortex-m)と
   [`cortex-m-rt`](https://crates.io/crates/cortex-m-rt)の上で動作する。C-SKYでは同じ役割を
   C-SKY対応Embassyと[`csky`](https://crates.io/crates/csky)/[`csky-rt`](https://crates.io/crates/csky-rt)が担う。
   runtimeはreset entry、メモリ初期化、
   vector table、割り込みの進入・復帰をアーキテクチャに合わせて提供する必要がある。

2. **targetと基本ライブラリ**  
   一般的なCortex-M target情報はrustcに内蔵され、`core`はrustupで配布される。
   C-SKY bare-metal targetはまだ公式配布物ではないため、custom target JSONでCPUや
   ABIなどを渡し、`build-std`で`core`と`compiler_builtins`をソースからビルドする。

   参考リンク：

   - https://github.com/pmnxis/csky-rust-buildroot/blob/main/config/csky-unknown-none-elfabiv2-ck804.json
   - https://github.com/pmnxis/csky-rust-buildroot/blob/main/scripts/build-example.sh#L38-L43
   - https://github.com/pmnxis/csky-rust-buildroot/blob/main/docs/patches.md#role-of-the-installed-rust-components

3. **codegenとlink**  
   rustcはClangでRustをコンパイルするのではなく、LLVMライブラリをcode-generation
   backendとして使う。Cortex-Mでは配布rustcのLLVM Arm backendと`rust-lld`を使える。
   現在のC-SKY構成では、プロジェクトローカルrustcのパッチ済みLLVM C-SKY backendで
   objectを生成し、パッチ済み`ld.lld`と`link.x`でbare-metal ELFへまとめる。Clangは
   同じLLVMエコシステムのC/C++ front endであり、このRustビルド経路には参加しない。

この研究では2つのLLVM treeを使用する。

1. Rust sourceに含まれるLLVMはrustcのC-SKY codegenを担当する。
2. standalone LLVM treeでは`ld.lld`、`llvm-readelf`、`llvm-objdump`などをビルドする。

custom target JSONはCPU、ISA feature、endian、pointer width、atomic capabilityを
rustcへ渡す。rustupはcustom target向け`core`を提供しないため、
`-Z build-std=core,compiler_builtins`で一緒にビルドする。`no_std`は`std`を使わない
という意味であり、`core`も不要という意味ではない。

### LLVM/LLDで補強した部分

実際の適用順序とpatch fileは
[`patches/llvm`](https://github.com/pmnxis/csky-rust-buildroot/tree/main/patches/llvm)で確認できる。

- assemblerの`.ltorg`/`.pool` literal-pool directive
- little-endian ELF32 C-SKY LLD backendと初期relocation
- PC-relative IMM16 relocationの範囲・alignment検査
- 自然alignmentの8/16/32-bit atomic load/store codegen

最後の項目はEmbassyへ直接つながる。CK80xにcompare-and-swapがないことと、通常の
atomic load/storeまで実行できないことは別問題だ。targetは`max-atomic-width = 32`、
`atomic-cas = false`で能力を表現し、LLVMは可能なatomic load/storeを生成する。
その結果、EmbassyへC-SKY専用queueを追加せず標準`embassy_sync::Channel`を使えた。

LLDのdynamic linking、TLS/GOT/PLT、ELF attributesのmerge、互換CPU間の`e_flags`
merge、long-branch thunk、big-endian regressionはまだ完成範囲ではない。まずMCUの
static bare-metal ELFを検証した。

## [`csky`](https://crates.io/crates/csky)と[`csky-rt`](https://crates.io/crates/csky-rt)

Armエコシステムの[`cortex-m`](https://crates.io/crates/cortex-m)と
[`cortex-m-rt`](https://crates.io/crates/cortex-m-rt)と同様に、2つの層へ分けた。

[`csky`](https://crates.io/crates/csky) crateはCPU自体の機能を提供する。

- PSR、VBR、SP、EPSR、EPC、CPUIDなどのarchitectural register API
- interrupt maskとcritical-section backend
- compiler barrierとhardware barrier
- `nop`、`wait`などの低レベル命令

[`csky-rt`](https://crates.io/crates/csky-rt)は、プログラムがmainへ到達する前の
処理とinterrupt entryを担当する。

- reset handler、stack、`.data` copy、`.bss`初期化
- `#[csky_rt::entry]` Rust entry macro
- 32/48/64-entry vector layoutとdefault handler
- flash vectorをRAMへ移すVBR policy
- Rust IRQ handlerへ移行するassembly wrapper

W806のclock、GPIO、UART、TIM、VIC registerはSoC HAL/BSPの責任である。汎用runtimeは
vector sizeとVBR policyを選択できるようにするが、特定製品のmemory mapは内蔵しない。

### IRQ frameはCPU featureによって異なる

C-SKYの`ipush`だけでは、Rust呼び出しに必要なすべてのregisterを保存できない。
wrapperはr15を追加で保存し、high registerを持つCK804E/EFではr18–r31も保存する。
反対に、それらのregisterを持たないCPUで同じframeを無条件に使ってはいけない。
したがってnested IRQ frameはCPU profileごとに分けて検証する必要がある。

※ *W806はCK804EFだが、現在のtargetではLLVMのFPU instruction featureを無効にしている。*

## Embassy executorとidle

EmbassyはMCUで`async`/`await`を使えるようにするRust embeddedエコシステムだ。
timerやprotocol state machineを複数taskへ分け、await境界を明確に表現する目的に適する。

主な変更は`embassy-executor`のarchitecture-specific部分だけに置いた。
`embassy-time`、`embassy-time-driver`、`embassy-time-queue-utils`、`embassy-sync`は
可能な限りそのまま使った。Channelの問題もEmbassyへ一時実装を追加せず、LLVMの
atomic対応を修正して解決した。

idle動作は予想以上に深い問題だった。まずspin loopで正しさを確認してから
`csky::asm::wait()`を適用した。interruptをmaskしたcritical section内で`wait32`を
実行すると起床できず、interrupt enable直後にsleepする境界にはcheck-then-sleep
raceも存在する。

現在の実装は次の順序を使用する。

1. interruptをmaskし、executorのwork flagを再確認する。
2. workがなければ、隣接した`psrset ie` → `wait32`で待機する。
3. W806は1 ms recurring IRQを使うため、短い境界raceに入っても次のtickで進む。

{{<mermaid>}}
flowchart LR
    CHECK["1. interruptをmask<br/>work flagを再確認"] --> WORK{"workあり?"}
    WORK -- "あり" --> RUN["executorを実行"]
    WORK -- "なし" --> WAIT["2. psrset ie → wait32<br/>interrupt enable後に待機"]
    WAIT --> IRQ["3. 今回または次の1 ms IRQ<br/>境界race後もwake"]
    IRQ --> RUN
{{< /mermaid >}}

## QEMUを先に使用した理由

QEMU patchの適用順序と役割は
[`patches/qemu`](https://github.com/pmnxis/csky-rust-buildroot/tree/main/patches/qemu)にまとめている。

実機だけではinterrupt frame、compiler codegen、peripheral設定の問題を分離しにくい。
QEMUなら小さなELFを作り、register、exception、memory状態を繰り返し検証できる。

初期のC-SKY QEMU 6.xを調べた後、XUANTIE QEMU 9 branchへ移行し、interrupt
controllerのpending mask、priority、IRQ lifecycle、EPSR保存を補強した。

公開buildrootのQEMUは特定の市販MCUを複製したboard modelではない。汎用CPU、
interrupt、timer、UART、semihostingでcompiler/runtime/executorを検証する。
特定製品のperipheral、ROM、application protocolは含まない。そのためQEMU通過が
特定実機のclock・IRQ・peripheral互換性を自動的に証明するわけではない。

### 検証した項目

- reset、stack、`.data`、`.bss`、vector base
- critical section、繰り返しIRQ、callee-saved register canary
- 256回のIRQ stressと`compiler_builtins`
- Embassy task wake、timer ordering、cancel/re-arm
- 容量8のqueueを飽和させた後の9番目task完了
- 同一deadlineのalarmと異なるinterrupt priority
- counter wrap直前・直後のdeadlineと64-bit Embassy `Instant`

CK801、CK802、CK803、CK804、CK805の5プロファイルでworkload matrixを実行した。

CK802のE2命令を使う64-bit division/remainderでは、LLVM codeとQEMUの結果が一致しない
問題が残っている。一時的な`-e2,+e1`回避策をCK802 E2対応完了とは見なしていない。

※ *E1とE2は、C-SKY ABIv2 toolchainで基本ISAの上に追加される拡張命令群を区別する
名称である。E1はCK801に対応する最初の拡張命令群、E2はCK802に対応する次の拡張命令群
であり、LLVMではE2がE1を含む。***-e2,+e1***はCK802 CPUの選択を維持したままE2専用
命令生成を無効にし、E1までの命令だけを使うLLVM target-feature overrideである。
CPUをCK801へ変更したり、ELF ABI v2をv1へ変更したりするoptionではない。*

## HLK-W806-KIT実機bring-up

![今回の検証に使用したHLK-W806-KIT](mcu-board-dc.webp)

_本記事の実機検証に使用したWinnerMicro W806ベースのHLK-W806-KIT。_

W806の主要構成は次のとおりだ。

```text
CPU              XT804 / CK804EF family
application      0x0801_0400
vector RAM/VBR   0x2000_0000, 64 entries
UART0            115200 baud
Embassy tick     APB TIM0, IRQ 30, 1 ms recurring
idle             C-SKY wait32
```

最初はtimerが動かなかったため、CORET、VIC、priority、PSRを一つずつ確認した。
busy-spinでTIM0 IRQとping/pongが動くことを先に証明し、その後`wait32`を適用すると、
再び最初の待機で停止した。

原因はW806 VICのinterrupt enableとwake enableが別registerであることだった。
IRQ30をISERだけで有効にするとCPU実行中にはIRQが届くが、`wait32`中のCPUを起こせない。
VIC `IWER0`（`0xe000_e140`）にIRQ30 bit `0x4000_0000`を設定すると、各tickで起床し
executorをpollするようになった。

実機で確認した出力は次のとおりだ。

```text
HLK-W806 Embassy multi-task started
ping: received=0, sent=1, elapsed_ms=250
pong: received=1, sent=2, elapsed_ms=1000
ping: received=2, sent=3, elapsed_ms=250
pong: received=3, sent=4, elapsed_ms=1000
```

この例では`DummyCounter { value: i32, time: Instant }`を標準Embassy Channelで受け渡す。
pingは250 ms、pongは1,000 msをawaitする。この出力は次の各層が一緒に動作した根拠だ。

```text
TIM0 hardware IRQ
  → csky-rt interrupt entry/frame
  → Embassy time driver and timer queue
  → C-SKY executor
  → embassy_sync::Channel
  → ping/pong futures
```

## 再現可能な`csky-rust-buildroot`

この作業を`csky-rust-buildroot`としてまとめた。Linux Buildrootそのものではなく、
Cargo `xtask`でcompiler、emulator、examplesを再現するsource buildroot兼example
workspaceである。

```sh
# distributionごとのdependencyをインストールする。
cargo xtask deps

# pinしたRust/LLVM、standalone LLVM、QEMU、Embassyを取得してpatchする。
cargo xtask fetch

# ローカルrustc、LLVM/LLD tool、QEMUをビルドする。
cargo xtask tools

# 基本timer exampleをQEMUで実行する。
cargo xtask run

# 標準Embassy Channel multi-task exampleを実行する。
EXAMPLE=embassy-multi-task RUN_SECONDS=15 cargo xtask run

# QEMU transportのdefmt frameをリアルタイムでdecodeする。
DEFMT_LOG=trace RUN_SECONDS=15 cargo xtask run-defmt

# HLK-W806-KITへflashし、UART logを表示する。
./scripts/run-w806.sh multi-task ttyUSB0 --manual-reset
```

`cargo run --example`より長いのは、単一packageだけをビルドする作業ではないためだ。
project-local rustc、custom target、`build-std`、patched LLD、image generation、QEMU
またはflashingを同じ組み合わせで選ぶ必要がある。host Cargoはorchestratorだが、実際の
codegenは`.local/rust/bin/rustc`、linkは`.local/bin/ld.lld`が担当する。

本作業はLinux hostでのみ開発した。WindowsとmacOSでは個別にテストしていない。

## 今後の課題

- CK801/CK802などのCPU profile選択とE1/E2 feature overrideの組み合わせが、実際の
  対応命令範囲へ与える影響の確認（現時点では情報不足）
- one-shot IRQにも適用可能なatomic idle/wakeup規則
- FPUを有効にしたCK804EFのIRQ context save/restore
- LLD relocation、attributes、thunk、endian対応の拡張
- rustc/LLVM/LLDのupstream化とrustup配布経路
- W806以外の公開C-SKY MCUにおけるclock、flash、IRQ、peripheral検証

QEMUとW806での成功を、すべてのC-SKY MCUの成功へ一般化してはいけない。CPU feature、
vector数、interrupt controller、wake register、ROM contract、memory mapはSoCごとに
異なる可能性がある。

## おわりに

出発点は「LLVMにC-SKY backendがあるなら、target JSON一つで済むのではないか」という
考えだった。実際にはcodegen、linker、runtime、vector/IRQ ABI、atomics、executor、
emulator、SoC wake mechanismそれぞれへの理解が必要であり、それらを一つの連続した
経路へ接続するために多くの作業が必要だった。

現在はGCCリンカを使わず、RustとLLVM/LLDでC-SKY bare-metal ELFを生成し、QEMUの複数
CPU profileでregression testを行い、実機W806で`wait32`ベースのEmbassy executor、
hardware timer、標準Channelを実行できる。

この基盤によりC-SKYバイナリのテストと解析が可能になり、QEMU仮想環境へ専用のscratch
registerを構成して外部状態や入力を注入することで、ファームウェアコードの動作と機能を
emulation上で検証することもできる。今後のCI/CD regression testやC-SKYファームウェア
研究にも活用できると期待している。さらに多くの市販C-SKY MCUで検証することが次の目標だ。

## 関連資料

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
