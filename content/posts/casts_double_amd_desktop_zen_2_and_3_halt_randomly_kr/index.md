---
title: "ZEN 2/3 의 QC/설계결함으로 인한 간혈적 리셋/멈춤 의혹 제기"
date: 2023-04-08T21:37:24+09:00
draft: false
category: ["CPU"]
tags: ["AMD", "Ryzen", "Cache", "Silicon Bug", "Korean_Article"]
---

이제 어느정도 ZEN4가 팔린지도 오래되었으며 ZEN3도 stepping? 내부적인 revision이 몇번 바뀌어 개선이 된 것 같아 적당한 시기다 싶어서 이 글을 작성합니다.
이 글은 다른 커뮤니티에도 올렸고 어떻게 보면 개발과 거리가 있으나 CPU 내부를 다루는 관계로 개발블로그에도 옮깁니다.

## 전력사용량이 크게 달라지는 경우 Instruction L1, L2 cache 가 장애가 발생함

![image](img/casts_double_amd_desktop_zen_2_and_3_halt_randomly_kr/how_many_report_in_korea.png)

_퀘이사존에 WHEA 만 검색해도 수많은 글을 볼 수 가 있습니다._

과거에 수많은 사람들이 AMD Zen2/ Zen3를 쓰면서 이러한 문제점들은 보고했습니다.

- 갑자기 WHEA 18에러가 Windows상의 Event Logger에 뜬다.
- 갑자기 꺼진다. [Halt] (로그 찾을 수 없음)
- 갑자기 리셋된다. [Reset] (로그 찾을 수 없음)
- 아무것도 안하고있는데 위와 같은 현상이 일어난다.
- C-State를 Disable하니 이러한 이슈가 덜 생긴다.
- C-State를 끄세요

저 또한 3950x를 과거에 쓰면서 이러한 이슈를 겪었으며 이로 인해서 매우 많은 시간을 허비했으며 결국에는 3950x를 5950x(최근의 Stepping Revision) 그리고 그 중간에 임시로 쓰게되는 5900x를 구매하면서 

컴퓨터가 2대가 되는 자가증식까지 하는 상황까지 겪었습니다. 그리고 이과정에 메인보드는 3번 구매했습니다.

여기 까지 글을 읽으시면 몇가지 키워드가 보이실겁니다.

| 키워드1 | 키워드2 | 키워드3 |
| :---: | :---: | :---: |
| Instruction L1, L2 cache |  C-State | Reset/Halt |

## C-State란
C-State부터 먼저 설명하겠습니다.
출처 : https://www.dell.com/support/kbdoc/ko-kr/000060621/c-state%EB%8A%94-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80

C-State 는 CPU소모전력을 줄일 수 있는 기능으로, 사용을 덜하는 경우 최대한으로 줄입니다.

실제로 AMD은 Intel이든 C-State를 켠상태에서 PC가 Idle시 사용되는 전력사용량은 UPS로 확인시 매우 큽니다. 10W정도에서 20W정도 차이가 납니다. 이정도 차이면  이번에 새로나온 Intel Alderlake-N100 MiniPC를 충분히 구동할 수 있는 전력이죠. 또한 이 기능은 1990년대에 나온 기능으로 사실상 그냥 지원을 온전히 보장 해주는게 맞는 좋은 기능입니다.

## Instruction L1, L2 cache 란

컴퓨터공학과 학부에서 배우는 "컴퓨터 구조론" 과목에서 배우는 메모리 모델부터 설명을 해보겠습니다.

![image](img/casts_double_amd_desktop_zen_2_and_3_halt_randomly_kr/MemoryHierarchy.png)

