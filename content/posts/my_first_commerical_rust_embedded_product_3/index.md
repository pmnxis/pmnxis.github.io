---
title: "러스트 임베디드 양산 제품 개발기 - 3 컴파일 타임에 맡기세요"
date: 2023-11-12T21:00:00+09:00
draft: false
categories: ["My Frist Mass Production With Rust Embedded"]
tags: ["회고록", "Rust", "Embedded", "Korean_Article"]
---

{{< alert >}}
**WARN!** 아직 작성 중인 글입니다. 중도에 내용이 변경될 수 있습니다.
{{< /alert >}}

![constcat](feature.jpg)

## 서론
이 글을 읽기 전에 rust 문법에 대한 이해가 부족하다면 이전 글 [2탄 기초 공부 방법 및 특징](/posts/my_first_commerical_rust_embedded_product_2/#러스트-공부) 에 적힌 #러스트-공부 문단을 읽고 올 것을 권합니다. 이 외에도 김기오 님이 남겨주신 좋은 글[평범한 C개발자의 Rust입문기: Rust에 입문하는 C개발자를 위한 안내서](https://wikidocs.net/book/12811)또한 추천드립니다.

## 컴파일 타임에 맡기세요
일반적인 함수는 컴파일 내에 결과가 예측이 되면 상수 (불변의 값)을 그대로 사용한다.

하지만 불변의 값으로 저장하는 것이 비효율적이라고 판단될 시 함수 형태로 instruction 상에 남아있다.

임베디드, 특히 저가의 MCU위에서 동작하는 펌웨어의 경우 램의 용량은 매우 작다. (필자가 사용한 MCU `STM32G030C8` 은 8KiB이다.).

최대한 컴파일 시간에 복잡한 연산도 상수의 성격을 띠는 데이터로 남긴 뒤 Flash 영역 (.text 혹은 .rodata) 에 적재시키기 위해서 일반적인 러스트 프로그래밍을 가끔씩 벗어나 이를 고려하여 코드를 작성했다.

### `const fn`
`const fn`은 일반 `fn`과는 다르게 컴파일 타임에 상수 성격을 띠는 데이터를 가져오는 것을 보장한다.

```rs
const fn const_add(a: i32, b: i32) -> i32 {
    a + b
}

const fn const_add_round_up(a: i32, b: i32) -> i32 {
    let added = const_add(a, b); // 안에 있는 함수는 무조건 const 형태여야 한다.
    (added / 10) * 10
}
```

하지만 그만큼 제약도 많다. `const fn` 스코프 안에 있는 함수는 `const fn`로 구성되어 있어야 하며, 
다른 연산자들 또한 컴파일 타임 내에 연산이 가능한 형태여야만 한다.

반대로 일반적인 `fn` 혹은 `async fn` 스코프 안에서는 `const fn`을 사용할 수 있다.

### `const impl`
일반적인 함수를 `const` 형태로 만드는 것은 간단하지만 구조체 struct를 위한 함수로 만드는 순간 어려운 문제들에 봉착하게 된다.
이 부분은 후술하게 될 `const trait` 에서 다루게 된다.

우선은 가장 간단한 특정 구조체 struct의 default에 대해서 다뤄보도록 하겠습니다.

```rs
pub enum Player {
    Undefined = 0,
    Player1 = 1,
    Player2 = 2,
}

impl Player {
    pub const fn default() -> Self {
        Self::Undefined
    }
}
```

결과적으로 코드는 매우 간단하지만 이와 같이 사용함으로서 `default()` 를 호출하는 쪽에서는 컴파일 타임 내에 상수를 얻을 수 있다.
더 복잡한 것을 원하는 경우에는 복잡한 값을 넣어주면된다. 위에서도 언급한것과 인자가 유동적이지 않는 한 함수 형태로 남지않기에 
분기문, 스택에서 왔다 갔다 하는 과정, 그 와중에 레지스터를 백업하고 롤백 하는 과정의 생략을 보장받을 수 있다.

### `const trait`
이제 대망의 `const trait` 이며 글을 작성한 시점(2023년 11월)에선 **nightly** feature이다.
여기서 부터는 왜 러스트가 `const fn / impl / trait` 이 아직까지 논의중인 RFC인지 동시에 서술한다.

#### 이미 존재하는 trait과의 충돌
위에서 언급한 `pub const fn default()`에서 이상한 점을 찾았는가?

그것은 바로 [`Default` trait](https://doc.rust-lang.org/core/default/trait.Default.html)이 이미 따로 있다는 것이다.

하지만 [`Default` trait](https://doc.rust-lang.org/core/default/trait.Default.html) 의 정의는 `const` 형태가 아니다.

불특정 다수의 구조체 struct가 `core::default::Default` trait의 impl가 딱 봐도 너무 간단한 경우가 많다.
하지만 눈대중으로 판단하기에 아무리 간단하더라도 이 경우엔 `const fn` 안에서는 사용할 수 없다.

#### `Into/From`의 경우
아직까지는 `const` 특성을 이용해 `Default` trait을 통한 추상화를 할 일은 거의 없었지만 외외로 [into, from](https://doc.rust-lang.org/core/convert/trait.Into.html) 가 문제였다.

현재로서는 [`core::default::From`](https://doc.rust-lang.org/src/core/convert/mod.rs.html#704-728) 의 Trait정의를 const trait에 맞게 `ConstInto`, `ConstFrom`으로 수정하여 사용하면 아래와 같이 사용할 수 있다.
  > - [Product에 사용한 ConstConvert 정의](https://github.com/pmnxis/billmock-app-rs/blob/0.3.1/src/types/const_convert.rs)

**const 형태를 띄는 Into/From 의 사용 예시**
```rs
pub struct UnpackedQuad4Bits {
    pub b0: u8,
    pub b1: u8,
    pub b2: u8,
    pub b3: u8,
}

#[derive(PartialEq)]
pub struct PackedQuad4Bits {
    inner: u16,
}

impl const ConstFrom<UnpackedQuad4Bits> for PackedQuad4Bits {
    fn const_from(value: UnpackedQuad4Bits) -> Self {
        PackedQuad4Bits {
            inner: ((value.b0 as u16 & 0xF) << 12)
                | ((value.b1 as u16 & 0xF) << 8)
                | ((value.b2 as u16 & 0xF) << 4)
                | (value.b3 as u16 & 0xF),
        }
    }
}

#[test]
fn test() {
        assert_eq!(
        PackedQuad4Bits::const_from(UnpackedQuad4Bits {
            b0: 0x0, b1: 0x1, b2: 0x2, b3: 0x3 }),
        PackedQuad4Bits { inner: 0x0123 }
    );
}
```

ConstInto / ConstFrom과 const 형태로 만들어서 사용하면 기존의 Into/From으로 `.into()` 형태로 곧바로 사용할 수 없으며
또다시 Into/From trait 정의에서 앞의 const 메서드를 호출해야 하지만
컴파일 타임 안에 into를 정의할 수 있다.

#### nightly feature인 `const trait`
아직까지 `const trait`은 nigtly feature이다. 개인적인 감상으로는 niglty compiler의 버전을 조금씩 바꿀 때마다 플래그 변경을 해줘야만 그대로 쓸 수 있었다.
하지만 그럼에도 불구하고 경우에 따라서는 매우 필요한 기능이다.(물론 안 쓰고 `const fn` 만 써서 할 수 있다.)

`const trait`을 정의하는 경우에는 `#[const_trait]`을 trait 정의 앞에 붙여줘야하며 `lib.rs` 혹은 `main.rs`에 `#![feature(const_trait_impl)]` 를 선언해줘야한다.

{{< alert >}}
**todo!** `const trait` 에 대한 논의 역사 서술 
{{< /alert >}}

- [Tracking issue for RFC 2632, impl const Trait for Ty and ~const (tilde const) syntax](https://github.com/rust-lang/rust/issues/67792)

-------------------------------
{{< alert icon="fire" cardColor="#4f5a63" iconColor="#1d3557" textColor="#f1faee" >}}
이 시리즈의 다른 글도 같이 봐주세요 : 
[러스트 임베디드 양산 제품 개발기](/categories/my-frist-mass-production-with-rust-embedded/)
{{< /alert >}}