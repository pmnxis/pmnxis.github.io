---
layout: default
title: 구름에듀 강의자 Rust 환경 세팅
subtitle: "Performance drop in groom edu with rust"
type: "TIL"
createDate: "2021-12-31"
til: true
text: true
author: Jinwoo Park
post-header: false
header-img: ""
order: 3
---

### Rust에서의 코딩 테스트
최근에 취미삼아서 Rust로 알고리즘 공부를 조금씩 하고 있었습니다만, 최근에 우연치 않게 Vector의 Binary Search가 C++과 비교하여 매우 느려 문제를 통과 못하는 상황에 직면했으며 Local에서 테스트를 해본 결과 goorm 사이트 내에서 빌드옵션을 `unoptimized` 상태로 빌드한 것이 문제인 것으로 판단되었습니다.

수강자가 이를 해결할 수 는 없으나 잠시나마 강의자의 권한을 빌려 이를 수정하여 해결해보았습니다.

### 솔루션 (채점자/강의자)
![Lambda_With_Turtle](/assets/img/rustccc.png)

```sh
# 빌드 옵션
-C opt-level=3

# 실행 옵션 (빈칸)

```
이와 같이 빌드옵션과 실행옵션을 바꿔주면됩니다.