[출처 : https://diveintosystems.org/book/C11-MemHierarchy/mem_hierarchy.html ]

근대의 메모리 시스템은 이와같이 피라미드 형태로 되어있습니다.

가장 꼭대기에 있는 register부터 cache을 거쳐 Flash Disk 그리고 그아래까지 접근시간과 대신 담고 있는 양 (용량)의 차이가 비례해서 올라갑니다.

CPU입장에서 register에 접근하는데는 가장 짧은 시간이 걸리는 대신 register의 면적당 가격은 매우 비싸며

뒤로 갈수록 시간은 좀 걸리는 대신 가격은 싸집니다.

이 그림에서는 register/cache/Main Memory 사이의 용량이 나와있지는 않지만 사용하는 시스템/목적에 따라 사용될 용량의 분배는 매우 중요할것이며,
"어떻게 나뉘냐"
를 잘 배분 하는 것이 중요합니다. 여기서 사용자가 마음대로 할 수 있는것이라면 디램을 많이 때려박아주는게 될 수 있겠습니다.

그러면 Instruction L1, L2 cache 은 무엇인가 하면

CPU는 데이터를 처리를 빨리 접근/읽기 위해서도 cache가 필요하지만

cache는 CPU가 명령어 소화를 빨리 하기위해서도 cache가 필요합니다.

데이터를 읽는 목적이나 명령어를 처리하는 과정에서 cache에 데이터가 없다면 DRAM(main memory)에서 가져오는 동안 사실상 일을 멈추고 있는 상태가 됩니다. (파이프라이닝 개념은 여기서 고려안하겠습니다.)

그리고 뒤에있는 1/2는 1은 CPU각 코어에 가장 빨리 접근할수 있는 공간이며 2는 1보다는 약간 멀리떨어진 계층입니다. (그래도 DRAM보다는 가깝습니다.)

다시 Zen2/3로 돌아와서 설명하자면 Zen2/3에서 나온 CPU는 16코어 짜리 제품으로서 가장 코어가 많습니다. 그리고 멀티다이 구조를 채택하여 8코어를 두개씩 패키징한 제품입니다.

그래서 실제로는 메모리 모델이 위에잇는 세모 피라미드 모양이 되지않고 피라미드위에 피라미드 2개가 있고 그 2개의 피라미드 위에 피라미드가 8개씩 추가로 있는 구조가 나옵니다.

## 위에서 제기한 증상에 대한 자세한 보고
조립 PC를 사용하는 대다수의 사용자들은 Windows를 사용하고 있으시며, 이 글에서 언급한 문제를 겪으셨다면

Windows Event Logger에서 WHEA18?이란 에러가 뜨거나 아예 에러가 뜨지 않은채 꺼지시(reset)거나 멈추셨(Halt)겁니다. Reset이냐 Halt냐는 아마 메인보드에 다를겁니다. 이는 제가 Asrock/Gigabyte/Asus, B550/X570을 3개다 써보면서 각기 다른식으로 에러가 나오는 것을 확인했습니다.

이 문제를 정확히 분석하려면 우리는 Windows를 들여다볼게 아니라 Linux 커뮤니티에서 해답을 찾을 수 잇습니다.

- [**Correctable MCE errors logged for CPU0/CPU12 L1 instruction cache with AMD Ryzen 9 3900X 12-Core Processor**] https://bugzilla.redhat.com/show_bug.cgi?id=1830404
- [**Random freezes and reboots AMD Ryzen**]  https://bugzilla.kernel.org/show_bug.cgi?id=210261

(두개의 링크 말고도 수많은 보고가 있으나 일단 2개만 게시합니다.)

글자체가 매우 기니 요약만 한다면

`랜덤한 시간에 아래와 같은 에러가 뜨며 멈춘다. 그리고 CPU를 바꾸니 그냥 해결됬다`

`C-State를 키면 문제가 발생하지않거나 덜 발생한다.`

```
May 01 15:06:59 kernel: mce: [Hardware Error]: Machine check events logged
May 01 15:06:59 kernel: [Hardware Error]: Corrected error, no action required.
May 01 15:06:59 kernel: [Hardware Error]: CPU:12 (17:71:0) MC1_STATUS[Over|CE|MiscV|AddrV|-|-|SyndV|-|-|-]: 0xdc20000000030151
May 01 15:06:59 kernel: [Hardware Error]: Error Addr: 0x000000076da32ae0
May 01 15:06:59 kernel: [Hardware Error]: IPID: 0x000100b000000000, Syndrome: 0x000000001a030507
May 01 15:06:59 kernel: [Hardware Error]: Instruction Fetch Unit Ext. Error Code: 3, IC Data Array Parity Error.
May 01 15:06:59 kernel: [Hardware Error]: cache level: L1, tx: INSN, mem-tx: IRD
May 01 15:06:59 kernel: mce: [Hardware Error]: Machine check events logged
May 01 15:06:59 kernel: [Hardware Error]: Corrected error, no action required.
May 01 15:06:59 kernel: [Hardware Error]: CPU:0 (17:71:0) MC1_STATUS[Over|CE|MiscV|AddrV|-|-|SyndV|-|-|-]: 0xdc20000000030151
May 01 15:06:59 kernel: [Hardware Error]: Error Addr: 0x0000000fbedc2ae0
May 01 15:06:59 kernel: [Hardware Error]: IPID: 0x000100b000000000, Syndrome: 0x000000001a030507
May 01 15:06:59 kernel: [Hardware Error]: Instruction Fetch Unit Ext. Error Code: 3, IC Data Array Parity Error.
May 01 15:06:59 kernel: [Hardware Error]: cache level: L1, tx: INSN, mem-tx: IRD
```

## 에러에 대한 원인 추측/분석
특정 상황이 매우 드물게 발생하는데 (CPU가 3Ghz 이상이니 1초에 3000000000사이클 일것이며 사이클 관점에서 봤을때 30분에서 24시간중 한번 발생한다하면 매우 드물다는 입장이지, 사용자 입장에서 잦거나 드물다는 표현은 아닙니다.)

어느 위에 언급한 C-State에서 힌트를 찾았는데요

- C-State가 켜져있다면 매우 짧은 시간에 CPU core에 인가되는 전력이 __매우 작다가__ 매우 커질것이며
- C-State가 꺼져있다면 매우 짧은 시간에 CPU core에 인가되는 전력이 __약간 적었다가__ 매우 커질것입니다.

그럼 원인은 짧은 시간내에 CPU Core에 인가되는 전력차에 따라 CPU Instruction $1/$2 에 영향이 가므로

CPU가 명령어를 수행하다가 $ (캐시)에서 읽어올 명령어 자체가  coruppted(잘못됨)되었으니 CPU는 명령어를 수행할 수 없습니다. 더이상 나아갈 수 도 없으며 사실 이경우 코어덤프 (메모리 덤프)를 남기는 것 조차 기적일 것 입니다. [대충 메모리 디버깅에는 두가지 방법이 있는데요 SW로 전체 메모리를 백업하는 것과 외제차 가격보다 비싼 하드웨어 장비로 메모리를 덤프뜨는 두가지 방법이 있으며 이때는 후자로만 제대로 분석 할 수있습니다.]

$1/$2 의 내용은 DRAM(Main Memory)와 미러링, 즉 빨리 접근하기위해 담아두는 공간이라고는 하지만.

실제 운영체제나 프로그램 설계에서는 linux의 per-cpu 변수 처럼 대체적으로 각 아키텍처의 $1(L1 캐시) 사이즈와 같거나 그보다 작은 크기를 블럭단위로 하며 SMP 시스템 (멀티 코어 시스템)에서 더 DRAM에 저장되지 않고 더 빠르게 읽고 쓰기위해 DRAM에 주소와 메모리 정보가 저장은 되어있지만 사실상 DRAM보다는 캐시에 정확한 정보가 적혀있다고 전제되는 값들이 있습니다.  만약 $1/$2를 믿을수 없는 상황이 되면 per-cpu와 같은 특성을 가진 변수(컴퓨터 프로그램이 사용하는 값)은 제대로 쓸수도 없게 될것이며 이는 Instruction에도 비슷하게 적용이 될수도있습니다. 

진짜 이문제가 CPU에 인가되는 전력의 급격한 변화에 따른 것인지는 어디까지나 추측이지만

이렇게 많은 사람들이 $1/$2에 문제가 있다고 하는 것은 개인적인 소견으로서는 리콜 감입니다.

뒤에 있는 per-cpu어쩌구 저쩌구는 학부에서 배우진 않지만 CPU $1/$2는 학부에서 배우는 수준의 지식이며, 기본적인 설계에서 미스가 있는 것 이니까요.

per-cpu 에 대한 자세한 링크 : http://jake.dothome.co.kr/per-cpu/

## 그래서 AMD와 유통사는 어떻게 대응하는가
그래서 AMD와 유통사는 어떻게 대응하는가.
유통사가 어떻게 불량점검을 하는지 보면 부팅을 하고 벤치마크 프로그램을 돌립니다.
이게 끝입니다. 사실상 이러한 문제는 전문적으로 Trace32같은 장비를 쓰지않는 이상 문제를 찾기 어렵습니다.

![image](img/casts_double_amd_desktop_zen_2_and_3_halt_randomly_kr/trace32_manual.png)
[출처 : https://www2.lauterbach.com/pdf/general_ref_c.pdf 172페이지] 

Trace32 를 사용하면 실시간으로 CPU의 모든 값을 볼수가있습니다. 일반인 혹은 대다수의 개발자가 고작 할수 있는 방법은 그냥 메모리 주소에 접근하는것이 다이나 이건 DRAM의 값인지 Cache 의 값인지 알수가 없습니다. 이러한 고급 개발툴도 필요하지만 이런 도구를 써서 유통사가 AS를 한다는것은 사실상 불가능에 가깝습니다. 따라서 AS를 진행하시는 직원 분들도 해당 이슈에 대해서는 매우 곤란할 것입니다. 

소비자 입장에서도 이를 입증하고 교환받기가 매우 까다롭습니다. 

그럼 제 생각에 잘못은 AMD에 있습니다.

처음에 QC에 문제가 있거나, 실리콘 설계를 할때 이러한 문제를 제대로 검증 및 수정하지 않고 판매한 것이 문제라고 생각합니다.

그래서 AMD는 자기네 CPU의 결함에 대해서 공개를 하거나 공유하는가?

아뇨 인텔보다는 훨씬 숨기고있습니다.

일단 인텔은 Errata Sheet (설계결함, 이슈에 대해 정리한 페이퍼) 를 나온지 얼마 안된 13세대에 대해서 공유했습니다.

[ **Intel Raptor Lake S - Errata Details** ]
https://edc.intel.com/content/www/us/en/design/products/platforms/details/raptor-lake-s/13th-generation-core-processor-specification-update/errata-details/

그런데 AMD는 소비자용 Zen2 : Family 17h Model 71h , Zen3 : Family 19h Model 21h에 대한 Errata Sheet를 공개하지 않았습니다.

칩을 구매해서 직접 PCB 회로를 만드시거나 리눅스 커널 드라이버를 작성하는 업무를 하시는 분들은 ErrataSheet를 들어보시고 가끔 거기서 문제점을 찾아 다시 부품 선정을 하거나 고려해서 소프트웨어 상으로 실리콘 버그를 해결 하는 경우가 있을 겁니다만

이렇게 심각한 문제에 대해서 아무런 고지를 하지않고 그냥 아름아름 인터넷상으로 C-State Disable하라는 글에 의존하면서 이러한 이슈를 숨기는것에 대해 매우 실망스럽습니다. 

앞으로는 이러한 메모리 계층 구조(memory hierarchy)에 심각한 문제가 있는 제품의 결함을 제대로 고시하지않는 AMD가 바뀌길 기대하며 memory 를 좋아하는 고양이 사진으로 이 글을 마칩니다.
![image](img/casts_double_amd_desktop_zen_2_and_3_halt_randomly_kr/memory_lambda.jpg)
