---
title: "Chama Optics 개발기"
date: 2026-02-14T00:00:00+09:00
draft: false
categories: ["Chama Optics"]
tags: ["Rust", "iOS", "Cross-Platform", "EXIF"]
description: "EXIF 기반 사진 프레임 + 얼굴 자동인식 프로그램, Rust 코어로 데스크탑부터 모바일까지"
---

> EXIF 기반 사진 프레임 + 얼굴 자동인식 프로그램, Rust 코어로 데스크탑부터 모바일까지

이 블로그에 오타쿠스러운 글을 대놓고 쓰는 건 이번이 아마 처음일 것이다.
솔직히 이 글을 쓰기 시작한 지는 꽤 되었는데, 청자를 개발자에게 맞춰야 할지 VTuber 오타쿠에게 맞춰야 할지 감이 잡히지 않았다.
결국 그냥 의식의 흐름대로 개발하고 기여한 것들을 나열하기로 했다.

이 블로그에서는 주로 Rust Embedded를 다루고 있었고, 이전에 [billmock-app-rs](https://github.com/pmnxis/billmock-app-rs)라는 Rust Embedded 양산 프로젝트를 진행한 바 있다.

이 글에서는 [Chama Optics](https://github.com/pmnxis/chama-optics)의 개발 과정을 소개한다.

2026년 2월 마지막 주에 **0.2.0**에 iOS / Android / macOS / Linux / Windows 의 정식 배포를 할 예정이며 App Store, Google Play 승인 전 개발 과정을 서술하고 있습니다.

> 🌐 [English Article](/en/posts/chama-optics-dev-story/) | [日本語アーティクル](/ja/posts/chama-optics-dev-story/)
>
> 어플리케이션 소개 및 릴리즈 노트는 [여기](/ko/posts/chama-optics-public-release/)에서 볼 수 있습니다.

---

## 프로젝트 소개

![Chama Optics](haachama-optics.webp)

**Chama Optics**는 DSLR/미러리스 카메라로 촬영한 사진의 EXIF 데이터를 분석하여 다양한 테마 프레임을 적용하고, 워터마크·모자이크·스티커 등의 효과를 추가할 수 있는 사진 후처리 프로그램이다. "Chama"라는 이름은 여행 VTuber Akai Haato(赤井はあと)의 별칭에서 유래했다.

수많은 모바일 기기를 거쳤지만 내 관심은 항상 전자기기를 **만드는** 쪽이었고, 스마트폰 앱 개발과는 거리가 멀었다. 그런 내가 Hololive JP 1기생 아카이 하아토(HAACHAMA)의 팬으로서 오타쿠 생활을 하면서, 임베디드가 아닌 소프트웨어 개발 영역에서 첫 프로젝트를 시작하게 되었다.

이 프로그램을 처음 구상한 것은 2025년 3월부터이다.
당시에는 웹앱으로 동작하길 원했으며, 라이브러리 테스트, WASM 환경 테스트, libheif 포팅 등을 진행하고 있었다.
2025년 8월, 일본 도쿄에서의 아마네 카나타 솔로라이브 LOCK-ON과 미국 뉴욕에서의 AnimeNYC World Tour + EN Concert(All for one)를 다녀온 뒤, 사진을 빠르게 정리하고 WEBP로 압축해서 게시할 필요성을 절감했다.
동시에 아카이 하아토도 최근 사진 찍는 것을 좋아하여, 멤버십 한정 글로 자기가 쓰는 카메라를 보여주거나 오시카츠하아톤일기([#推し活はあとん日記](https://x.com/hashtag/%E6%8E%A8%E3%81%97%E6%B4%BB%E3%81%AF%E3%81%82%E3%81%A8%E3%82%93%E6%97%A5%E8%A8%98))에서 사진 게시를 유도하고 있었기에, 하아토(HAACHAMA) 이름으로 앱을 하나 개발해주고 싶었다.

| 최근 3D Live | Akai Haato X(twitter) |
| ----------- | --------- |
| {{< x user="akaihaato" id="1954294114519851374" >}} | {{< x user="akaihaato" id="1916078150804799563" >}} |

프로그램을 개발할 때에는 아래의 철칙을 따랐다.
- 데스크탑상에서 어떠한 아키텍처 차별이 있어서는 안 된다.
- MS나 Apple의 개발 생태계에 최소한으로 얽매여야 한다.
- 리소스를 많이 사용하면 안 되며 빨라야 한다.

목표는 무슨 거창한 것처럼 보이지만 내가 그냥 Rust 매니아라서 저게 목표다.

---

## 시작: EXIF 프레임을 좀 더 편하게

처음부터 거창한 크로스플랫폼 앱을 계획했던 건 아니었다.

사진기 매니아들 사이에서는 사진을 올릴 때 EXIF 정보를 바탕으로 카메라 모델명, 렌즈, 셔터스피드 등을 프레임에 넣어 공유하는 문화가 있다. 나도 이 방식을 즐겨 사용하고 있었고, 기존에는 [exif-frame](https://github.com/jeonghyeon-net/exif-frame)이라는 웹 기반 도구를 참고하여 사용하고 있었다. 하지만 HEIF 포맷을 지원하지 않는 점과 고해상도 이미지 출력에 제한이 있는 점이 아쉬워서, 직접 만들자는 생각에서 Chama Optics가 시작되었다.

처음에는 **데스크탑만을 고려**했다. 모바일에 대한 막연한 상상은 있었지만, 여행지에서도 어차피 언제나 맥북을 들고 다녔기에 모바일은 전혀 염두에 두지 않았다. 카메라에서 SD 카드를 빼고, 맥북에서 사진을 정리하고, 프레임을 입혀서 올리는 — 그 워크플로우가 당연했으니까.

---

## 방향 전환: "행사장에서 맥북을 꺼낼 순 없잖아"

방향이 바뀐 건 두 가지 계기가 있었다.

### 행사 문화의 차이 — AnimeNYC에서 느낀 것

2025년 8월, **AnimeNYC**와 **Hololive World Tour / EN Concert**를 위해 미국을 방문했다. 그때 재밌는 차이를 느꼈다. 미국에서는 행사장에서 찍은 사진을 올릴 때 **사람들의 얼굴을 모자이크하지 않는 경향**이 강했다. 하지만 한국과 일본의 행사에서는 **다른 사람의 얼굴을 반드시 모자이크 처리**하는 것이 예의이자 암묵적인 규칙이었다.

![AnimeNYC 행사 사진 예시](P1090028-OPTICS.webp)

"다른 사람 얼굴 모자이크"는 매번 수작업으로 하기엔 너무 번거로운 작업이다. 특히 사진이 수십, 수백 장이 되면 더욱. **자동 얼굴 인식 + 모자이크/스티커 기능**이 필요하다는 생각이 이때부터 강하게 들었다.

### 곧 다가올 홀로라이브 엑스포

또 하나의 동기는 **2026년 3월의 홀로라이브 엑스포/페스티벌**이었다. 행사장에서 바로 사진을 찍고, 그 자리에서 프레임을 입히고, 모자이크까지 처리해서 SNS에 올릴 수 있다면? 그런데 행사장에서 맥북을 열 순 없다. **스마트폰에서 바로 처리할 수 있어야 했다.**

### 주변의 요청

여기에 주변에서 **iOS 버전 개발에 대한 요청**이 더해졌다. 그래서 iOS용을 개발하기 시작했더니, 이번에는 **Android 버전에 대한 요청**도 들어왔다.

이렇게 해서 데스크탑 전용이었던 프로그램이 iOS, 나아가 Android까지 지원하는 방향으로 확장되었다. 모바일에서는 미러리스 카메라 사용자보다는 **일반 사용자의 경험**을 더 고려하는 쪽으로 설계 방향을 잡았다. EXIF 프레임이라는 출발점은 그대로지만, "행사 현장에서 빠르게 사진을 처리해서 공유한다"는 새로운 사용 시나리오가 추가된 셈이다.

---

## 아키텍처: 데스크탑에서 시작해 모바일까지

처음에는 Rust + egui로 데스크탑 앱만 만들 생각이었다. 그래서 핵심 로직을 모두 Rust로 작성한 것이 결과적으로 좋은 선택이 되었다. iOS/Android로 확장할 때 **이미지 처리, EXIF 파싱, 테마 렌더링, 인코딩/디코딩 같은 핵심 코드를 그대로 재사용**할 수 있었기 때문이다.

![ChamaOptics Architecture](architecture_layers.svg)

데스크탑에서는 Rust 코어를 **직접 링킹**하여 사용하고, iOS에서는 **C FFI를 통해 Swift에서 호출**, Android에서는 **JNA(Java Native Access)를 통해 Kotlin에서 호출**하는 구조다. Rust 코어는 git submodule로 관리되며, EXIF 해석, 이미지 오버레이(텍스트, EXIF, 여백, 스케일링, 인코딩/디코딩) 같은 핵심 기능을 모든 플랫폼에서 공유한다.

단, 얼굴 인식만은 플랫폼마다 다른 전략을 사용한다.
- **데스크탑(macOS/Windows/Linux)**: ONNX Runtime + InsightFace(SCRFD det_10g) 모델. Speed Mode에 따라 640×640 고정 입력 크기의 슬라이딩 윈도우를 다단계(2560/1280/640)로 적용하여 작은 얼굴까지 감지하고, NMS로 중복을 제거하는 파이프라인
- **iOS**: Apple Vision Framework를 네이티브로 사용. ONNX 모델 없이도 빠르고 정확하며, 프라이버시 면에서도 유리
- **Android**: Google ML Kit (`com.google.mlkit:face-detection`) 활용 — Google이 제공하는 온디바이스 얼굴 인식 라이브러리, Rust 코어의 speed_mode를 FAST/ACCURATE 퍼포먼스 모드로 매핑

---

## Web 버전을 포기하게 된 이유

나는 Web App이나 Web과 브라우저의 동작 방식에 대해서 해박하지 않다. 그럼에도 Chama Optics는 초기에 **Web(WASM)에서의 구동을 염두에 두고 있었다.** egui가 WASM을 지원하니까 "데스크탑이랑 웹이랑 동시에 되겠지"라는 막연한 기대가 있었다. 하지만 다음 두 가지 기능을 구현하면서 포기하게 되었다.

- **HEIF 디코딩**
- **egui Web에서의 Drag & Drop**

### HEIF: WASM 위에 WASM, 그 사이에 JS

libheif를 브라우저에서 돌리기 어려운 것 이전에, 근본적인 구조가 납득이 되지 않았다. libheif는 이미 WASM으로 컴파일된 상태이고, egui 앱 또한 WASM이다. 이 둘 사이의 통신을 **JavaScript를 통한 FFI로 몇 번이나 거쳐야 한다**는 것이 이해가 되지 않았다. 대부분의 언어 간 FFI는 C 기반으로 하는 것에 반해, 왜 JS 생태계에서는 이렇게 해야 하는지 이해할 수 없었다.

### Drag & Drop: 데스크탑 개발자의 기대와 현실

Drag & Drop 이외에도, WASM이 브라우저로부터 이벤트를 받을 때 JS가 아닌 DOM을 바이너리 형태로 받는다거나, 좀 더 기존 데스크탑/임베디드 개발에서 쓰일 법한 방법이 제공될 줄 알았으나 아니었다.

### 솔직히 C와 Rust밖에 쓸 줄 모른다

나는 **C와 Rust밖에 쓸 줄 모른다.** 즉, Web 개발 자체에 대해서 매우 무지하거나, 과거에 하더라도 이상한 방식으로 개발했다.

예전에 수많은 데이터를 웹에 나열해야 할 일이 있었는데, 어떻게 할지 몰라서 넣을 데이터들을 CSV로 만들고 `,`와 `\n`을 `<div>` 등의 HTML 태그로 변경하는 것을 **hex editor로 전체 치환**해서 static web을 만들어 배포한 적이 있다. 21세기가 25%나 지나간 현대 프로그램 개발에 있어서 스스로도 "이게 뭐요" 싶었다.

물론 WASM은 웹이기에 웹 생태계와 개발자들의 방식을 따르는 것이 보편적일 것이다. 하지만 나는 웹 개발자가 아니기 때문에 이해가 되지 않았다. 나는 **JS/Web 생태계와 매우 거리가 멀었고**, 이런 개발 환경 자체의 방향성 차이를 극복하는 것보다 네이티브 모바일 앱을 만드는 게 훨씬 자연스러웠다. v0.1.9-beta에서 공식적으로 WASM 지원을 제거했고, 그 에너지를 iOS/Android 네이티브에 쏟기로 했다.

---

## 타임라인으로 보는 개발 여정

### v0.1.0\~v0.1.1 (2025-10-19\~21) — 첫 프리릴리즈

![v0.1.0 스크린샷](https://github.com/user-attachments/assets/2471db65-8b0b-44e4-9d2b-31701184878e)

macOS/Windows 바이너리 첫 배포. Film 테마 프레임, 일본어 번역, 일괄 저장, 파일명 접두어/접미어 설정. macOS 코드 사이닝 DMG 배포 및 한/영/일 설치 가이드 위키 작성.

### v0.1.2\~v0.1.6 (2025-10-27\~11-24) — 테마 확장과 워터마크

| Film Date 테마 | Strap 테마 | Monitor 테마 | Lightroom 테마 |
|:---:|:---:|:---:|:---:|
| ![Film Date](https://github.com/user-attachments/assets/a6bf0e51-d3b1-4779-9d65-080b225958f4) | ![Strap](https://github.com/user-attachments/assets/039ab49f-85b1-414b-95e3-2da166cea27f) | ![Monitor](https://github.com/user-attachments/assets/e92b81a0-4465-4dad-9097-7e8b4814fc15) | ![Lightroom](https://github.com/user-attachments/assets/ce1022cd-aea3-4260-9d5f-5e15997388de) |

Film Date/Film Glow/Just Frame/Strap/Monitor/Lightroom 테마 추가. 워터마크(위치 9곳, 투명도, 블렌드 모드), 폰트 선택(내장 + OS 폰트), 내장 카메라 제조사 로고 자동 적용, HEIF 방향 수정, 가변 폰트 초기 지원, Longside 스케일 옵션.

### v0.1.7 (2025-11-26\~12-19) — CJK 렌더링 개선과 오픈소스 기여

| One Line 테마 | Shot On Two Line 테마 |
|:---:|:---:|
| ![One Line](https://github.com/user-attachments/assets/337337f3-7c17-467a-b965-06481cba98c8) | ![Shot On 2](https://github.com/user-attachments/assets/a9ba4540-6540-418c-8e21-4f9961d7bff7) |

| Nikon PhotoStyle | Lumix Photo Style + LUT |
|:---:|:---:|
| ![Nikon](https://github.com/user-attachments/assets/28016fd2-2d4d-4043-88e1-c29f7577a32a) | ![Lumix](https://github.com/user-attachments/assets/62219e6a-c6cb-49ac-9803-cda29321b998) |

One Line/Two Line/Shot On 테마. CJK 글리프 렌더링 대폭 개선 및 SourceHanSans fallback 내장. Lumix LUT·Nikon PhotoStyle 이름을 EXIF에서 추출하기 위해 [exif-rs](https://github.com/kamadak/exif-rs)에 PR 제출 후 선반영.

### v0.1.8 (2025-12-25\~27) — UI 리뉴얼과 성능 개선

| 이미지 리스트 탭 | 테마 설정 탭 |
|:---:|:---:|
| ![Tab1](https://github.com/user-attachments/assets/798d9c93-833a-4e9f-876d-ee3fe7182dab) | ![Tab2](https://github.com/user-attachments/assets/4dd969fa-6f75-4f73-8c8a-f89ac66e1b1b) |

탭 기반 인터페이스 전환(4개 탭), EXIF 변수 자동완성, 이미지 자동 그룹화, 2MP MPF 프리뷰 기반 테마 미리보기, Rayon 멀티코어 병렬 처리, 시스템 폰트 로딩 메모리 이슈 수정. [egui](https://github.com/emilk/egui)에도 PR 제출.

### v0.1.9 (2026-01-18\~02-04) — 얼굴 인식, LUT, iOS 첫 배포

| 얼굴 인식 (데스크탑) | 모자이크 적용 |
|:---:|:---:|
| ![Face Detection](https://github.com/user-attachments/assets/a038cdaa-f755-4d8f-97c9-71f57004e739) | ![Mosaic](https://github.com/user-attachments/assets/6ed05280-1980-448a-9243-aff0836d2470) |

| 컬러 그레이딩 UI | LUT 적용 결과 |
|:---:|:---:|
| ![LUT UI](https://github.com/user-attachments/assets/554f96b2-217a-4603-93f5-1df41309f77e) | ![LUT Result](https://github.com/user-attachments/assets/4d49174c-2624-4b3c-a8a1-fc2b56d357c1) |

| iOS 갤러리 | iOS 편집 |
|:---:|:---:|
| ![Gallery](previews-d2OPTICS.webp) | ![Editor](color-themes-d2OPTICS.webp) |

데스크탑 단독 릴리즈 마지막이자 iOS 앱 첫 배포 버전. ONNX(InsightFace) 얼굴 감지 + 모자이크/스트로크/스티커 오버레이. 1D/3D LUT 컬러 그레이딩([wagahai-lut](https://github.com/pmnxis/wagahai-lut)). iOS는 SwiftUI + Vision Framework 네이티브 얼굴 인식, Rust FFI 브리지(`ffi_ios.rs` + `RustBridge.swift`). 인도네시아어 번역 추가.

---

## 기술적 도전과 해결

프로젝트 전반에 걸쳐 적용된 성능 전략들을 먼저 정리하면 다음과 같다.

- **Rayon 병렬 처리** — 대량 이미지 일괄 내보내기 시 멀티코어 활용. 단, 색상 보정 등 픽셀 단위 처리에서는 10만 픽셀 이상인 경우에만 `par_chunks_exact_mut()`으로 병렬화하고, 작은 이미지는 컨텍스트 스위칭 오버헤드를 피하기 위해 순차 처리한다.
- **`fast_image_resize` 기반 리사이징** — `image` 크레이트의 기본 리사이즈 대신 SIMD 최적화된 `fast_image_resize`를 사용하여 썸네일 생성 및 프리뷰 리사이징 속도를 크게 개선
- **Lazy 로딩과 캐싱** — LUT 파일은 `lut_cache: HashMap<Uuid, CubeLut>`에 최초 사용 시 파싱하여 캐싱하고, EXIF 썸네일도 `thumbnail_cache`에 지연 로드한다. 백그라운드 스레드용 복제(`clone_for_thread()`)시에는 캐시를 제외하여 불필요한 메모리 복제를 방지한다.
- **퍼셉추얼 해시 기반 이미지 그룹화** — 이미지 로드 시 8×8 그레이스케일 평균 해시(64-bit)를 미리 계산하여, 이후 유사 이미지 그룹화를 해밍 거리 O(1) 비교로 수행. 원본 이미지를 다시 로드하지 않고 메타데이터만으로 그룹화한다.
- **빌드 프로파일 최적화** — Release 빌드에서 `opt-level = 3`, `lto = "fat"`, `codegen-units = 1`을 적용하고, Dev 빌드에서도 `fast_image_resize`, `mozjpeg`, `ab_glyph` 등 성능에 민감한 의존성은 `opt-level = 3`으로 개별 설정하여 디버그 중에도 이미지 처리 성능을 유지한다.

### 1. 크로스플랫폼 FFI의 복잡성

Rust 코어를 3개 플랫폼(데스크탑/iOS/Android)에서 사용하기 위해 각각 다른 FFI 전략을 택했다.

| 플랫폼 | FFI 방식 | 특징 |
|:---|:---|:---|
| 데스크탑 (egui) | 직접 링킹 | Rust → Rust, FFI 불필요 |
| iOS (SwiftUI) | C FFI (`@_silgen_name`) | Swift에서 C 함수 직접 호출 |
| Android (Compose) | JNA (Java Native Access) | Kotlin에서 JNA를 통해 .so 호출 |

이 브리지 계층을 유지보수하면서도 안정적인 메모리 관리(문자열 할당/해제, 불투명 포인터 핸들 패턴)를 보장하는 것이 주요 과제였다.

### 2. EXIF 파싱의 끝없는 변수

카메라마다 EXIF 기록 방식이 다르다.
- **셔터스피드/F값의 부동소수점 문제** — 1/125초가 `0.008000000` 같은 지저분한 값으로 기록되는 경우 자동 보정
- **HEIF/HEIC 방향 정보 오류** — 일부 이미지에서 방향이 틀어지는 문제
- **렌즈 정보 없는 카메라** — Nikon Coolpix 같은 컴팩트 카메라 대응
- **MakerNote에 숨겨진 정보** — Lumix LUT명, Nikon PhotoStyle, Sony Creative Look 등 제조사별 비공개 EXIF 필드 파싱

이를 위해 [exif-rs](https://github.com/kamadak/exif-rs) 라이브러리에 직접 PR을 보내 필요한 기능을 추가했다.

### 3. MakerNote 파싱: 제조사별 촬영 설정 추출

최근 미러리스 카메라들은 자체적으로 색감을 맞춰주는 기능이 매우 뛰어나다. Lumix의 Photo Style, Nikon의 Picture Control, Sony의 Creative Look 등이 그것이다. 사진가들 사이에서는 "어떤 색감 설정으로 찍었는지"가 카메라 기종이나 렌즈만큼이나 중요한 정보인데, 이 정보를 프레임에 같이 넣어줄 수 있으면 좋겠다는 생각이 들었다.

[EXIF 표준](https://www.cipa.jp/std/documents/download_e.html?DC-008-Translation-2023-E)의 **MakerNote**(Tag 0x927C)는 카메라 제조사가 자유롭게 사용할 수 있는 비표준 영역이다. 포맷이 제조사마다, 심지어 같은 제조사의 모델마다 다르고, 문서화도 빈약하다. 하지만 여기에는 **"어떤 색감 설정으로 찍었는지"** 같은, 사진가에게 중요한 정보가 숨어 있다.

![MakerNote 파싱 흐름](makernote_parsing.svg)

Chama Optics에서는 `exif.maker_note_vendor()`로 제조사를 먼저 식별한 뒤, 각 제조사별 전용 파서로 분기한다.

**Nikon** — `PictureControlData` / `PictureControlData2` 태그에서 Picture Control 이름을 추출한다. `"VitalityFilm_Pmango"` 같은 사용자 정의 프로필명이나, `"Flat"`, `"Vivid"` 같은 프리셋 이름이 여기에 들어 있다.

**Panasonic (Lumix)** — 가장 풍부한 데이터를 제공한다. `PhotoStyleName`에서 기본 Photo Style 이름(`"NostalgicKintex"`)을, `LutPrimaryFile`/`LutSecondaryFile`에서 적용된 LUT 파일명(`"KintexYellow33.CUBE"`)과 Gain 값까지 추출한다. 이 정보는 Chama Optics의 LUT 컬러 그레이딩 기능과 직접 연결된다.

**Sony** — `Sony_0x9416` 태그에서 Creative Style/Creative Look 정보(`"Vivid"`, `"Standard"`, `"Portrait"` 등)를 추출한다.

이 MakerNote 파싱 기능은 기존 exif-rs에 없었기 때문에 직접 구현하여 [PR #57](https://github.com/kamadak/exif-rs/pull/57)로 제출했다.

#### EXIF IFD 엔트리 구조와 MakerNote의 offset 문제

MakerNote를 파싱하려면 먼저 EXIF의 IFD(Image File Directory) 구조를 이해해야 한다. EXIF 데이터는 TIFF 포맷을 기반으로 하며, 각 IFD 엔트리는 정확히 **12바이트**로 구성된다.

- **Tag** (2바이트) — 필드 식별자 (예: `0x927C` = MakerNote)
- **Type** (2바이트) — 데이터 타입
- **Count** (4바이트) — 값의 개수
- **Value/Offset** (4바이트) — 데이터가 4바이트 이하면 값 자체, 초과하면 **데이터의 위치를 가리키는 offset**

표준 EXIF에서 이 offset은 **TIFF 헤더 시작점으로부터의 거리**다. 단순명쾌하다. 그런데 MakerNote 내부의 IFD에서는 이 규칙이 무너진다.

![EXIF IFD 엔트리 구조와 MakerNote offset 방식](exif_makernote_structure.svg)

**문제는 이것이다**: MakerNote 내부에도 IFD와 동일한 구조의 엔트리들이 존재하는데, 여기서 offset이 "어디를 기준으로 한 거리인지"가 **제조사마다 다르다.**

- **TIFF-Relative 방식** (Panasonic, Canon, Sony, Leica, Sigma) — MakerNote 안의 offset이 원본 TIFF 헤더 시작점 기준. MakerNote 자체에 TIFF 헤더가 없으며, offset에서 `tiff_offset`(TIFF 시작점부터 MakerNote까지의 거리)을 빼야 실제 데이터 위치를 찾을 수 있다.
- **MakerNote-Relative 방식** (Nikon, Olympus, Fujifilm, Samsung, Apple, Pentax) — MakerNote 안의 offset이 MakerNote 시작점 기준. 자체적으로 완결된 구조(self-contained)이며, Nikon의 경우 MakerNote 안에 독자적인 TIFF 헤더까지 가지고 있다.

추가로 바이트 오더(엔디안)도 제조사별로 다르다. Nikon은 자체 TIFF 헤더에서, Olympus/Apple은 프로프라이어터리 헤더 내의 `"II"`/`"MM"` 바이트에서, Samsung은 IFD 태그 번호의 패턴으로 자동 판별한다.

결국 10개 제조사(Panasonic, Nikon, Sony, Canon, Olympus, Fujifilm, Samsung, Apple, Sigma, Pentax)에 대해 각각의 헤더 포맷, offset 보정 공식, 바이트 오더 판별 로직을 구현하여, 총 23개 파일 약 5,900줄의 PR이 되었다.

### 4. 카메라 제조사 로고 시스템: CSV → build.rs → 바이너리 임베딩

Strap 테마, Film 테마 등에서 사진 프레임에 **카메라 제조사 로고**를 자동으로 삽입하려면 두 가지가 필요하다. (1) EXIF에서 제조사를 인식하는 것, (2) 해당 제조사의 SVG 로고를 렌더링하는 것.

#### 컴파일 타임: CSV에서 SVG 다운로드 & 임베딩

이전에 Rust Embedded 양산 프로젝트에서 [`const fn`/`const impl`으로 컴파일 타임에 최대한 맡기는 접근](/ko/posts/my_first_commerical_rust_embedded_product_3/)과 [`build.rs`를 활용한 빌드 스크립트 기법](/ko/posts/my_first_commerical_rust_embedded_product_4/)을 다룬 적이 있다. Chama Optics의 로고 시스템은 이 경험의 연장선에서 `build.rs` + `include_bytes!()`를 적극 활용한다.

![컴파일 타임 로고 파이프라인](logo_build_pipeline.svg)

`assets/logo_mnf.csv`에 35개 제조사의 로고 정보가 정의되어 있다. `cargo build`가 실행되면 `build.rs`가 이 CSV를 읽어 다음을 수행한다.

1. **SVG 다운로드** — 각 행의 `url` 컬럼에서 SVG를 가져온다. Wikimedia Commons URL이면 HTTP로 다운로드하고, 로컬 경로(`assets/logo_mnf/contax.svg`)이면 직접 읽는다. 네트워크 실패 시 최대 3회, 5초 간격으로 재시도한다.
2. **MD5 해시 검증** — 다운로드된 파일의 MD5 해시를 CSV의 `expected_md5` 값과 비교한다. **파일이 이미 존재하고 해시가 일치하면 재다운로드를 건너뛴다.** 해시가 불일치하면 `panic!`으로 빌드를 중단한다 — Wikimedia 측에서 SVG가 변경되었다면 의도적으로 확인해야 하기 때문이다.
3. **Rust 코드 생성** — `assets/auto_generated/logo_assets.rs`를 생성하며, 각 SVG를 `include_bytes!()`로 바이너리에 임베딩한다. 런타임에 파일 로딩이 필요 없다.

```rust
// 자동 생성되는 코드 예시
pub const LOGO_ASSETS: &[ArtAsset] = &[
    ArtAsset {
        key: "canon.svg",
        data: include_bytes!(".../assets/download/canon.svg"),
        color_type: ColorType::Color,
        mnf: "canon", model: "",
        mnf_model_rel: MnfRelation::Any,
    },
    // ... 35개 제조사
];
```

#### 런타임: EXIF → 로고 매칭 → SVG 래스터라이즈

![런타임 로고 매칭](logo_runtime_matching.svg)

사진이 로드되면 EXIF의 `Tag::Make`(제조사)와 `Tag::Model`(모델명)을 추출하고, `LOGO_ASSETS` 배열을 순회하며 매칭한다.

매칭 규칙은 두 가지다.
- **`MnfRelation::Any`** — 제조사명 **또는** 모델명 중 하나만 일치하면 됨 (대부분의 경우)
- **`MnfRelation::Both`** — 제조사명 **그리고** 모델명 둘 다 일치해야 함 (특수한 경우)

`Both`가 필요한 실제 사례: **Sigma**는 2025년에 로고를 변경했다. 새 로고를 사용하는 카메라는 `SIGMA BF` 모델뿐이므로, CSV에 `mnf="sigma", model="sigma bf", mnf_model_rel=Both`로 `sigma2025.svg`(신로고)를 등록하고, 나머지 Sigma 카메라는 `mnf="sigma", mnf_model_rel=Any`로 `sigma.svg`(구 로고)를 사용하도록 분리했다.

매칭된 SVG는 `usvg`로 파싱 후 `resvg`+`tiny-skia`로 래스터라이즈하여, 프레임 내 적절한 위치와 크기로 합성된다. `color_type`(Black/Color)과 `fill_ops`(Default/Monochrome)에 따라 렌더링 방식이 달라져, 배경색에 맞는 로고 표현이 가능하다.

### 5. CJK 폰트 렌더링과 가변 폰트(Variable Font) 최적화

일본어·한국어·중국어 텍스트를 이미지에 렌더링할 때 수많은 문제가 발생했다.
- 일부 CJK 한자(ideograph)가 렌더링되지 않는 문제
- 가변 폰트에서 글리프 폭이 맞지 않는 문제

해결책으로 **SourceHanSans를 빌트인 fallback 폰트**로 내장하여, 선택한 폰트에서 지원하지 않는 글리프를 자동으로 대체 렌더링하도록 했다. 구체적으로는 텍스트를 문자 단위로 순회하면서, 주 폰트에서 `GlyphId(0)` (글리프 없음)이 반환되면 SourceHanSans fallback 폰트로 전환하여 렌더링한다.

#### 가변 폰트 weight 재매핑

Chama Optics에서 사용하는 주 폰트인 **BarlowGX.ttf**는 가변 폰트(Variable Font)이지만, 내부 weight 축 값이 **22~188**이라는 비표준 범위를 사용하고 있었다. CSS 표준이나 FreeType 등에서 사용하는 **100~900** 범위와 맞지 않아, `ab_glyph`에서 `set_variation(b"wght", 400.0)`으로 Regular weight를 지정해도 의도한 결과가 나오지 않았다. 추가로 기본 width가 wdth=300(Condensed)으로 설정되어 있어 글리프 폭도 맞지 않았다.

단순히 `fvar`(Font Variations 메타데이터)만 수정하면 될 줄 알았지만, 실제 글리프 폭을 담고 있는 `hmtx` 테이블은 여전히 Condensed 기준이었다. **메타데이터만 바꾸면 렌더링 결과는 변하지 않는다.** 결국 BarlowGX.ttf에서 9개 weight 인스턴스를 wdth=500(Regular width)으로 추출하고, 이를 마스터 소스로 사용하여 `fontTools.varLib.build()`로 가변 폰트를 통째로 재빌드하여 해결했다. 결과물이 `Barlow-Variable-Remapped.ttf`와 `Barlow-Variable-Remapped-Narrow.ttf`이다.

#### 여러 폰트 파일을 하나로 합치기 — 절대적인 파일 사이즈 축소

가변 폰트의 또 다른 장점은 **여러 weight 파일을 하나로 합칠 수 있다**는 점이다. 기존에 Barlow-Thin.ttf, Barlow-Light.ttf, Barlow-Regular.ttf, Barlow-Bold.ttf, Barlow-Black.ttf 등 9개 이상의 정적 폰트 파일이 필요했던 것을, 가변 폰트 하나로 대체할 수 있다.

CJK 폰트도 마찬가지다. SourceHanSans(한일중 문자를 위한 최적의 선택)는 원래 weight별로 별도 파일이 제공되지만, 가변 폰트 버전(`SourceHanSansVF`)을 사용하면 하나의 파일로 200~800 범위의 weight를 모두 커버한다. 다만 이 폰트도 BarlowGX와 동일한 문제가 있어 weight 축을 표준 범위로 재매핑하여 `SourceHanSansVF-remapped.otf`를 생성했다.

더 나아가 `fontTools`를 활용하여 **서로 다른 문자 집합을 가진 폰트들을 하나로 병합**하는 작업도 진행했다. 라틴 문자 폰트 + 일본어 폰트 + 한국어 폰트를 합쳐 하나의 파일로 만들 수 있으며, WOFF2 압축 해제, TTC(Font Collection) 처리, 특정 weight로 인스턴스 추출, UTF-8 기반 문자 서브셋팅 등을 조합하여 최종 파일 크기를 최소화했다.

최종적으로 Chama Optics에 내장되는 폰트 파일은:

| 폰트 | 정적 폰트 시 용량 | 가변 폰트 용량 | 절감 |
|:---|---:|---:|:---|
| `Barlow-Variable-Remapped.ttf` (100~900) | ~1.35 MB (9 weight) | **385 KB** | **~3.5x** |
| `Barlow-Variable-Remapped-Narrow.ttf` (100~900) | ~1.45 MB (9 weight) | **207 KB** | **~7x** |
| `SourceHanSansVF-remapped.otf` (200~800) | ~105 MB (7 weight) | **30 MB** | **~3.5x** |
| `DejaVuSansMono.ttf` (정적) | — | 327 KB | — |
| `digital-7.ttf` (정적) | — | 34 KB | — |
| **합계** | **~108 MB** | **~31 MB** | **~3.5x** |

Barlow의 경우가 특히 극적이다. 원본 Barlow 프로젝트에는 9 weight × 3 width × 2 (upright+italic) = 54개 정적 TTF 파일이 있고 합계 **8.5 MB**인데, Chama Optics에서 필요한 normal + narrow 두 가변 폰트는 합쳐서 **592 KB**에 불과하다. 모바일 앱 번들 크기에 민감한 환경에서 이 차이는 결정적이다.

#### egui에서 가변 폰트 weight 선택 로딩

데스크탑 버전(egui)에서는 가변 폰트의 weight를 **사용자가 자유롭게 조절**할 수 있다. 핵심은 `ab_glyph` 크레이트의 `set_variation` API다.

```rust
pub struct VariableFontPack {
    pub label: &'static str,
    pub font: ab_glyph::FontRef<'static>,
    pub default: u16,       // 기본 weight (예: 300)
    pub start: u16,         // 최소 weight (예: 100)
    pub end_include: u16,   // 최대 weight (예: 900)
}

impl VariableFontPack {
    pub fn get_font_by_weight(&self, weight: u16) -> ab_glyph::FontArc {
        let clamped = weight.clamp(self.start, self.end_include);
        let mut font = self.font.clone();
        font.set_variation(b"wght", clamped as f32);
        font.into()
    }
}
```

사용자가 테마 설정에서 weight 슬라이더를 조절하면, 해당 weight 값으로 `set_variation(b"wght", weight)`를 호출하여 런타임에 폰트의 굵기가 변경된다. 100(Thin)부터 900(Black)까지 연속적인 값을 지정할 수 있으며, 350이나 450 같은 중간값도 보간(interpolation)되어 부드러운 weight 전환이 가능하다.

이 로직은 데스크탑뿐 아니라 iOS/Android에서도 동일하게 동작한다. iOS에서는 `FontSelectionView`에서 가변 폰트인 경우에만 weight 슬라이더를 표시하고, 선택된 weight 값을 FFI를 통해 Rust 코어에 전달한다. Android에서도 Kotlin에서 `fontWeight` 파라미터로 Rust FFI에 전달하는 동일한 구조다.

CJK fallback도 weight를 반영한다. 주 폰트가 Barlow weight 700(Bold)이고 CJK 문자가 나오면, SourceHanSans도 700에 가까운 weight로 렌더링하여 **라틴 문자와 CJK 문자의 굵기가 일관되게** 보이도록 했다.

#### 빌트인 폰트와 시스템 폰트

Chama Optics에서 사용하는 폰트는 두 종류로 나뉜다. **빌트인(builtin) 폰트**와 **시스템(OS) 폰트**.

빌트인 폰트는 앱에 기본 내장되는 폰트로, Barlow(라틴), SourceHanSans(CJK fallback), D2Coding(모노스페이스), Digital-7(Segment Display 스타일) 등이 있다. 시스템 폰트는 사용자의 OS에 설치된 폰트를 가져와서 테마에 적용할 수 있도록 하는 기능이다. EXIF 프레임에 표시되는 텍스트의 폰트를 사용자가 자유롭게 선택할 수 있어야 하기 때문에, 빌트인 폰트만으로는 부족하다.

**데스크탑에서는 `include_bytes!`로 폰트를 바이너리에 내장**한다.

```rust
pub(crate) const FONT_BARLOW: BuiltInFonts = BuiltInFonts {
    name: "Barlow",
    data: include_bytes!("../../assets/fonts/Barlow-Variable-Remapped.ttf"),
};
```

데스크탑은 실행 파일 하나로 배포하는 것이 편리하기 때문에, 폰트 파일을 컴파일 시점에 바이너리에 포함시킨다. 별도의 폰트 디렉토리 없이 실행 파일만 있으면 바로 동작한다.

반면 **iOS/Android에서는 폰트를 파일 경로로 동적 로딩**한다. 모바일 앱은 바이너리 크기에 민감하고, 앱 번들 내에 리소스 파일로 분리하는 것이 플랫폼 관례이기도 하다. Swift/Kotlin에서 FFI를 통해 폰트 디렉토리 경로를 Rust 코어에 전달하면, Rust 쪽에서 `std::fs::read()`로 필요한 시점에 파일을 읽어 로딩한다.

시스템 폰트는 데스크탑에서만 지원한다. `font-kit` 크레이트의 `SystemSource`를 사용하여 OS에 설치된 폰트 목록을 열거하고, 사용자가 선택한 폰트를 로드한다. 이 작업은 UI를 블로킹하지 않도록 백그라운드 스레드에서 수행되며, `Arc<RwLock<Vec<SystemFont>>>`로 스레드 안전하게 공유한다.

#### font-kit macOS 메모리 폭주 디버깅

시스템 폰트 열거를 구현한 뒤 macOS에서 심각한 문제가 발생했다. 앱 실행 직후 **메모리 사용량이 1.0GB, 피크 1.5GB까지 치솟는** 현상이었다. ([#5](https://github.com/pmnxis/chama-optics/issues/5))

`MallocStackLogging`과 `malloc_history`로 추적한 결과, 원인은 `font-kit`의 macOS 백엔드(`core_text`)에 있었다. `font_kit::SystemSource::all_fonts()`가 시스템 폰트 목록을 열거하면서, **각 폰트의 전체 파일 데이터를 메모리에 읽어들이고 있었다**:

```
435 calls for 2045941700 bytes:  ← 약 2GB
  font_kit::sources::core_text::create_handles_from_core_text_collection
    font_kit::utils::slurp_file    ← 폰트 파일 전체를 메모리로 읽음
      alloc::raw_vec::RawVecInner::try_allocate_in
```

macOS에는 수백 개의 시스템 폰트가 설치되어 있고, CJK 폰트(예: Apple SD Gothic Neo, Hiragino 등)는 개별 파일이 수십 MB에 달한다. `slurp_file`이 이 파일들을 전부 메모리에 올리면서 435개 폰트에 대해 약 2GB를 할당한 것이다. (Windows에서는 동일 코드에서 약 90MB 수준이었다.)

해결 방법은 `font-kit`를 포크하여 `all_fonts()` 호출 시 **폰트 데이터를 읽지 않고 메타데이터(이름, 경로)만 수집**하도록 수정하는 것이었다. 수정 후 메모리 사용량은 **144.9MB**(피크 389.4MB)로 대폭 감소했다.

### 6. LUT 컬러 그레이딩: wagahai-lut의 최적화 철학

라이브러리 이름의 유래는 [wagahaida_L(라플라스 다크니스)](https://x.com/wagahaida_L)의 트윗에서 따왔다.

| LaplusDarknesss | wagahaida_L |
| ---------- | ---------- | 
| {{< x user="LaplusDarknesss" id="1940063453647147386" >}} | {{< x user="wagahaida_L" id="1992881206682423708" >}} |

> 여담으로 v0.2.0에서 준비 중인 체키풍(폴라로이드) 이미지 자동 생성 기능 또한 라플라스 다크니스에게서 아이디어를 얻었다. 요망하게 머리가 좋다고 생각한다.

v0.1.9에서 추가된 LUT 컬러 그레이딩 기능은 직접 개발한 [wagahai-lut](https://github.com/pmnxis/wagahai-lut) ([crates.io](https://crates.io/crates/wagahai-lut)) 라이브러리를 사용한다.

#### CUBE LUT이란?

CUBE LUT(Look-Up Table)은 [Adobe가 정의한 `.cube` 파일 포맷](https://web.archive.org/web/20220220033515/https://wwwimages2.adobe.com/content/dam/acom/en/products/speedgrade/cc/pdfs/cube-lut-specification-1.0.pdf)으로, 색상 변환 정보를 담고 있다. 1D LUT과 3D LUT 두 종류가 있다.

![CUBE LUT 개념](lut_cube_concept.svg)

**1D LUT**은 R, G, B 각 채널을 독립적으로 변환한다. 입력값을 테이블에서 찾아 출력값으로 바꾸는 단순한 구조다. 밝기/대비 조정에 적합하지만, 채널 간 상호작용(예: 빨간색을 파란색으로 바꾸는 것)은 불가능하다. 테이블 크기는 보통 1,024(10-bit)에서 65,536(16-bit)개 엔트리이며, 인접한 두 엔트리 사이의 값은 선형 보간(linear interpolation)으로 계산한다.

**3D LUT**은 RGB 3차원 색상 공간 전체를 매핑한다. 입력 (R, G, B)가 완전히 다른 (R', G', B')로 변환될 수 있어, 영화/사진의 크리에이티브 색감 보정(film look, color grading)에 사용된다. 큐브 내부의 격자점(lattice point)이 알려진 매핑을 정의하고, 격자점 사이의 값은 주변 8개 꼭짓점으로부터 **삼선형 보간**(trilinear interpolation)으로 계산한다. 일반적인 크기는 17³(4,913점), 33³(35,937점), 65³(274,625점)이다.

#### wagahai-lut의 최적화 전략

기존 Rust LUT 라이브러리들은 범용성에 초점이 맞춰져 있었다. wagahai-lut은 "24MP 사진 수십 장을 일괄 처리해도 빨라야 한다"는 Chama Optics의 요구사항에 맞춰, 메모리 레이아웃부터 SIMD 수준까지 최적화했다. 다만 x86_64와 ARM64 양쪽을 모두 지원해야 하므로 직접 어셈블리를 작성하는 대신 [`wide`](https://crates.io/crates/wide) 크레이트를 사용하여 아키텍처에 구애받지 않는 보편적인 벡터 최적화를 택했다.

![wagahai-lut 최적화 전략](lut_optimization.svg)

**1) Structure of Arrays (SoA) 메모리 레이아웃**

일반적인 3D LUT 구현은 `[Rgb, Rgb, Rgb, ...]` 형태의 AoS(Array of Structures) 레이아웃을 사용한다. 하지만 삼선형 보간은 한 번에 한 채널씩 8개 꼭짓점 값을 읽어야 하므로, AoS에서는 캐시 라인에 불필요한 채널 데이터가 함께 로드된다.

wagahai-lut은 3D LUT을 `r: Vec<f32>`, `g: Vec<f32>`, `b: Vec<f32>` 세 개의 분리된 배열로 저장한다. 이 SoA 레이아웃 덕분에 한 채널의 보간에 필요한 8개 값이 메모리상 가까이 위치하여 CPU 캐시 적중률이 높아진다.

**2) SIMD 병렬 처리 (`wide::f32x4`)**

1D LUT 처리에서는 `wide` 크레이트의 `f32x4` SIMD 벡터를 사용하여 R, G, B 세 채널의 선형 보간을 **단일 벡터 연산**으로 수행한다. 4개 레인 중 3개를 R, G, B에 할당하고, 곱셈·덧셈이 한 번의 명령어로 처리된다.

**3) 고정 크기 특수화 (Fixed-Size Specialization)**

1D LUT은 `Bit10(1024)`, `Bit12(4096)`, `Bit14(16384)`, `Bit16(65536)` 등 일반적인 크기에 대해 `Box<[Rgb; SIZE]>` 고정 크기 배열을 사용한다. 컴파일 타임에 크기가 결정되므로 경계 검사(bounds checking)를 건너뛰고 `get_unchecked()`로 직접 접근이 가능하다. 3D LUT도 17³, 33³, 65³ 같은 일반적인 크기를 별도 타입으로 제공한다.

**4) In-Place 처리와 루프 최적화**

`apply_rgb_mut()` / `apply_rgba_mut()` 함수는 이미지 버퍼를 제자리에서(in-place) 수정하여 추가 메모리 할당이 전혀 없다. 핫 루프에서는 도메인 범위의 역수(`inv_domain_range`)를 루프 밖에서 미리 계산하고, 원시 포인터(raw pointer) 연산으로 `get_pixel()`/`put_pixel()` 호출 오버헤드를 제거하며, 바이트 슬라이스를 선형으로 순회하여 CPU 캐시 프리페치를 극대화한다.

#### 벤치마크 결과

M4 Max(Stable Rust) 기준 처리 시간 (JPEG 디코딩/인코딩 시간 포함):

| 해상도 | 1D LUT | 3D LUT |
|:---|---:|---:|
| 1920×1080 (FHD) | 14.39 ms | 19.40 ms |
| 6000×4000 (24MP) | 159.91 ms | 223.15 ms |
| 8144×5424 (44MP) | 294.34 ms | 417.09 ms |

24MP 사진 기준 3D LUT 적용이 약 0.22초로, Chama Optics에서 수십 장의 사진을 일괄 처리할 때 Rayon 병렬화와 결합하면 실용적인 속도를 달성할 수 있다.

### 7. 데스크탑 얼굴 인식: Speed Mode와 슬라이딩 윈도우 알고리즘

데스크탑에서는 ONNX Runtime + InsightFace(det_10g) 모델을 사용한다. 이 모델의 입력 크기는 **고정 640×640 픽셀**이다. 하지만 실제 사진은 24MP(6000×4000) 이상인 경우가 대부분이고, 640×640으로 전체 이미지를 축소하면 인물이 작게 촬영된 단체 사진에서는 얼굴을 놓치게 된다.

이 문제를 해결하기 위해 **Speed Mode에 따른 다단계 슬라이딩 윈도우 알고리즘**을 구현했다.

먼저, 이미지의 짧은 변을 기준으로 **피라미드 최대 깊이 `m_max`** 를 동적 계산한다.

```
m_max = floor(log2(min_side × 0.9 / 640))
```

각 depth에서의 윈도우 크기는 다음과 같다.

```
depth 0 → window = 640 × 2^(m_max)       ← 가장 큰 윈도우
depth 1 → window = 640 × 2^(m_max - 1)
...
depth m_max → window = 640               ← 가장 작은 윈도우
```

Speed Mode는 이 피라미드에서 **실제 탐색하는 깊이 수(`num_levels`)** 를 제한한다.

| 모드 | num_levels | 전체 동작 |
|:---|:---:|:---|
| Fastest | 0 | 전체 이미지 → 640×640 리사이즈 → 단일 추론 |
| Fast | 0 | + 짧은변(`min(W,H)`) 크기 정사각 슬라이딩 윈도우 |
| Normal | min(1, m_max+1) | + 피라미드 depth 0 (가장 큰 윈도우) |
| Slow | min(2, m_max+1) | + 피라미드 depth 0..1 |
| Slowest | min(3, m_max+1) | + 피라미드 depth 0..2 |
| Slowest + ILC | m_max+1 | + 피라미드 전체 (640px 베이스까지) |

> **ILC 카메라 확장**: `Slowest` 모드에서 EXIF Make가 전문 ILC 브랜드(Panasonic, Sony, Canon, Sigma, Fuji, Hasselblad, Nikon, Leica)와 일치하면 `num_levels = m_max + 1`로 확장하여 640px 베이스 윈도우까지 탐색한다. 고해상도 ILC 사진에서 먼 거리의 작은 얼굴까지 빠짐없이 검출하기 위한 확장이다.

아래 표는 카메라별 실제 예시다.

| 카메라 | min_side | m_max | Normal | Slow | Slowest | Slowest+ILC |
|:---|:---:|:---:|:---|:---|:---|:---|
| S5M2 | 4000 | 2 | 2560 | 2560, 1280 | 2560, 1280, 640 | (동일) |
| S1R | 5424 | 2 | 2560 | 2560, 1280 | 2560, 1280, 640 | (동일) |
| A7R5 | 6336 | 3 | 5120 | 5120, 2560 | 5120, 2560, 1280 | +640 |

알고리즘의 흐름은 다음과 같다.

1. **1단계 (공통)**: 전체 이미지를 640×640으로 리사이즈하여 단일 추론. 큰 얼굴은 이 단계에서 잡힌다.
2. **2단계 (Fast 이상)**: 이미지의 짧은 변(`min(width, height)`) 크기의 정사각 슬라이딩 윈도우를 10% 겹침으로 이동시키며 각 윈도우를 640×640으로 축소하여 추론. 비정상적인 종횡비(파노라마 등)에서의 누락을 방지.
3. **3단계 (Normal 이상)**: `640 × 2^(m_max - depth)` 크기의 윈도우를 depth별로 순회. 큰 윈도우부터 작은 윈도우 순으로 점진적 세밀화.
4. **최종**: NMS(Non-Maximum Suppression, IoU 임계값 0.4)로 중복 감지 제거.

각 Speed Mode의 동작을 시각화한 다이어그램 (6000×4000 원본 이미지 기준):

#### Fastest

![Speed Mode: Fastest](speed_mode_fastest.svg)

#### Fast

![Speed Mode: Fast](speed_mode_fast.svg)

#### Normal

![Speed Mode: Normal](speed_mode_normal.svg)

#### Slow

![Speed Mode: Slow](speed_mode_slow.svg)

#### Slowest

![Speed Mode: Slowest](speed_mode_slowest.svg)

아래는 Slowest 모드로 처리한 실제 행사 사진 예시다. 대규모 단체 사진에서 뒷줄 구석의 작은 얼굴까지 빠짐없이 검출하여 모자이크 처리한 결과물이다.

![Slowest 모드 적용 예시](P1167220-OPTICS.webp)

> 2025년 AGF 아마네 카나타 팬 단체 사진회

각 모드의 사용 시나리오:

| 모드 | 평균 소요 시간 | 적합한 상황 |
|:---|:---|:---|
| Fastest | ~0.5초 | 1~2명 인물 사진 |
| Fast | ~0.6초 | 파노라마 등 비정상 종횡비의 1~2명 사진 |
| Normal | ~7초 | 약 10명 정도의 단체 사진 |
| Slow | ~13초 | 40~50명 규모의 단체 사진 |
| Slowest | ~28초 | 50명 이상의 대규모 단체 사진 |

Fastest가 전체를 640×640 하나로 축소하여 ~0.5초만에 끝나는 반면, Slowest는 2560/1280/640 세 단계의 윈도우를 겹치며 탐색하기 때문에 ~28초가 걸린다. 하지만 행사장 단체 사진에서 뒷줄 구석의 작은 얼굴까지 잡아내려면 이 정도의 탐색이 필요하다.

실행 환경(Execution Provider)도 플랫폼별로 최적화되어 있다.

- **macOS**: CoreML Execution Provider 자동 선택 — Apple의 Neural Engine/GPU 가속 활용
- **Windows/Linux**: CPU 또는 OnnxAuto (자동 감지)

macOS에서는 사용자가 CPU를 선택하더라도 내부적으로 CoreML로 자동 전환되어 **Apple Silicon의 Neural Engine을 활용**한다. 이는 CPU 대비 수 배의 성능 향상을 가져온다.

iOS에서는 슬라이딩 윈도우 알고리즘 구조(Fastest/Fast/피라미드 깊이)는 데스크탑과 동일하지만, 추론 엔진으로 InsightFace ONNX 대신 **Apple Vision Framework(`VNDetectFaceRectanglesRequest`)**를 사용한다. Vision이 내부적으로 스케일링을 처리하므로 640×640 리사이즈 단계가 불필요하며, ONNX 모델 없이도 정확한 얼굴 인식이 가능하다.

한편 Android에서는 Google ML Kit (`com.google.mlkit:face-detection`)를 사용하며, 다단계 누적 구조로 동작한다.
- **Pass 1 (모든 속도)**: 전체 이미지를 최대 1024px로 디코드, `PERFORMANCE_MODE_FAST`, `minFaceSize=0.2`
- **Pass 2 (Fast 이상)**: Pass 1의 1024px 비트맵에서 `min(w,h)` 크기 정사각 윈도우를 10% 겹침으로 슬라이딩
- **Pass 3+ (Normal 이상)**: 피라미드 멀티레벨 디코드 — `base = floor(min(minSide/2, maxSide/3) × 1.1)` 기준으로 Normal은 L0, Slow는 L0–L1, Slowest는 L0–L2까지 `PERFORMANCE_MODE_ACCURATE`, `minFaceSize=0.1`로 탐색
- 모든 Pass 결과를 NMS(IoU 0.4)로 병합하여 중복 제거

### 8. iOS 네이티브 통합

iOS 앱은 단순히 Rust 코어를 감싸는 것이 아니라, 플랫폼의 장점을 최대한 활용했다.
- **Apple Vision Framework (VisionKit)** — `VNDetectFaceRectanglesRequest` 기반 슬라이딩 윈도우 얼굴 인식. 데스크탑과 동일한 Fastest/Fast/피라미드 depth 구조를 사용하되, ONNX 모델 없이 Vision이 스케일링과 추론을 직접 처리
- **PhotosUI** — iOS 사진 라이브러리에서 직접 이미지 선택
- **Metal 렌더링** — GPU 가속 이미지 처리
- **iPad 지원** — 넓은 화면에 최적화된 레이아웃

### 9. MPF 및 내장 프리뷰 이미지 추출

JPEG 파일 안에 숨어 있는 서브 이미지를 추출하는 기능은 Chama Optics의 성능에 결정적인 역할을 한다. 이 기능은 [exif-rs PR #58](https://github.com/kamadak/exif-rs/pull/58) ([+1,364줄](https://github.com/kamadak/exif-rs/pull/58), PR #57 기반)로 구현했다.

#### JPEG 안에 숨어 있는 이미지들

JPEG 파일 하나 안에는 실제로 여러 개의 이미지가 들어 있을 수 있다.

![JPEG 파일 구조와 MPF 서브 이미지](jpeg_mpf_structure.svg)

JPEG 안에 내장된 이미지는 크게 세 가지 소스에서 추출할 수 있다.

1. **EXIF IFD(1) 썸네일** — 표준 EXIF 썸네일 (보통 160×120)
2. **APP2 세그먼트 (MPF)** — [CIPA DC-007](https://www.cipa.jp/std/documents/download_e.html?DC-007-Translation-2021-E) 표준에 정의된 Multi-Picture Format. 메인 EOI 이후에 별도의 완전한 JPEG 스트림으로 저장된다.
3. **MakerNote 내부 프리뷰** — 제조사별 비표준 프리뷰 이미지

#### 왜 MPF 프리뷰가 중요한가: 메모리와 성능

사진 목록에서 썸네일을 보여줄 때, 가장 단순한 방법은 **원본 이미지를 로드한 뒤 리사이즈**하는 것이다. 하지만 이건 끔찍하게 비효율적이다.

| 방식 | 메모리 사용 | 처리 시간 |
|:---|:---|:---|
| 원본(24MP) 로드 → 리사이즈 | ~72MB (24M × 3bytes) | 느림 |
| IFD(1) 썸네일 사용 | ~76KB (160×120) | 빠름, 하지만 너무 작아서 흐림 |
| **MPF 프리뷰(~2MP) 사용** | **~8MB** | **빠르고, 시각적으로 충분** |

IFD(1)의 썸네일은 너무 작아서 목록용으로는 괜찮지만 미리보기용으로는 흐릿하다. 원본을 로드하면 24MP 이미지가 메모리에 ~72MB를 차지하고 디코딩 시간도 오래 걸린다. **MPF에 포함된 1~2MP 프리뷰 이미지**는 이 둘 사이의 달콤한 지점이다 — 시각적으로 충분히 선명하면서도 메모리와 CPU 오버헤드가 원본의 1/10 이하다.

특히 Chama Optics처럼 **수십 장의 사진을 동시에 리스트로 보여주고, 테마 미리보기까지 제공**해야 하는 프로그램에서는 이 차이가 결정적이다. 50장의 24MP 사진을 원본으로 로드하면 ~3.6GB, MPF 프리뷰로 로드하면 ~400MB — 약 9배의 차이다.

기존 exif-rs 사용자에게 영향을 주지 않기 위해 `mpf` feature flag로 제공하도록 하였다.

### 10. HEIF/HEIC 디코더: 플랫폼별 전략

최근 JPEG 외에 HEIF(High Efficiency Image Format)로 사진을 저장하는 기기가 늘고 있다. 특히 iOS는 촬영 시 HEIF를 기본으로 사용하며, 사진을 외부로 전달할 때 JPEG로 변환할지 HEIF 그대로 줄지를 OS가 자체적으로 판단한다 — 앱에서 이를 통제하기가 쉽지 않다. 일부 미러리스 카메라(Sony, Canon 등)도 HEIF 촬영을 지원하기 시작했다. 호환성 때문에 JPEG만 쓰는 사용자도 있지만, HEIF로 들어오는 파일을 처리하지 못하면 사진 앱으로서 치명적이다. 문제는 HEIF 디코딩 지원이 플랫폼마다 크게 다르다는 점이다.

Chama Optics는 **가능한 한 OS 네이티브 디코더를 사용하고, 네이티브 지원이 없는 플랫폼에서만 libheif를 사용**하는 전략을 택했다.

![HEIF 디코더 전략](heif_decoder_strategy.svg)

**iOS/macOS** — Apple의 ImageIO 프레임워크가 HEIF를 네이티브로 지원한다. 별도의 외부 라이브러리 없이 OS API만으로 디코딩이 가능하다. iOS에서는 Swift 앱 레이어에서 디코딩한 픽셀 버퍼를 C FFI로 Rust에 전달하고, macOS에서는 Rust가 직접 macOS API 바인딩을 통해 호출한다.

**Android** — API 26(Android 8.0) 이상에서 BitmapFactory와 MediaCodec이 HEIF를 네이티브로 지원한다. Kotlin 앱에서 디코딩한 뒤 JNA를 통해 Rust에 전달한다.

**Windows/Linux** — 네이티브 HEIF 디코더가 없거나 제한적이다. 이 경우 [libheif_rs](https://crates.io/crates/libheif-rs)(libheif의 Rust 바인딩)를 사용한다. C FFI는 libheif_rs가 내부적으로 처리하므로 Rust 코드에서는 안전한 API만 호출하면 된다. libheif는 내부적으로 libde265(HEVC 디코더)와 libaom(AV1/AVIF)을 사용한다.

이 전략의 핵심은 `#[cfg(target_os)]` 조건부 컴파일이다. 네이티브 디코더가 있는 플랫폼에서는 외부 의존성 없이 최적의 성능을 얻고, libheif_rs가 필요한 플랫폼에서만 링킹한다. 결과적으로 macOS 빌드에서는 libheif 관련 코드가 아예 컴파일되지 않는다.

### 11. 테마 파라미터 시스템: Rust → JSON → 플랫폼별 UI

Chama Optics의 테마에는 40개 이상의 설정 파라미터가 있다 — 폰트 weight, 워터마크 위치·투명도, 프레임 스타일, 로고 표시 여부, 색상, 여백 등. 이 파라미터들이 데스크탑과 모바일에서 **동일한 결과**를 보장해야 한다는 것이 핵심 요구사항이었다.

![테마 파라미터 시스템](theme_param_json.svg)

데스크탑(egui)에서는 Rust 구조체를 **직접 참조**하여 UI 위젯을 그린다. `Slider::new(&mut config.font_weight, 100..=900)` 같은 코드로 구조체 필드가 곧 UI 상태가 된다 — JSON 직렬화도, 중간 변환도 없다.

문제는 모바일이다. iOS(SwiftUI)와 Android(Jetpack Compose)는 Rust 구조체에 직접 접근할 수 없다. 40개 이상의 파라미터 각각에 대해 Swift/Kotlin 쪽에서 수동으로 UI를 만들고, FFI로 값을 주고받는 코드를 일일이 작성한다면? 파라미터가 하나 추가될 때마다 Rust, Swift, Kotlin 세 곳을 동시에 수정해야 한다.

이 문제를 `proc_macro`로 해결했다. Rust 구조체 정의에 `#[derive(ThemeParam)]`을 붙이면, 컴파일 타임에 다음이 자동 생성된다.

- **JSON 스키마**: 각 필드의 UI 타입(슬라이더, 토글, enum 선택, 색상 피커 등), 범위, 기본값을 포함하는 JSON
- **FFI 함수**: `get_param_json()`, `set_param()` 등 모바일에서 호출할 수 있는 C ABI 함수
- **역직렬화 로직**: JSON으로 받은 값을 Rust 구조체에 반영하는 코드

모바일 앱은 이 JSON을 파싱하여 **네이티브 UI 요소를 동적으로 생성**한다. `"type": "slider"` → SwiftUI의 `Slider`, Jetpack Compose의 `Slider()`. `"type": "toggle"` → `Toggle` / `Switch`. 사용자가 값을 변경하면 FFI를 통해 Rust 코어에 전달되고, Rust 코어는 동일한 렌더링 파이프라인으로 결과를 반환한다.

결과적으로 Rust 구조체 하나가 **UI 명세이자 데이터 모델이자 직렬화 포맷**의 역할을 동시에 한다. 새 파라미터를 추가할 때 Rust에 필드 하나를 추가하고 어트리뷰트로 UI 힌트를 달면, proc_macro가 JSON 스키마를 갱신하고, 모바일 앱은 다음 빌드에서 자동으로 해당 UI를 표시한다. Swift/Kotlin 코드를 수정할 필요가 없다.

임베디드에서의 `build.rs` 남발과 `const fn` 집착이 이런 식으로 응용되었다. 솔직히 proc_macro로 하는 것이 깔끔한지는 모르겠다 — 본인도 "괴랄하다"고 생각하는 부분이다. 하지만 **40개 이상의 파라미터를 3개 플랫폼에서 수동 동기화하는 것보다는 확실히 낫다.** Rust의 절차형 매크로(procedural macro)에 대해 더 알고 싶다면 [이 글](https://priver.dev/blog/rust/procedural-macros/)이 좋은 참고가 된다.

### 12. 다국어 번역 시스템: YAML 하나로 3개 플랫폼 번역 자동 생성

4개 언어(영어, 한국어, 일본어, 인도네시아어)를 지원하면서 **번역 문자열이 3개 플랫폼에서 동기화되어야 한다.** 번역 키 하나를 추가하거나 문구를 수정할 때마다 iOS의 `.strings`, Android의 `strings.xml`, 데스크탑의 Rust 코드를 각각 손으로 고쳐야 한다면? 결국 빠뜨리거나 어긋나게 된다.

해결 방식은 단순하다. **rust-core의 YAML 파일을 유일한 원본으로 두고, 빌드 시점에 각 플랫폼 포맷으로 자동 변환**하는 것이다.

![i18n 빌드 파이프라인](i18n_build_pipeline.svg)

#### YAML: 번역의 원본

`rust-core/locales/` 디렉토리에 23개의 YAML 파일이 있다. `common.yml`, `gallery.yml`, `theme.yml`, `face_detection.yml` 등 기능 단위로 분리되어 있으며, 총 약 3,900줄이다.

```yaml
# rust-core/locales/gallery.yml
gallery:
  empty_state_title:
    en: "No Images Yet"
    ko: "이미지 없음"
    ja: "画像がありません"
    id: "Belum Ada Gambar"
```

이 구조는 `rust_i18n` 크레이트가 요구하는 형식 그대로다. 데스크탑에서는 `rust_i18n::i18n!("locales")`로 컴파일 타임에 YAML을 임베딩하고, `t!("gallery.empty_state_title")`로 호출한다. 별도의 변환이 필요 없다. `build.rs`에서 `cargo:rerun-if-changed=locales`를 선언해두었으므로, YAML이 수정되면 자동으로 재컴파일된다.

문제는 iOS와 Android다.

#### iOS: generate_ios_strings.sh

iOS는 `NSLocalizedString`과 `.strings` 파일을 사용한다. `generate_ios_strings.sh`는 Python3 + PyYAML로 YAML을 파싱하여, 각 로케일별 `Localizable.strings`를 생성한다.

```bash
# build_ios.sh 에서 자동 호출
./generate_ios_strings.sh
```

YAML의 계층 구조를 점(`.`) 표기법으로 평탄화하여 `.strings` 포맷으로 변환한다.

```
/* Auto-generated from rust-core/locales - DO NOT EDIT */
"gallery.empty_state_title" = "이미지 없음";
"common.actions.save" = "저장";
```

iOS 고유의 요구사항도 있었다. 같은 키라도 iOS에서는 다른 문구를 사용해야 하는 경우가 있다 — 예를 들어 데스크탑에서 "파일 불러오기"라고 쓰는 곳을 iOS에서는 "사진 선택"이라고 써야 자연스럽다. 이를 위해 **`_ios` 접미어 오버라이드**를 구현했다. YAML에서 `import.label_ios`가 정의되어 있으면 iOS 빌드에서는 `import.label` 대신 이 값을 사용한다. 데스크탑과 Android에는 영향을 주지 않는다.

이 스크립트는 `build_ios.sh`에서 Rust 크로스 컴파일 전에 자동으로 호출되므로, YAML을 수정하고 Xcode 빌드를 돌리면 번역이 자동 반영된다.

#### Android: generate_android_strings.sh

Android는 `strings.xml`과 `R.string.*` 리소스 시스템을 사용한다. 핵심적인 차이가 두 가지 있다.

**첫째, 키 포맷이 다르다.** Android 리소스 이름에는 점(`.`)을 사용할 수 없다. YAML의 `gallery.empty_state_title`을 Android에서는 `gallery_empty_state_title`로 변환해야 한다.

```python
def yml_key_to_android_key(yml_key):
    return yml_key.replace('.', '_')
```

**둘째, 로케일 디렉토리 규칙이 다르다.** Android는 인도네시아어를 `id`가 아닌 `in`으로 표기한다 — `values-in/strings.xml`. 이 매핑을 스크립트에서 처리한다.

```bash
ANDROID_LOCALE_MAP["en"]="values"
ANDROID_LOCALE_MAP["ko"]="values-ko"
ANDROID_LOCALE_MAP["ja"]="values-ja"
ANDROID_LOCALE_MAP["id"]="values-in"    # Android uses "in" for Indonesian
```

iOS 스크립트와 또 다른 점은 **diff 기반 동기화**라는 것이다. iOS는 매번 파일을 통째로 덮어쓰지만, Android 스크립트는 기존 `strings.xml`에 이미 있는 키는 건드리지 않고 **누락된 키만 추가**한다. Android 쪽에서 수동으로 관리하는 엔트리(앱 이름 등)를 보존하기 위해서다. `--check` 모드로 실행하면 파일을 수정하지 않고 누락된 번역만 보고한다.

Android에서 이 키들을 실제로 사용할 때는 `ThemeI18n.kt`에서 YAML 점 표기법 키를 `R.string.*` 리소스 ID로 매핑한다.

```kotlin
object ThemeI18n {
    fun translate(context: Context, key: String): String {
        val resourceId = keyToResourceId(key)
        return if (resourceId != 0) context.getString(resourceId) else key
    }
}
```

#### 세 플랫폼의 키 변환 비교

<div style="overflow-x:auto;">
<table>
<thead><tr><th style="white-space:nowrap;">요소</th><th>Desktop (Rust)</th><th>iOS (Swift)</th><th>Android (Kotlin)</th></tr></thead>
<tbody>
<tr><td style="white-space:nowrap;"><b>원본</b></td><td><code>t!("gallery.empty_state_title")</code></td><td><code>NSLocalizedString("gallery.empty_state_title")</code></td><td><code>R.string.gallery_empty_state_title</code></td></tr>
<tr><td style="white-space:nowrap;"><b>키 구분자</b></td><td><code>.</code> (점)</td><td><code>.</code> (점)</td><td><code>_</code> (밑줄)</td></tr>
<tr><td style="white-space:nowrap;"><b>생성 방식</b></td><td>컴파일 타임 임베딩</td><td>빌드 스크립트 자동 생성</td><td>빌드 스크립트 diff 동기화</td></tr>
<tr><td style="white-space:nowrap;"><b>플랫폼 오버라이드</b></td><td>—</td><td><code>_ios</code> 접미어</td><td>—</td></tr>
<tr><td style="white-space:nowrap;"><b>인도네시아어 코드</b></td><td><code>id</code></td><td><code>id</code></td><td><code>in</code></td></tr>
</tbody>
</table>
</div>

이 구조 덕분에 번역을 추가하거나 수정할 때 **YAML 파일 하나만 고치면** 세 플랫폼 모두에 반영된다. 23개 YAML 파일, 4개 언어, 3개 플랫폼을 수동으로 동기화하는 것은 현실적으로 불가능하다 — 내가 생각한 방법은 자동화뿐이었다.

---

## 오픈소스 기여 활동

Chama Optics 개발 과정에서 의존하는 오픈소스 프로젝트들에도 적극적으로 기여했다.

| 프로젝트 | 기여 내용 |
|:---|:---|
| [exif-rs](https://github.com/kamadak/exif-rs) | MakerNote 파싱 — 10개 제조사 지원 ([PR #57](https://github.com/kamadak/exif-rs/pull/57), +5,946 lines) |
| [exif-rs](https://github.com/kamadak/exif-rs) | MPF 및 내장 프리뷰 이미지 추출 ([PR #58](https://github.com/kamadak/exif-rs/pull/58), +1,364 lines) |
| [exif-rs](https://github.com/kamadak/exif-rs) | TIFF field 접근 개선 ([PR #51](https://github.com/kamadak/exif-rs/pull/51), approved) |
| [font-kit](https://github.com/servo/font-kit) | macOS 시스템 폰트 열거 시 메모리 폭주 수정 ([PR #271](https://github.com/servo/font-kit/pull/271)) |
| [egui](https://github.com/emilk/egui) | variable font 로드 시 main weight 설정 기능 개선 ([PR #7790](https://github.com/emilk/egui/pull/7790), approved) |
| [wagahai-lut](https://github.com/pmnxis/wagahai-lut) | 1D/3D LUT 컬러 그레이딩 라이브러리 ([crates.io](https://crates.io/crates/wagahai-lut)) |

---

## 릴리즈 요약

| 버전 | 날짜 | 주요 변경사항 |
|:---|:---|:---|
| v0.1.0 | 2025-10-19 | 첫 프리릴리즈, macOS/Windows, Film 테마 |
| v0.1.1 | 2025-10-19 | 일본어 번역, 일괄 저장, 접두어/접미어 |
| v0.1.2 | 2025-10-27 | Glow 효과, Film Date/Glow 테마 |
| v0.1.3 | 2025-11-03 | 워터마크(9곳 위치), 폰트 선택 |
| v0.1.4\~5 | 2025-11-05\~12 | Just Frame, Strap 테마, 카메라 로고 |
| v0.1.6 | 2025-11-24 | Monitor, Lightroom 테마, Longside 스케일 |
| v0.1.7 | 2025-12-19 | One/Two Line, Shot On 테마, CJK 수정, PhotoStyle |
| v0.1.8 | 2025-12-27 | 탭 UI, 그룹화, 테마 미리보기, 멀티코어 |
| v0.1.9 | 2026-02-04 | 얼굴 인식, LUT 컬러 그레이딩, **iOS TestFlight 첫 배포** |

---

## AI와 함께하는 프로그래밍 (바이브 코딩?)

처음에는 AI 코딩에 대해서 회의적이었다.

그래서 데스크탑 버전의 대다수는 아직도 내 손코딩 + `cargo fmt/clippy/check`에 의존하고 있다. Rust Embedded에서의 습관인 `const` 남발을 데스크탑에서도 적용하려는 내 의도를 AI(Claude)는 여전히 제대로 파악하지 못하고 있다.

하지만 모바일용을 개발하면서 **혼자서 이걸 다 하는 건 미친 짓**이라고 생각했다. 기존 데스크탑 버전과 동일한 결과를 모바일에서 제공하는 것이 우선 사항이었고, 네이티브 API와 Rust FFI를 직접 호출하는 형태가 많을 것으로 예상되었다.

그러던 참에 친구 결혼식 청첩장 모임을 가면서, 같이 차를 타고 있던 친구가 "**Flutter에서 iOS 26에서 현재 Liquid UI가 제대로 안 돼**"라는 말을 했다. 개발하더라도 네이티브로만 개발해야겠다는 생각이 굳어졌고, 동시에 AI에게 모바일용 코드를 맡기기로 결정했다.

결과적으로 **Rust 코어는 내가 직접**, **모바일 UI(SwiftUI/Jetpack Compose)와 FFI 브리지는 AI와 함께** 작성하는 분업 체제가 만들어졌다. Rust 쪽은 내가 의도한 패턴과 스타일이 있어서 AI가 잘 맞추지 못하지만, Swift/Kotlin처럼 내가 잘 모르는 언어로 플랫폼 네이티브 코드를 작성하는 데에는 AI가 큰 도움이 되었다.

이 경험을 하고 나니, Flutter 같은 크로스플랫폼 프레임워크의 위치에 대해 생각하게 되었다. 물론 Flutter나 React Native가 해결하는 문제 — 하나의 코드베이스로 여러 플랫폼을 커버하는 것 — 는 여전히 유효하다. 하지만 AI가 각 플랫폼의 네이티브 코드를 충분히 잘 작성해줄 수 있는 시대가 되면, "네이티브를 몰라서 크로스플랫폼을 선택한다"는 동기는 점점 약해질 수 있다. 모바일 개발을 전혀 몰랐던 내가 AI의 도움만으로 SwiftUI와 Jetpack Compose를 각각 네이티브로 작성할 수 있었다는 사실이, 그 가능성을 보여주는 하나의 사례가 아닐까 싶다.

---

## 앞으로의 계획

v0.1.9를 기점으로 데스크탑 단독 릴리즈는 종료하고, v0.2.0부터는 iOS, Android 모바일 앱과 함께 릴리즈할 예정이다. 추가적인 기능보다는 간간이 안정화와 테마 추가에 집중할 생각이다.

당장의 목표는 **2026년 3월 홀로라이브 엑스포/페스티벌**에서 실전 투입하는 것이다. 행사장에서 미러리스로 사진을 찍고, iPhone에서 바로 프레임을 입히고, 얼굴을 자동으로 모자이크 처리해서 SNS에 올리는 — 카메라 사용자든 일반 스마트폰 사용자든, 각자의 환경에서 편하게 쓸 수 있는 워크플로우를 제공하는 것이 방향이다.

사이드 프로젝트로서 Chama Optics는 "사진을 찍는 사람이 사진을 더 잘 보여줄 수 있게 돕는 도구"를 목표로, 좀 더 편한 워크플로우를 제공해 나갈 것이다. 그리고 이번 개발에서 쌓은 절차형 매크로와 최적화 경험을 바탕으로, 다시 Rust Embedded 쪽에도 좀 더 신경 써볼 예정이다.

---

## 참고 문헌 및 인용

#### 표준 문서
- CIPA DC-008 — [Exchangeable image file format (Exif) Version 3.0](https://www.cipa.jp/std/documents/download_e.html?DC-008-Translation-2023-E)
- CIPA DC-007 — [Multi-Picture Format (MPF)](https://www.cipa.jp/std/documents/download_e.html?DC-007-Translation-2021-E)
- Adobe — [Adobe Cube LUT Spec 1.0](https://web.archive.org/web/20220220033515/https://wwwimages2.adobe.com/content/dam/acom/en/products/speedgrade/cc/pdfs/cube-lut-specification-1.0.pdf)

#### 라이브러리 및 프레임워크
- [exif-rs](https://github.com/kamadak/exif-rs) — Rust EXIF 파싱 라이브러리
- [egui](https://github.com/emilk/egui) — Rust 즉시 모드 GUI 프레임워크
- [font-kit](https://github.com/servo/font-kit) — 크로스플랫폼 폰트 로딩 라이브러리
- [wagahai-lut](https://github.com/pmnxis/wagahai-lut) ([crates.io](https://crates.io/crates/wagahai-lut)) — 1D/3D LUT 컬러 그레이딩 라이브러리
- [libheif-rs](https://crates.io/crates/libheif-rs) — libheif Rust 바인딩
- [wide](https://crates.io/crates/wide) — 크로스플랫폼 SIMD 벡터 크레이트
- [ONNX Runtime](https://onnxruntime.ai/) — 크로스플랫폼 ML 추론 엔진
- [InsightFace SCRFD](https://github.com/deepinsight/insightface/tree/master/detection/scrfd) — 얼굴 감지 모델 (det_10g)
- [exif-frame](https://github.com/jeonghyeon-net/exif-frame) — EXIF 프레임 웹 도구 (Chama Optics의 초기 참고)

#### 참고 자료
- [Rust Procedural Macros](https://priver.dev/blog/rust/procedural-macros/) — 절차형 매크로 참고
- [ExifTool TagNames](https://exiftool.org/TagNames/) — 제조사별 MakerNote 태그 참고
- [Exiv2 MakerNote Documentation](https://exiv2.org/makernote.html) — MakerNote 구조 및 제조사별 포맷 참고

---

## Special Thanks

- [SkuldNorniern](https://github.com/SkuldNorniern) — 디버깅 및 얼굴 인식 관련 도움
- [miniex](https://github.com/miniex) — 폰트 시스템 디버깅 및 얼굴 인식 관련 도움
- [jcm7612](https://github.com/jcm7612) — 디버깅 및 피드백
- [shiemika324](https://x.com/shiemika324) — 일러스트 및 아이콘 일러스트 제공
