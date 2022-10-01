---
title: "ARMv8A Memory IP Review"
date: 2021-12-14
draft: false
category: "Linux"
tags: ["ARMv8A", "Electronics", "Korean_Article", "Rust"]
---

## Introduction

ARMv8A는 흔히 aarch64로도 불리며 현재 ARMv7A를 뒤로한채 널리 사용하는 아키텍처중 하나입니다. 본 글에서는 ARMv8A의 메모리 시스템을 IP단위로 보려합니다.

![Simplifieid_Ip_Diagram](img/ARMv8A_IP.png)

사용하는 메모리 (DDR4 , LPDDR4 , DDR3 , LPDDR3 , DDR2)나 사용하는 아키텍처(ARM v8.1 or 8.2)에 조금씩 다르나 대체적으로 위 사진과 같은 형태로 구성이 되어있습니다.

## Components
### CPU
Instruction을 처리합니다.

### GIC
Generic Interrupt Controller; GIC는 각종 Nested한 Interrupt를 관리하며, Interrupt발생시 CPU에서 동작중이던 PC/Register를 백업하고 해당하는 Inetrrupt Vector를 Execution하도록 합니다.
### CCI / CCN
Cache Coherent Interconnect / Netowrk

### DMC
DRAM을 관리합니다. DRAM은 휘발성 메모리로서 Read/Write 이외에 Refresh, Callibration 와 같은 작업이 필요합니다.
추가적으로 EEC, RAS에 대한 관리도 수행합니다. Linux드라이버에서는 `edac` 디렉터리에서 EEC, RAS에 대한 관리 드라이버 코드를 확인 할 수 있습니다.

### NIC
각종 Peripheral 을 연결하는데 사용합니다. 

### MMU
- PA/VA (Physical/Virtual Address) 변환
- DMA 컨트롤


## Reference
- CCI-400 ; https://developer.arm.com/Processors/CoreLink%20CCI-400
- CCI-500 ; https://developer.arm.com/Processors/CoreLink%20CCI-500
- 
- DMC-400 ; DDR3/DDR2 DMC ; https://developer.arm.com/documentation/ddi0466/f/introduction/about-the-dmc-400
- DMC-500 ; LPDDR4/LPDDR3 DMC ; https://developer.arm.com/documentation/100131/0000
