---
title: "러스트 임베디드 양산 제품 개발기 - 4 빌드 스크립트 활용"
date: 2023-11-13T21:00:00+09:00
draft: false
categories: ["My Frist Mass Production With Rust Embedded"]
tags: ["회고록", "Rust", "Embedded", "Korean_Article"]
---

{{< alert >}}
**WARN!** 아직 작성 중인 글입니다. 중도에 내용이 변경될 수 있습니다.
{{< /alert >}}

![constcat](feature.jpg)

## 읽기 앞서서
이 글을 읽기 전에 rust 문법에 대한 이해가 부족하다면 이전 글 [2탄 기초 공부 방법 및 특징](/posts/my_first_commerical_rust_embedded_product_2/#러스트-공부) 에 적힌 #러스트-공부 문단을 읽고 올 것을 권합니다. 이 외에도 김기오 님이 남겨주신 좋은 글[평범한 C개발자의 Rust입문기: Rust에 입문하는 C개발자를 위한 안내서](https://wikidocs.net/book/12811)또한 추천드립니다.

## 빌드 스크립트 (`build.rs`)



-------------------------------
{{< alert icon="fire" cardColor="#4f5a63" iconColor="#1d3557" textColor="#f1faee" >}}
이 시리즈의 다른 글도 같이 봐주세요 : 
[러스트 임베디드 양산 제품 개발기](/categories/my-frist-mass-production-with-rust-embedded/)
{{< /alert >}}