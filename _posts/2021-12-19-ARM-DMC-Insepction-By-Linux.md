---
layout: default
title: Walk from linux for ARM DMC (Dynamic Memory Controller) control
subtitle: "OS and memory controller become shy when they meet."
type: "TIL"
createDate: "2021-12-19"
til: true
text: true
author: Jinwoo Park
post-header: false
header-img: ""
order: 2
---

### Preface
Let's walk through ARM DMC (Dynamic Memory Controller) from linux aspect.
![Lambda_With_Turtle](/assets/img/lambda_with_turtle.jpg)

Most of case, If you are embedded or software engineer (Except silicon, FPGA, hardware engineer), don't care memory controller. Some expert engineer use `barrier` for communicate with multi-core or NUMA system, external bus or memory space, but barrier is controlled by CCI/CCN(Cache Coherent Interconnect / Netowrk) not DMC. However some case, OS (in this article linux) communicate with DMC with some specific driver or method.

### EDAC
- WIP

### Get Timing by some reason
- WIP

### Unfamiliar RAM
- WIP

## How ARM DMC IP working for computer architecture.
- WIP