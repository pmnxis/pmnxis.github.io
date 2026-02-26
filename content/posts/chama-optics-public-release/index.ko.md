---
title: "Chama Optics 배포"
date: 2026-02-17T00:00:00+09:00
draft: false
categories: ["Chama Optics"]
tags: ["Hololive", "HAACHAMA", "Photography"]
description: "하아톤이 만든 사진 앱, Chama Optics — 행사 사진부터 오시카츠까지"
summary: "Chama Optics를 릴리즈 합니다. 사진을 좋아하는 여행하는 VTuber 아카이 하아토(赤井はあと)에게서 영감을 받아 만든, 미러리스·DSLR 카메라 사용자를 위한 사진 후처리 앱"
showHero: true
heroStyle: "background"
---

## Chama Optics

> 🌐 [English Release Note](/en/posts/chama-optics-public-release/) | [日本語リリースノート](/ja/posts/chama-optics-public-release/)

<p style="text-align:center;"><img src="https://github.com/pmnxis/chama-optics/raw/master/assets/mac-icon.png" alt="Chama Optics" style="width:50%; display:block; margin:0 auto;"></p>

> 행사장에서 찍은 사진, 그 자리에서 보정/프레임 입히고 모자이크까지.

**Chama Optics**는 사진을 좋아하는 여행하는 VTuber [아카이 하아토(赤井はあと)](https://www.youtube.com/@AkaiHaato)에게서 영감을 받아 만든, 미러리스·DSLR 카메라 사용자를 위한 사진 후처리 앱입니다. 촬영 정보 프레임, 얼굴 자동 모자이크, 색감 보정, 체키 (폴라로이드풍 프레임) 등의 기능을 제공합니다.

- **행사·여행·스트릿 촬영** —<br>결혼식, VTuber 오프라인 행사, 거리나 관광지 등 다른 사람 얼굴 모자이크가 필요할 때
- **팬 사진 해시태그 참가자** —<br>[#推し活はあとん日記](https://x.com/hashtag/%E6%8E%A8%E3%81%97%E6%B4%BB%E3%81%AF%E3%81%82%E3%81%A8%E3%82%93%E6%97%A5%E8%A8%98), [#Towavel](https://x.com/hashtag/Towavel) 같은 태그로 사진을 올릴 때
- **DSLR/미러리스 카메라 사용자·장비 리뷰어** —<br>촬영 정보(기종, 렌즈, 조리개 등)를 프레임에 넣어서 공유할 때

<p style="text-align:right; font-size:0.85em; opacity:0.7;">기술적인 개발 과정이 궁금하다면 <a href="/ko/posts/chama-optics-dev-story/">개발기</a>를 참고 부탁드립니다.</p>

<p style="text-align:center; margin:1.5em 0 0.5em;"><a href="#다운로드" style="font-size:1.2em; font-weight:600;">⬇ 다운로드로 바로 가기</a></p>

<style>
.chama-gallery{display:flex;gap:4px;padding:8px 0;overflow:hidden}
.chama-gallery>div{min-width:0}
.chama-gallery img{max-height:160px;max-width:100%;height:auto;width:auto;border-radius:8px;display:block;transition:all .2s}
.chama-gallery .cap{display:none}
.chama-feat:has(details[open]) .chama-gallery{overflow-x:auto;-webkit-overflow-scrolling:touch;scroll-snap-type:x mandatory;gap:12px}
.chama-feat:has(details[open]) .chama-gallery>div{flex:0 0 auto;scroll-snap-align:start}
.chama-feat:has(details[open]) .chama-gallery img{max-height:420px;max-width:none}
.chama-feat:has(details[open]) .chama-gallery .cap{display:block;font-size:.85em;opacity:.7;margin-top:4px;text-align:center}
</style>

## 주요 기능

### 촬영 정보(EXIF) 프레임

<div class="chama-feat">
<div class="chama-gallery">
<div>
<img src="/img/chama-optics/ss-ko-exif.webp" alt="EXIF 편집 (iPhone)">
<div class="cap">iPhone</div>
</div>
<div>
<img src="/img/chama-optics/ss-ko-exif-ipad.webp" alt="EXIF 편집 (iPad)">
<div class="cap">iPad</div>
</div>
<div>
<img src="https://github.com/user-attachments/assets/a6bf0e51-d3b1-4779-9d65-080b225958f4" alt="Film Date">
<div class="cap">Film Date</div>
</div>
<div>
<img src="https://github.com/user-attachments/assets/039ab49f-85b1-414b-95e3-2da166cea27f" alt="Strap">
<div class="cap">Strap</div>
</div>
<div>
<img src="https://github.com/user-attachments/assets/e92b81a0-4465-4dad-9097-7e8b4814fc15" alt="Monitor">
<div class="cap">Monitor</div>
</div>
<div>
<img src="https://github.com/user-attachments/assets/ce1022cd-aea3-4260-9d5f-5e15997388de" alt="Lightroom">
<div class="cap">Lightroom</div>
</div>
<div>
<img src="https://github.com/user-attachments/assets/337337f3-7c17-467a-b965-06481cba98c8" alt="One Line">
<div class="cap">One Line</div>
</div>
<div>
<img src="https://github.com/user-attachments/assets/a9ba4540-6540-418c-8e21-4f9961d7bff7" alt="Shot On Two Line">
<div class="cap">Shot On Two Line</div>
</div>
<div>
<img src="https://github.com/user-attachments/assets/28016fd2-2d4d-4043-88e1-c29f7577a32a" alt="Nikon PhotoStyle">
<div class="cap">Nikon PhotoStyle</div>
</div>
<div>
<img src="https://github.com/user-attachments/assets/62219e6a-c6cb-49ac-9803-cda29321b998" alt="Lumix Photo Style + LUT">
<div class="cap">Lumix Photo Style + LUT</div>
</div>
</div>
<details>
<summary>자세히 보기</summary>

카메라로 찍은 사진에는 카메라 기종, 렌즈, 셔터스피드, 조리개, ISO 같은 촬영 정보가 들어 있습니다. Chama Optics는 이 정보를 자동으로 읽어서 다양한 테마의 프레임에 넣어줍니다.

카메라 제조사 로고(Canon, Nikon, Sony, Lumix 등)도 촬영 정보에서 자동으로 인식해서 프레임에 넣어줍니다. Lumix의 경우 적용된 Photo Style 이름이나 LUT 파일명까지 표시할 수 있습니다.

</details>
</div>

### 얼굴 자동 인식 + 모자이크/스티커

<div class="chama-feat">
<div class="chama-gallery">
<div>
<img src="/img/chama-optics/ss-ko-face.webp" alt="얼굴 감지 (iPhone)">
<div class="cap">iPhone</div>
</div>
<div>
<img src="/img/chama-optics/ss-ko-face-ipad.webp" alt="얼굴 감지 (iPad)">
<div class="cap">iPad</div>
</div>
<div>
<img src="/img/chama-optics/IMG_2194-OPTICS.webp" alt="얼굴 감지 결과 (결혼식)">
<div class="cap">결과 (결혼식)</div>
</div>
<div>
<img src="/img/chama-optics/IMG_2195-OPTICS.webp" alt="얼굴 감지 결과 (행사)">
<div class="cap">결과 (행사)</div>
</div>
</div>

> ‼️ 단체사진이나 사람이 20명 이상인 경우, <strong>"설정" → "얼굴 감지" → "감지 속도" → "Slowest"</strong>로 설정해주세요.

<details>
<summary>자세히 보기</summary>

행사장이나 관광지에서 다른 사람의 얼굴이 찍혔을 때, 얼굴을 자동으로 감지하여 모자이크 또는 스티커로 가려줍니다.

- 모자이크(Mosaic), 스트로크(Stroke), 스티커(Sticker), 모자이크+스트로크(Mosaic+Stroke) 등 다양한 방식으로 가릴 수 있습니다
- 감지된 얼굴을 개별적으로 선택하여 효과를 적용하거나 해제할 수 있습니다
- 설정에서 사용자 정의 스티커를 추가하면, 체키 탭의 주사위 굴리기에도 자동 포함됩니다

</details>
</div>

### 컬러 그레이딩 (LUT)

<div class="chama-feat">
<div class="chama-gallery">
<div>
<img src="/img/chama-optics/ss-ko-lut.webp" alt="LUT 선택 (iPhone)">
<div class="cap">iPhone</div>
</div>
<div>
<img src="/img/chama-optics/ss-ko-lut-ipad.webp" alt="LUT 선택 (iPad)">
<div class="cap">iPad</div>
</div>
<div>
<img src="/img/chama-optics/IMG_2193-OPTICS.webp" alt="LUT 적용 결과">
<div class="cap">적용 결과 (사진)</div>
</div>
<div>
<img src="https://github.com/user-attachments/assets/554f96b2-217a-4603-93f5-1df41309f77e" alt="컬러 그레이딩 UI">
<div class="cap">컬러 그레이딩 UI (데스크탑)</div>
</div>
<div>
<img src="https://github.com/user-attachments/assets/4d49174c-2624-4b3c-a8a1-fc2b56d357c1" alt="적용 결과">
<div class="cap">적용 결과 (데스크탑)</div>
</div>
</div>
<details>
<summary>자세히 보기</summary>

영화나 사진에서 특정 색감을 입히는 데 사용하는 **LUT(Look-Up Table)** 파일을 적용할 수 있습니다. `.cube` 파일을 불러오면 사진에 영화 같은 색감을 입힐 수 있습니다.

</details>
</div>

### 체키 (폴라로이드풍 프레임)

<div class="chama-feat">
<div class="chama-gallery">
<div>
<img src="/img/chama-optics/ss-ko-cheki.webp" alt="체키 모드 (iPhone)">
<div class="cap">체키 모드 (iPhone)</div>
</div>
<div>
<img src="/img/chama-optics/ss-ko-cheki-ipad.webp" alt="체키 모드 (iPad)">
<div class="cap">체키 모드 (iPad)</div>
</div>
<div>
<img src="/img/chama-optics/IMG_2191-OPTICS.webp" alt="체키 결과 (행사)">
<div class="cap">결과 (행사)</div>
</div>
<div>
<img src="/img/chama-optics/ss-ko-cheki-text.webp" alt="텍스트 설정 (iPhone)">
<div class="cap">텍스트 설정 (iPhone)</div>
</div>
<div>
<img src="/img/chama-optics/ss-ko-cheki-text-ipad.webp" alt="텍스트 설정 (iPad)">
<div class="cap">텍스트 설정 (iPad)</div>
</div>
<div>
<img src="/img/chama-optics/ss-cheki-sample.webp" alt="체키 출력 결과">
<div class="cap">출력 결과</div>
</div>
<div>
<img src="/img/chama-optics/IMG_2192-OPTICS.webp" alt="체키 결과 (일상)">
<div class="cap">결과 (일상)</div>
</div>
</div>
<details>
<summary>자세히 보기</summary>

폴라로이드 체키풍의 프레임을 자동으로 생성합니다. 랜덤 캐릭터 스티커 배치, 날짜/서명 텍스트, 테두리 설정 등을 커스터마이즈할 수 있습니다.

- 체키 활성화 시 폴라로이드풍 하단 여백이 자동 생성됩니다
- 주사위 굴리기로 랜덤 캐릭터 스티커를 이미지에 배치할 수 있습니다
- 텍스트/서명, 폰트, 색상, 위치(TL/TC/TR/BL/BC/BR) 등을 자유롭게 설정할 수 있습니다

</details>
</div>

### iOS / Android / Desktop

<div class="chama-feat">
<div class="chama-gallery">
<div>
<img src="/img/chama-optics/ios-preview.webp" alt="iOS 미리보기">
<div class="cap">iOS</div>
</div>
<div>
<img src="/img/chama-optics/android-preview.webp" alt="Android 미리보기">
<div class="cap">Android</div>
</div>
<div>
<img src="/img/chama-optics/macos.webp" alt="Desktop (macOS)">
<div class="cap">Desktop</div>
</div>
</div>
<details>
<summary>자세히 보기</summary>

App Store에서 배포 중입니다. 데스크탑과 동일한 기능을 iPhone에서 사용할 수 있습니다.

아키텍처 차별이 없도록 macOS, Linux, Windows, FreeBSD, iOS, Android, Web (wasm32) 를 지원합니다.

</details>
</div>

---


## 다운로드

### Desktop

<div id="chama-downloads" style="display: grid; grid-template-columns: 1fr; gap: 12px; margin: 1.5em 0;">
<a id="dl-macos" href="https://github.com/pmnxis/chama-optics/releases/latest" target="_blank" rel="noopener" style="display:flex; align-items:center; gap:16px; padding:14px 20px; border-radius:8px; background:var(--tw-prose-pre-bg, #1e293b); color:var(--tw-prose-body); text-decoration:none; border:1px solid rgba(128,128,128,0.2); transition: background 0.15s;"><span style="display:flex; align-items:center; gap:4px; flex-shrink:0;"><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 384 512" style="width:20px; height:20px;"><path fill="currentColor" d="M318.7 268.7c-.2-36.7 16.4-64.4 50-84.8-18.8-26.9-47.2-41.7-84.7-44.6-35.5-2.8-74.3 20.7-88.5 20.7-15 0-49.4-19.7-76.4-19.7C63.3 141.2 4 184.8 4 273.5q0 39.3 14.4 81.2c12.8 36.7 59 126.7 107.2 125.2 25.2-.6 43-17.9 75.8-17.9 31.8 0 48.3 17.9 76.4 17.9 48.6-.7 90.4-82.5 102.6-119.3-65.2-30.7-61.7-90-61.7-91.9zm-56.6-164.2c27.3-32.4 24.8-61.9 24-72.5-24.1 1.4-52 16.4-67.9 34.9-17.5 19.8-27.8 44.3-25.6 71.9 26.1 2 49.9-11.4 69.5-34.3z"/></svg><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 512" style="width:24px; height:24px;"><path fill="currentColor" d="M64 96c0-35.3 28.7-64 64-64h384c35.3 0 64 28.7 64 64v256c0 35.3-28.7 64-64 64H128c-35.3 0-64-28.7-64-64V96zM0 403.2C0 392.6 8.6 384 19.2 384h601.6c10.6 0 19.2 8.6 19.2 19.2 0 42.4-34.4 76.8-76.8 76.8H76.8C34.4 480 0 445.6 0 403.2zM512 96H128v256h384V96z"/></svg></span><span style="display:inline-block"><span style="font-weight:600;">macOS (Apple Silicon)</span><br><span id="dl-macos-ver" style="font-size:0.8em; opacity:0.6;">Loading...</span></span></a>
<a id="dl-windows" href="https://github.com/pmnxis/chama-optics/releases/latest" target="_blank" rel="noopener" style="display:flex; align-items:center; gap:16px; padding:14px 20px; border-radius:8px; background:var(--tw-prose-pre-bg, #1e293b); color:var(--tw-prose-body); text-decoration:none; border:1px solid rgba(128,128,128,0.2); transition: background 0.15s;"><span style="display:flex; align-items:center; gap:4px; flex-shrink:0;"><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512" style="width:20px; height:20px;"><path fill="currentColor" d="M0 32h214.6v214.6H0V32zm233.4 0H448v214.6H233.4V32zM0 265.4h214.6V480H0V265.4zm233.4 0H448V480H233.4V265.4z"/></svg><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 512" style="width:24px; height:24px;"><path fill="currentColor" d="M64 96c0-35.3 28.7-64 64-64h384c35.3 0 64 28.7 64 64v256c0 35.3-28.7 64-64 64H128c-35.3 0-64-28.7-64-64V96zM0 403.2C0 392.6 8.6 384 19.2 384h601.6c10.6 0 19.2 8.6 19.2 19.2 0 42.4-34.4 76.8-76.8 76.8H76.8C34.4 480 0 445.6 0 403.2zM512 96H128v256h384V96z"/></svg></span><span style="display:inline-block"><span style="font-weight:600;">Windows (x86_64)</span><br><span id="dl-windows-ver" style="font-size:0.8em; opacity:0.6;">Loading...</span></span></a>
<style>.dl-tri{margin-left:auto; transition:transform 0.2s; flex-shrink:0;}details[open] .dl-tri{transform:rotate(90deg);}.dl-copy-btn{flex-shrink:0; display:flex; align-items:center; justify-content:center; width:36px; border:1px solid rgba(128,128,128,0.3); border-radius:4px; background:rgba(0,0,0,0.2); color:inherit; cursor:pointer;}.dl-copy-btn:hover{background:rgba(0,0,0,0.4);}</style>
<details style="border-radius:8px; background:var(--tw-prose-pre-bg, #1e293b); border:1px solid rgba(128,128,128,0.2);"><summary style="display:flex; align-items:center; gap:16px; padding:14px 20px; cursor:pointer; color:var(--tw-prose-body); list-style:none;"><span style="display:flex; align-items:center; gap:4px; flex-shrink:0;"><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512" style="width:20px; height:20px;"><path fill="currentColor" d="M220.8 123.3c1 .5 1.8 1.7 3 1.7 1.1 0 2.8-.4 2.9-1.5.2-1.4-1.9-2.3-3.2-2.9-1.7-.7-3.9-1-5.5-.1-.4.2-.8.7-.6 1.1.3 1.3 2.3 1.1 3.4 1.7zm-21.9 1.7c1.2 0 2-1.2 3-1.7 1.1-.6 3.1-.4 3.5-1.6.2-.4-.2-.9-.6-1.1-1.6-.9-3.8-.6-5.5.1-1.3.6-3.4 1.5-3.2 2.9.1 1 1.8 1.5 2.8 1.4zM420 403.8c-3.6-4-5.3-11.6-7.2-19.7-1.8-8.1-3.9-16.8-10.5-22.4-1.3-1.1-2.6-2.1-4-2.9-1.3-.8-2.7-1.5-4.1-2 9.2-27.3 5.6-54.5-3.7-79.1-11.4-30.1-31.3-56.4-46.5-74.4-17.1-21.5-33.7-41.9-33.4-72C311.1 85.4 315.7.1 234.8 0 132.4-.2 158 103.4 156.9 135.2c-1.7 23.4-6.4 41.8-22.5 64.7-18.9 22.5-45.5 58.8-58.1 96.7-6 17.9-8.8 36.1-6.2 53.3-6.5 5.8-11.4 14.7-16.6 20.2-4.2 4.3-10.3 5.9-17 8.3s-14 6-18.5 14.5c-2.1 3.9-2.8 8.1-2.8 12.4 0 3.9.6 7.9 1.2 11.8 1.2 8.1 2.6 15.7.8 20.8-5.2 14.4-5.9 24.4-2.2 31.7 3.8 7.3 11.4 10.5 20.1 12.3 17.3 3.6 40.8 2.7 59.3 12.5 19.8 10.4 39.9 14.1 55.9 10.4 11.6-2.6 21.1-9.6 25.9-20.2 12.5-.1 26.3-5.4 48.3-6.6 14.9-1.2 33.6 5.3 55.1 4.1.6 2.3 1.4 4.6 2.5 6.7v.1c8.3 16.7 23.8 24.3 40.3 23 16.6-1.3 34.1-11 48.3-27.9 13.6-16.4 36-23.2 50.9-32.2 7.4-4.5 13.4-10.1 13.9-18.3.4-8.2-4.4-17.3-15.5-29.7z"/></svg><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 576 512" style="width:24px; height:24px;"><path fill="currentColor" d="M64 0C28.7 0 0 28.7 0 64v288c0 35.3 28.7 64 64 64h176l-10.7 32H160c-17.7 0-32 14.3-32 32s14.3 32 32 32h256c17.7 0 32-14.3 32-32s-14.3-32-32-32h-69.3L336 416h176c35.3 0 64-28.7 64-64V64c0-35.3-28.7-64-64-64H64zM512 64v288H64V64h448z"/></svg></span><span style="display:inline-block"><span style="font-weight:600;">Linux</span><br><span id="dl-linux-ver" style="font-size:0.8em; opacity:0.6;">Debian · Ubuntu · Fedora · Rocky · Arch</span></span><span class="dl-tri">&#9654;</span></summary><div style="padding:0 20px 14px;"><div style="display:flex; gap:6px; align-items:stretch;"><code style="flex:1; display:block; padding:8px 10px; background:rgba(0,0,0,0.3); border-radius:4px; font-size:0.78em; word-break:break-all; user-select:all;">curl -sSf https://raw.githubusercontent.com/pmnxis/chama-optics/master/quick-install-linux.sh | bash</code><button class="dl-copy-btn" onclick="navigator.clipboard.writeText('curl -sSf https://raw.githubusercontent.com/pmnxis/chama-optics/master/quick-install-linux.sh | bash');this.innerHTML='✓';var b=this;setTimeout(function(){b.innerHTML='<svg xmlns=&quot;http://www.w3.org/2000/svg&quot; viewBox=&quot;0 0 448 512&quot; style=&quot;width:14px;height:14px&quot;><path fill=&quot;currentColor&quot; d=&quot;M208 0H332.1c12.7 0 24.9 5.1 33.9 14.1l67.9 67.9c9 9 14.1 21.2 14.1 33.9V336c0 26.5-21.5 48-48 48H208c-26.5 0-48-21.5-48-48V48c0-26.5 21.5-48 48-48zM48 128h80v64H64v256h192v-32h64v48c0 26.5-21.5 48-48 48H48c-26.5 0-48-21.5-48-48V176c0-26.5 21.5-48 48-48z&quot;/></svg>';},1500)" title="복사"><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512" style="width:14px;height:14px"><path fill="currentColor" d="M208 0H332.1c12.7 0 24.9 5.1 33.9 14.1l67.9 67.9c9 9 14.1 21.2 14.1 33.9V336c0 26.5-21.5 48-48 48H208c-26.5 0-48-21.5-48-48V48c0-26.5 21.5-48 48-48zM48 128h80v64H64v256h192v-32h64v48c0 26.5-21.5 48-48 48H48c-26.5 0-48-21.5-48-48V176c0-26.5 21.5-48 48-48z"/></svg></button></div><div style="display:flex; justify-content:space-between; align-items:center; margin-top:4px; font-size:0.75em; opacity:0.5;"><span>터미널에서 실행</span><span>자세한 내용: <a href="https://github.com/pmnxis/chama-optics/blob/master/README_LINUX.md" target="_blank" rel="noopener" style="color:inherit; text-decoration:underline;">README_LINUX.md</a></span></div><div style="margin-top:6px; font-size:0.75em; opacity:0.6;">💡 glibc &lt; 2.38 시 ONNX Runtime 자동 다운그레이드</div></div></details>
<details style="border-radius:8px; background:var(--tw-prose-pre-bg, #1e293b); border:1px solid rgba(128,128,128,0.2);"><summary style="display:flex; align-items:center; gap:16px; padding:14px 20px; cursor:pointer; color:var(--tw-prose-body); list-style:none;"><span style="display:flex; align-items:center; gap:4px; flex-shrink:0;"><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512" style="width:20px; height:20px;"><path fill="currentColor" d="M303.7 96.2c11.1-11.1 115.5-77 139.2-53.2 23.7 23.7-42.1 128.1-53.2 139.2-11.1 11.1-39.4.9-63.1-22.9-23.8-23.7-34.1-52-22.9-63.1zM109.9 68.1C73.6 47.5 22 24.6 5.6 41.1c-16.6 16.6 7.1 69.4 27.9 105.7 18.5-32.2 44.8-59.3 76.4-78.7zM406.7 174c3.3 11.3 2.7 20.7-2.7 26.1-20.3 20.3-87.5-27-109.3-70.1-18-32.3-11.1-53.4 14.9-48.7 5.7-3.6 12.3-7.6 19.6-11.6-29.8-15.5-63.6-24.3-99.5-24.3-119.1 0-215.6 96.5-215.6 215.6 0 119 96.5 215.6 215.6 215.6S445.3 380.1 445.3 261c0-38.4-10.1-74.5-27.7-105.8-3.9 7-7.6 13.3-10.9 18.8z"/></svg><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 576 512" style="width:24px; height:24px;"><path fill="currentColor" d="M64 0C28.7 0 0 28.7 0 64v288c0 35.3 28.7 64 64 64h176l-10.7 32H160c-17.7 0-32 14.3-32 32s14.3 32 32 32h256c17.7 0 32-14.3 32-32s-14.3-32-32-32h-69.3L336 416h176c35.3 0 64-28.7 64-64V64c0-35.3-28.7-64-64-64H64zM512 64v288H64V64h448z"/></svg></span><span style="display:inline-block"><span style="font-weight:600;">FreeBSD</span><br><span id="dl-freebsd-ver" style="font-size:0.8em; opacity:0.6;">FreeBSD 13 · 14 · ⚠ 얼굴 인식 기능 미지원</span></span><span class="dl-tri">&#9654;</span></summary><div style="padding:0 20px 14px;"><div style="display:flex; gap:6px; align-items:stretch;"><code style="flex:1; display:block; padding:8px 10px; background:rgba(0,0,0,0.3); border-radius:4px; font-size:0.78em; word-break:break-all; user-select:all;">fetch -o - https://raw.githubusercontent.com/pmnxis/chama-optics/master/quick-install-freebsd.sh | sh</code><button class="dl-copy-btn" onclick="navigator.clipboard.writeText('fetch -o - https://raw.githubusercontent.com/pmnxis/chama-optics/master/quick-install-freebsd.sh | sh');this.innerHTML='✓';var b=this;setTimeout(function(){b.innerHTML='<svg xmlns=&quot;http://www.w3.org/2000/svg&quot; viewBox=&quot;0 0 448 512&quot; style=&quot;width:14px;height:14px&quot;><path fill=&quot;currentColor&quot; d=&quot;M208 0H332.1c12.7 0 24.9 5.1 33.9 14.1l67.9 67.9c9 9 14.1 21.2 14.1 33.9V336c0 26.5-21.5 48-48 48H208c-26.5 0-48-21.5-48-48V48c0-26.5 21.5-48 48-48zM48 128h80v64H64v256h192v-32h64v48c0 26.5-21.5 48-48 48H48c-26.5 0-48-21.5-48-48V176c0-26.5 21.5-48 48-48z&quot;/></svg>';},1500)" title="복사"><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512" style="width:14px;height:14px"><path fill="currentColor" d="M208 0H332.1c12.7 0 24.9 5.1 33.9 14.1l67.9 67.9c9 9 14.1 21.2 14.1 33.9V336c0 26.5-21.5 48-48 48H208c-26.5 0-48-21.5-48-48V48c0-26.5 21.5-48 48-48zM48 128h80v64H64v256h192v-32h64v48c0 26.5-21.5 48-48 48H48c-26.5 0-48-21.5-48-48V176c0-26.5 21.5-48 48-48z"/></svg></button></div><div style="display:flex; justify-content:space-between; align-items:center; margin-top:4px; font-size:0.75em; opacity:0.5;"><span>터미널에서 실행</span><span>자세한 내용: <a href="https://github.com/pmnxis/chama-optics/blob/master/README_FREEBSD.md" target="_blank" rel="noopener" style="color:inherit; text-decoration:underline;">README_FREEBSD.md</a></span></div></div></details>
<a id="web-app-jump" href="https://pmnxis.github.io/chama-optics/" target="_blank" rel="noopener" style="display:flex; align-items:center; gap:16px; padding:14px 20px; border-radius:8px; background:var(--tw-prose-pre-bg, #1e293b); color:var(--tw-prose-body); text-decoration:none; border:1px solid rgba(128,128,128,0.2); transition: background 0.15s;"><span style="display:flex; align-items:center; gap:4px; flex-shrink:0;"><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512" style="width:20px; height:20px;"><path fill="currentColor" d="M352 256c0 22.2-1.2 43.6-3.3 64H163.3c-2.2-20.4-3.3-41.8-3.3-64s1.2-43.6 3.3-64h185.4c2.2 20.4 3.3 41.8 3.3 64zm28.8-64h122.4c5.3 20.5 8.1 41.9 8.1 64s-2.8 43.5-8.1 64H380.8c2.1-20.6 3.2-42 3.2-64s-1.1-43.4-3.2-64zm112.6-32H376.7c-10-63.9-29.8-117.4-55.3-151.6C399.8 29.1 463.4 97 493.4 160zM256 0c-35.3 0-68 83.8-73.6 192h147.2C324 83.8 291.3 0 256 0zm-190.7 8.4C112.2 29.1 48.6 97 18.6 160h116.7c10-63.9 29.8-117.4 55.3-151.6zM8.1 192C2.8 212.5 0 233.9 0 256s2.8 43.5 8.1 64h123.1c-2.1-20.6-3.2-42-3.2-64s1.1-43.4 3.2-64H8.1zM135.3 352H18.6c30 63 93.6 130.9 172 151.6c-25.5-34.2-45.3-87.7-55.3-151.6zm120.3 160c35.3 0 68-83.8 73.6-192H182.4c5.6 108.2 38.3 192 73.6 192zm65.4-8.4c25.5-34.2 45.3-87.7 55.3-151.6h116.7c-30 63-93.6 130.9-172 151.6z"/></svg><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512" style="width:24px; height:24px;"><path fill="currentColor" d="M64 32C28.7 32 0 60.7 0 96V416c0 35.3 28.7 64 64 64H448c35.3 0 64-28.7 64-64V96c0-35.3-28.7-64-64-64H64zM96 96H416c17.7 0 32 14.3 32 32s-14.3 32-32 32H96c-17.7 0-32-14.3-32-32s14.3-32 32-32z"/></svg></span><span style="display:inline-block"><span style="font-weight:600;">Web App (wasm32)</span><br><span style="font-size:0.8em; opacity:0.6;">⚠ 실험적 지원 · 일부 기능 제한</span></span><span style="margin:-14px -20px -14px auto; width:24px; align-self:stretch; border-radius:0 7px 7px 0; background:repeating-linear-gradient(-45deg, #f59e0b, #f59e0b 5px, #111 5px, #111 10px); flex-shrink:0; opacity:0.55;"></span></a>
</div>

### Mobile

<div style="display: grid; grid-template-columns: 1fr; gap: 12px; margin: 1.5em 0;">
<a id="dl-ios" href="https://apps.apple.com/app/chama-optics/id6758618498" target="_blank" rel="noopener" style="display:flex; align-items:center; gap:16px; padding:14px 20px; border-radius:8px; background:var(--tw-prose-pre-bg, #1e293b); color:var(--tw-prose-body); text-decoration:none; border:1px solid rgba(128,128,128,0.2); transition: background 0.15s;"><span style="display:flex; align-items:center; gap:4px; flex-shrink:0;"><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 384 512" style="width:20px; height:20px;"><path fill="currentColor" d="M318.7 268.7c-.2-36.7 16.4-64.4 50-84.8-18.8-26.9-47.2-41.7-84.7-44.6-35.5-2.8-74.3 20.7-88.5 20.7-15 0-49.4-19.7-76.4-19.7C63.3 141.2 4 184.8 4 273.5q0 39.3 14.4 81.2c12.8 36.7 59 126.7 107.2 125.2 25.2-.6 43-17.9 75.8-17.9 31.8 0 48.3 17.9 76.4 17.9 48.6-.7 90.4-82.5 102.6-119.3-65.2-30.7-61.7-90-61.7-91.9zm-56.6-164.2c27.3-32.4 24.8-61.9 24-72.5-24.1 1.4-52 16.4-67.9 34.9-17.5 19.8-27.8 44.3-25.6 71.9 26.1 2 49.9-11.4 69.5-34.3z"/></svg><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 384 512" style="width:24px; height:24px;"><path fill="currentColor" d="M16 64C16 28.7 44.7 0 80 0h224c35.3 0 64 28.7 64 64v384c0 35.3-28.7 64-64 64H80c-35.3 0-64-28.7-64-64V64zm128 384c0 8.8 7.2 16 16 16h64c8.8 0 16-7.2 16-16s-7.2-16-16-16h-64c-8.8 0-16 7.2-16 16zM80 64v320h224V64H80z"/></svg></span><span style="display:inline-block"><span style="font-weight:600;">iOS (iPhone / iPad)</span><br><span style="font-size:0.8em; opacity:0.6;">App Store 배포 중</span></span></a>
<a id="dl-android" href="https://play.google.com/apps/testing/com.github.pmnxis.chamaoptics" target="_blank" rel="noopener" style="display:flex; align-items:center; gap:16px; padding:14px 20px; border-radius:8px; background:var(--tw-prose-pre-bg, #1e293b); color:var(--tw-prose-body); text-decoration:none; border:1px solid rgba(128,128,128,0.2); transition: background 0.15s;"><span style="display:flex; align-items:center; gap:4px; flex-shrink:0;"><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 576 512" style="width:20px; height:20px;"><path fill="currentColor" d="M420.55 301.93a24 24 0 1 1 24-24 24 24 0 0 1-24 24m-265.1 0a24 24 0 1 1 24-24 24 24 0 0 1-24 24m273.7-144.48 47.94-83a10 10 0 1 0-17.27-10l-48.54 84.07a301.25 301.25 0 0 0-246.56 0L116.18 64.45a10 10 0 1 0-17.27 10l47.94 83C64.53 202.22 8.24 285.55 0 384h576c-8.24-98.45-64.54-181.78-146.85-226.55"/></svg><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 384 512" style="width:24px; height:24px;"><path fill="currentColor" d="M16 64C16 28.7 44.7 0 80 0h224c35.3 0 64 28.7 64 64v384c0 35.3-28.7 64-64 64H80c-35.3 0-64-28.7-64-64V64zm128 384c0 8.8 7.2 16 16 16h64c8.8 0 16-7.2 16-16s-7.2-16-16-16h-64c-8.8 0-16 7.2-16 16zM80 64v320h224V64H80z"/></svg></span><span style="display:inline-block"><span style="font-weight:600;">Android (비공개 테스트)</span><br><span style="font-size:0.8em; opacity:0.6;">2026년 3월 3일 이후 공개 테스트 전환 예정</span></span><span style="margin:-14px -20px -14px auto; width:24px; align-self:stretch; border-radius:0 7px 7px 0; background:repeating-linear-gradient(-45deg, #f59e0b, #f59e0b 5px, #111 5px, #111 10px); flex-shrink:0; opacity:0.55;"></span></a>
</div>

<p style="font-size:0.8em; opacity:0.7; margin-top:4px;">🤖 <b>Android 비공개 테스트 안내</b><br>현재 Google Play 비공개 테스트 단계로, 참여하려면 <a href="https://forms.gle/GFnvyHk7bU14RmdU9" target="_blank" rel="noopener">이 폼</a>에 Play Store 이메일을 등록해야 합니다. Google Play 테스트 정책(12인 이상 14일 테스트) 충족 후, <b>3월 3일 이후 공개 테스트로 전환</b>되어 이메일 등록 없이 누구나 설치할 수 있습니다.</p>
<p style="font-size:0.8em; opacity:0.7;">🔒 <a href="/ko/chama-optics-privacy/">개인정보처리방침</a><br>개인정보 수집 없음 · 회원가입/로그인 없음 · 광고/추적 없음 · 모든 사진 처리는 기기 내에서만 수행 · 얼굴 인식 데이터 서버 전송 없음 · 오프라인 동작</p>

<p style="font-size:0.8em; opacity:0.7;">🌐 앱 내 인도네시아어(Bahasa Indonesia) 번역은 기계번역으로 제공되며, 별도의 검수가 이루어지지 않아 오역이 있을 수 있습니다.</p>

<div style="font-size:0.8em; opacity:0.7; margin-top:2em; line-height:1.6;">
<p>📜 <b>라이선스 · 크레딧</b><br>
<b>Akai Haato</b>는 <b>COVER 주식회사</b> 소속 <b>hololive JP 1기생</b> VTuber입니다.<br>파생저작물 관련 가이드라인: <a href="https://hololivepro.com/en/terms/" target="_blank" rel="noopener">hololive Derivative Works Guidelines</a><br>
<b>ChamaOptics (Desktop Core)</b>: <b>MIT / NON-AI-MIT</b> 혼용 라이선스 · <a href="https://github.com/pmnxis/chama-optics" target="_blank" rel="noopener">GitHub</a><br>
<b>ChamaOptics Mobile</b>: Closed Software License<br>
<b>앱 아이콘</b>: <b>NON-AI / CC-BY-NC</b> · 일러스트레이터 <b>シエミカ</b> (<a href="https://x.com/shiemika324" target="_blank" rel="noopener">@shiemika324</a>) 님의 팬아트이며, 사용 허가를 받은 원화입니다.<br>
※ NON-AI 라이선스는 본 소프트웨어 소스코드 일부에 적용되어 있으며, 관련 리소스(아이콘, 일러스트, 문서 포함)를 인공지능(AI)에 의한 학습, 데이터셋 구축, 검색, 분석, 파생 생성에 사용하는 것을 금지합니다.</p>
</div>

<script>
document.addEventListener('DOMContentLoaded', function() {
  fetch('https://api.github.com/repos/pmnxis/chama-optics/releases/latest')
    .then(function(r) {
      if (!r.ok) throw new Error('API ' + r.status);
      return r.json();
    })
    .then(function(release) {
      var ver = release.tag_name;
      var assets = release.assets || [];
      var map = {
        'dl-macos': { pattern: /macos_arm64.*\.dmg$/i, verEl: 'dl-macos-ver', suffix: '.dmg' },
        'dl-windows': { pattern: /win_x86_64.*\.zip$/i, verEl: 'dl-windows-ver', suffix: '.zip' }
      };
      function fmtSize(bytes) {
        if (bytes >= 1073741824) return (bytes / 1073741824).toFixed(1) + ' GB';
        if (bytes >= 1048576) return (bytes / 1048576).toFixed(1) + ' MB';
        if (bytes >= 1024) return (bytes / 1024).toFixed(1) + ' KB';
        return bytes + ' B';
      }
      Object.keys(map).forEach(function(id) {
        var cfg = map[id];
        var asset = assets.find(function(a) { return cfg.pattern.test(a.name); });
        if (asset) {
          var el = document.getElementById(id);
          if (el) el.href = asset.browser_download_url;
          var verEl = document.getElementById(cfg.verEl);
          if (verEl) verEl.textContent = ver + ' \u00B7 ' + fmtSize(asset.size);
        }
      });
      var linuxVerEl = document.getElementById('dl-linux-ver');
      if (linuxVerEl) linuxVerEl.textContent = ver + ' \u00B7 Debian \u00B7 Ubuntu \u00B7 Fedora \u00B7 Rocky \u00B7 Arch';
      var freebsdVerEl = document.getElementById('dl-freebsd-ver');
      if (freebsdVerEl) freebsdVerEl.textContent = ver + ' \u00B7 FreeBSD 13 \u00B7 14 \u00B7 \u26A0 \uC5BC\uAD74 \uC778\uC2DD \uAE30\uB2A5 \uBBF8\uC9C0\uC6D0';
    })
    .catch(function(err) {
      console.warn('Chama Optics release fetch failed:', err);
      ['dl-macos-ver', 'dl-windows-ver', 'dl-linux-ver', 'dl-freebsd-ver'].forEach(function(id) {
        var el = document.getElementById(id);
        if (el) el.textContent = 'GitHub Releases 페이지에서 다운로드';
      });
    });
});
</script>
