---
title: "ARMv8A Memory IP Review"
date: 2021-12-14
draft: false
categories: ["Linux"]
tags: ["ARMv8A", "Electronics", "Korean_Article", "Rust"]
---

## Introduction

ARMv8A, also commonly known as aarch64, is one of the widely used architectures that has largely succeeded ARMv7A. In this article, we will examine the ARMv8A memory system at the IP (Intellectual Property) block level.

![Simplifieid_Ip_Diagram](img/ARMv8A_IP.png)

While there are slight differences depending on the memory type used (DDR4, LPDDR4, DDR3, LPDDR3, DDR2) and the architecture version (ARM v8.1 or 8.2), the overall structure generally follows the form shown in the diagram above.

## Components
### CPU
Processes instructions.

### GIC
Generic Interrupt Controller; The GIC manages various nested interrupts. When an interrupt occurs, it backs up the PC/registers that were active on the CPU and directs execution to the corresponding interrupt vector.
### CCI / CCN
Cache Coherent Interconnect / Network

### DMC
Manages DRAM. DRAM is volatile memory that requires operations beyond simple Read/Write, such as Refresh and Calibration.
Additionally, it handles ECC and RAS management. In the Linux driver codebase, you can find the ECC and RAS management driver code under the `edac` directory.

### NIC
Used to connect various peripherals.

### MMU
- PA/VA (Physical/Virtual Address) translation
- DMA control


## Reference
- CCI-400 ; https://developer.arm.com/Processors/CoreLink%20CCI-400
- CCI-500 ; https://developer.arm.com/Processors/CoreLink%20CCI-500
-
- DMC-400 ; DDR3/DDR2 DMC ; https://developer.arm.com/documentation/ddi0466/f/introduction/about-the-dmc-400
- DMC-500 ; LPDDR4/LPDDR3 DMC ; https://developer.arm.com/documentation/100131/0000
