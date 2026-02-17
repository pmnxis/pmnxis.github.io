---
title: "Chama Optics Release"
date: 2026-02-17T00:00:00+09:00
draft: false
categories: ["Chama Optics"]
tags: ["Hololive", "HAACHAMA", "Photography"]
description: "Chama Optics — A photo post-processing app made by a Haaton, from event photos to oshikatsu"
summary: "Releasing Chama Optics — a photo post-processing app for mirrorless and DSLR camera users, inspired by the travel-loving VTuber Akai Haato (赤井はあと)."
showHero: true
heroStyle: "background"
---

## Chama Optics

> 🌐 [한국어 릴리즈 노트](/ko/posts/chama-optics-public-release/) | [日本語リリースノート](/ja/posts/chama-optics-public-release/)

<p style="text-align:center;"><img src="https://github.com/pmnxis/chama-optics/raw/master/assets/mac-icon.png" alt="Chama Optics" style="width:50%; display:block; margin:0 auto;"></p>

> Photos from the venue — edit, frame, and mosaic, all on the spot.

**Chama Optics** is a photo post-processing app for mirrorless and DSLR camera users, inspired by the travel-loving VTuber [Akai Haato (赤井はあと)](https://www.youtube.com/@AkaiHaato). It offers features such as EXIF info frames, automatic face mosaic, color grading, and cheki (Polaroid-style frames).

- **Events, Travel & Street Photography** —<br>When you need to mosaic other people's faces at weddings, VTuber offline events, streets, or tourist spots
- **Fan Photo Hashtag Participants** —<br>When posting photos with tags like [#推し活はあとん日記](https://x.com/hashtag/%E6%8E%A8%E3%81%97%E6%B4%BB%E3%81%AF%E3%81%82%E3%81%A8%E3%82%93%E6%97%A5%E8%A8%98) or [#Towavel](https://x.com/hashtag/Towavel)
- **DSLR/Mirrorless Camera Users & Gear Reviewers** —<br>When sharing shooting info (camera body, lens, aperture, etc.) embedded in a frame

<p style="text-align:right; font-size:0.85em; opacity:0.7;">For the technical development process, see the <a href="/en/posts/chama-optics-dev-story/">dev story</a>.</p>

<p style="text-align:center; margin:1.5em 0 0.5em;"><a href="#download" style="font-size:1.2em; font-weight:600;">⬇ Jump to Download</a></p>

## Key Features

### EXIF Info Frame

<details>
<summary>Automatically reads EXIF data and places it in various themed frames. (Show details)</summary>

Photos taken with a camera contain shooting information such as camera model, lens, shutter speed, aperture, and ISO. Chama Optics automatically reads this data and places it in various themed frames.

| Film Date | Strap | Monitor | Lightroom |
|:---:|:---:|:---:|:---:|
| ![Film Date](https://github.com/user-attachments/assets/a6bf0e51-d3b1-4779-9d65-080b225958f4) | ![Strap](https://github.com/user-attachments/assets/039ab49f-85b1-414b-95e3-2da166cea27f) | ![Monitor](https://github.com/user-attachments/assets/e92b81a0-4465-4dad-9097-7e8b4814fc15) | ![Lightroom](https://github.com/user-attachments/assets/ce1022cd-aea3-4260-9d5f-5e15997388de) |

| One Line | Shot On Two Line | Nikon PhotoStyle | Lumix Photo Style + LUT |
|:---:|:---:|:---:|:---:|
| ![One Line](https://github.com/user-attachments/assets/337337f3-7c17-467a-b965-06481cba98c8) | ![Shot On 2](https://github.com/user-attachments/assets/a9ba4540-6540-418c-8e21-4f9961d7bff7) | ![Nikon](https://github.com/user-attachments/assets/28016fd2-2d4d-4043-88e1-c29f7577a32a) | ![Lumix](https://github.com/user-attachments/assets/62219e6a-c6cb-49ac-9803-cda29321b998) |

Camera manufacturer logos (Canon, Nikon, Sony, Lumix, etc.) are also automatically detected from the shooting data and included in the frame. For Lumix, the applied Photo Style name or LUT filename can also be displayed.

</details>

### Auto Face Detection + Mosaic/Sticker

<details>
<summary>Automatically detects faces in photos and covers them with mosaic or stickers. (Show details)</summary>

\<Work in progress\>

</details>

### Color Grading (LUT)

<details>
<summary>Apply cinematic color tones using .cube LUT files. (Show details)</summary>

You can apply **LUT (Look-Up Table)** files, used in film and photography to apply specific color tones. Load a `.cube` file to give your photos a cinematic color grade.

| Color Grading UI | Result |
|:---:|:---:|
| ![LUT UI](https://github.com/user-attachments/assets/554f96b2-217a-4603-93f5-1df41309f77e) | ![LUT Result](https://github.com/user-attachments/assets/4d49174c-2624-4b3c-a8a1-fc2b56d357c1) |

</details>

### Cheki (Polaroid-style Frame)

<details>
<summary>Automatically generates Polaroid-style cheki frames. (Show details)</summary>

\<Work in progress\>

</details>

### iOS / Android

<details>
<summary>Use the same features as desktop on mobile. (Show details)</summary>

Available via iOS TestFlight since February 2026. The same features as the desktop version can be used on iPhone.

| Gallery | Editor |
|:---:|:---:|
| ![Gallery](https://github.com/user-attachments/assets/65adb13a-18ee-41d2-976e-dd50d0c4dd40) | ![Editor](https://github.com/user-attachments/assets/4abef726-89b4-44f9-873d-685b2f9bcfb7) |

</details>

---


## Download

<div id="chama-downloads" style="display: grid; grid-template-columns: 1fr; gap: 12px; margin: 1.5em 0;">
<a id="dl-macos" href="https://github.com/pmnxis/chama-optics/releases/latest" target="_blank" rel="noopener" style="display:flex; align-items:center; gap:16px; padding:14px 20px; border-radius:8px; background:var(--tw-prose-pre-bg, #1e293b); color:var(--tw-prose-body); text-decoration:none; border:1px solid rgba(128,128,128,0.2); transition: background 0.15s;"><span style="display:flex; align-items:center; gap:4px; flex-shrink:0;"><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 384 512" style="width:20px; height:20px;"><path fill="currentColor" d="M318.7 268.7c-.2-36.7 16.4-64.4 50-84.8-18.8-26.9-47.2-41.7-84.7-44.6-35.5-2.8-74.3 20.7-88.5 20.7-15 0-49.4-19.7-76.4-19.7C63.3 141.2 4 184.8 4 273.5q0 39.3 14.4 81.2c12.8 36.7 59 126.7 107.2 125.2 25.2-.6 43-17.9 75.8-17.9 31.8 0 48.3 17.9 76.4 17.9 48.6-.7 90.4-82.5 102.6-119.3-65.2-30.7-61.7-90-61.7-91.9zm-56.6-164.2c27.3-32.4 24.8-61.9 24-72.5-24.1 1.4-52 16.4-67.9 34.9-17.5 19.8-27.8 44.3-25.6 71.9 26.1 2 49.9-11.4 69.5-34.3z"/></svg><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 512" style="width:24px; height:24px;"><path fill="currentColor" d="M64 96c0-35.3 28.7-64 64-64h384c35.3 0 64 28.7 64 64v256c0 35.3-28.7 64-64 64H128c-35.3 0-64-28.7-64-64V96zM0 403.2C0 392.6 8.6 384 19.2 384h601.6c10.6 0 19.2 8.6 19.2 19.2 0 42.4-34.4 76.8-76.8 76.8H76.8C34.4 480 0 445.6 0 403.2zM512 96H128v256h384V96z"/></svg></span><span style="display:inline-block"><span style="font-weight:600;">macOS (Apple Silicon)</span><br><span id="dl-macos-ver" style="font-size:0.8em; opacity:0.6;">Loading...</span></span></a>
<a id="dl-windows" href="https://github.com/pmnxis/chama-optics/releases/latest" target="_blank" rel="noopener" style="display:flex; align-items:center; gap:16px; padding:14px 20px; border-radius:8px; background:var(--tw-prose-pre-bg, #1e293b); color:var(--tw-prose-body); text-decoration:none; border:1px solid rgba(128,128,128,0.2); transition: background 0.15s;"><span style="display:flex; align-items:center; gap:4px; flex-shrink:0;"><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512" style="width:20px; height:20px;"><path fill="currentColor" d="M0 32h214.6v214.6H0V32zm233.4 0H448v214.6H233.4V32zM0 265.4h214.6V480H0V265.4zm233.4 0H448V480H233.4V265.4z"/></svg><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 640 512" style="width:24px; height:24px;"><path fill="currentColor" d="M64 96c0-35.3 28.7-64 64-64h384c35.3 0 64 28.7 64 64v256c0 35.3-28.7 64-64 64H128c-35.3 0-64-28.7-64-64V96zM0 403.2C0 392.6 8.6 384 19.2 384h601.6c10.6 0 19.2 8.6 19.2 19.2 0 42.4-34.4 76.8-76.8 76.8H76.8C34.4 480 0 445.6 0 403.2zM512 96H128v256h384V96z"/></svg></span><span style="display:inline-block"><span style="font-weight:600;">Windows (x86_64)</span><br><span id="dl-windows-ver" style="font-size:0.8em; opacity:0.6;">Loading...</span></span></a>
<a id="dl-linux" href="https://github.com/pmnxis/chama-optics" target="_blank" rel="noopener" style="display:flex; align-items:center; gap:16px; padding:14px 20px; border-radius:8px; background:var(--tw-prose-pre-bg, #1e293b); color:var(--tw-prose-body); text-decoration:none; border:1px solid rgba(128,128,128,0.2); transition: background 0.15s;"><span style="display:flex; align-items:center; gap:4px; flex-shrink:0;"><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512" style="width:20px; height:20px;"><path fill="currentColor" d="M220.8 123.3c1 .5 1.8 1.7 3 1.7 1.1 0 2.8-.4 2.9-1.5.2-1.4-1.9-2.3-3.2-2.9-1.7-.7-3.9-1-5.5-.1-.4.2-.8.7-.6 1.1.3 1.3 2.3 1.1 3.4 1.7zm-21.9 1.7c1.2 0 2-1.2 3-1.7 1.1-.6 3.1-.4 3.5-1.6.2-.4-.2-.9-.6-1.1-1.6-.9-3.8-.6-5.5.1-1.3.6-3.4 1.5-3.2 2.9.1 1 1.8 1.5 2.8 1.4zM420 403.8c-3.6-4-5.3-11.6-7.2-19.7-1.8-8.1-3.9-16.8-10.5-22.4-1.3-1.1-2.6-2.1-4-2.9-1.3-.8-2.7-1.5-4.1-2 9.2-27.3 5.6-54.5-3.7-79.1-11.4-30.1-31.3-56.4-46.5-74.4-17.1-21.5-33.7-41.9-33.4-72C311.1 85.4 315.7.1 234.8 0 132.4-.2 158 103.4 156.9 135.2c-1.7 23.4-6.4 41.8-22.5 64.7-18.9 22.5-45.5 58.8-58.1 96.7-6 17.9-8.8 36.1-6.2 53.3-6.5 5.8-11.4 14.7-16.6 20.2-4.2 4.3-10.3 5.9-17 8.3s-14 6-18.5 14.5c-2.1 3.9-2.8 8.1-2.8 12.4 0 3.9.6 7.9 1.2 11.8 1.2 8.1 2.6 15.7.8 20.8-5.2 14.4-5.9 24.4-2.2 31.7 3.8 7.3 11.4 10.5 20.1 12.3 17.3 3.6 40.8 2.7 59.3 12.5 19.8 10.4 39.9 14.1 55.9 10.4 11.6-2.6 21.1-9.6 25.9-20.2 12.5-.1 26.3-5.4 48.3-6.6 14.9-1.2 33.6 5.3 55.1 4.1.6 2.3 1.4 4.6 2.5 6.7v.1c8.3 16.7 23.8 24.3 40.3 23 16.6-1.3 34.1-11 48.3-27.9 13.6-16.4 36-23.2 50.9-32.2 7.4-4.5 13.4-10.1 13.9-18.3.4-8.2-4.4-17.3-15.5-29.7z"/></svg><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 576 512" style="width:24px; height:24px;"><path fill="currentColor" d="M64 0C28.7 0 0 28.7 0 64v288c0 35.3 28.7 64 64 64h176l-10.7 32H160c-17.7 0-32 14.3-32 32s14.3 32 32 32h256c17.7 0 32-14.3 32-32s-14.3-32-32-32h-69.3L336 416h176c35.3 0 64-28.7 64-64V64c0-35.3-28.7-64-64-64H64zM512 64v288H64V64h448z"/></svg></span><span style="display:inline-block"><span style="font-weight:600;">Linux (Build from source)</span><br><span id="dl-linux-ver" style="font-size:0.8em; opacity:0.6;">No distro-specific guarantees · Tested on Debian 12 amd64</span></span></a>
<a id="dl-ios" href="https://testflight.apple.com/join/qh9raejp" target="_blank" rel="noopener" style="display:flex; align-items:center; gap:16px; padding:14px 20px; border-radius:8px; background:var(--tw-prose-pre-bg, #1e293b); color:var(--tw-prose-body); text-decoration:none; border:1px solid rgba(128,128,128,0.2); transition: background 0.15s;"><span style="display:flex; align-items:center; gap:4px; flex-shrink:0;"><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 384 512" style="width:20px; height:20px;"><path fill="currentColor" d="M318.7 268.7c-.2-36.7 16.4-64.4 50-84.8-18.8-26.9-47.2-41.7-84.7-44.6-35.5-2.8-74.3 20.7-88.5 20.7-15 0-49.4-19.7-76.4-19.7C63.3 141.2 4 184.8 4 273.5q0 39.3 14.4 81.2c12.8 36.7 59 126.7 107.2 125.2 25.2-.6 43-17.9 75.8-17.9 31.8 0 48.3 17.9 76.4 17.9 48.6-.7 90.4-82.5 102.6-119.3-65.2-30.7-61.7-90-61.7-91.9zm-56.6-164.2c27.3-32.4 24.8-61.9 24-72.5-24.1 1.4-52 16.4-67.9 34.9-17.5 19.8-27.8 44.3-25.6 71.9 26.1 2 49.9-11.4 69.5-34.3z"/></svg><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 384 512" style="width:24px; height:24px;"><path fill="currentColor" d="M16 64C16 28.7 44.7 0 80 0h224c35.3 0 64 28.7 64 64v384c0 35.3-28.7 64-64 64H80c-35.3 0-64-28.7-64-64V64zm128 384c0 8.8 7.2 16 16 16h64c8.8 0 16-7.2 16-16s-7.2-16-16-16h-64c-8.8 0-16 7.2-16 16zM80 64v320h224V64H80z"/></svg></span><span style="display:inline-block"><span style="font-weight:600;">iOS (iPhone / iPad)</span><br><span style="font-size:0.8em; opacity:0.6;">Available via TestFlight, App Store release planned for Feb 2026</span></span></a>
<a id="dl-android" href="https://play.google.com/apps/testing/com.github.pmnxis.chamaoptics" target="_blank" rel="noopener" style="display:flex; align-items:center; gap:16px; padding:14px 20px; border-radius:8px; background:var(--tw-prose-pre-bg, #1e293b); color:var(--tw-prose-body); text-decoration:none; border:1px solid rgba(128,128,128,0.2); transition: background 0.15s;"><span style="display:flex; align-items:center; gap:4px; flex-shrink:0;"><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 576 512" style="width:20px; height:20px;"><path fill="currentColor" d="M420.55 301.93a24 24 0 1 1 24-24 24 24 0 0 1-24 24m-265.1 0a24 24 0 1 1 24-24 24 24 0 0 1-24 24m273.7-144.48 47.94-83a10 10 0 1 0-17.27-10l-48.54 84.07a301.25 301.25 0 0 0-246.56 0L116.18 64.45a10 10 0 1 0-17.27 10l47.94 83C64.53 202.22 8.24 285.55 0 384h576c-8.24-98.45-64.54-181.78-146.85-226.55"/></svg><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 384 512" style="width:24px; height:24px;"><path fill="currentColor" d="M16 64C16 28.7 44.7 0 80 0h224c35.3 0 64 28.7 64 64v384c0 35.3-28.7 64-64 64H80c-35.3 0-64-28.7-64-64V64zm128 384c0 8.8 7.2 16 16 16h64c8.8 0 16-7.2 16-16s-7.2-16-16-16h-64c-8.8 0-16 7.2-16 16zM80 64v320h224V64H80z"/></svg></span><span style="display:inline-block"><span style="font-weight:600;">Android (Closed Testing)</span><br><span style="font-size:0.8em; opacity:0.6;">Open testing planned after March 3, 2026</span></span></a>
</div>

<p style="font-size:0.8em; opacity:0.7; margin-top:4px;">🤖 <b>Android Closed Testing Notice</b><br>Currently in Google Play closed testing. To participate, register your Play Store email via <a href="https://forms.gle/GFnvyHk7bU14RmdU9" target="_blank" rel="noopener">this form</a>. After meeting Google Play testing requirements (12+ testers for 14 days), <b>open testing will begin after March 3</b>, allowing anyone to install without email registration.</p>
<p style="font-size:0.8em; opacity:0.7;">🔒 <a href="/en/chama-optics-privacy/">Privacy Policy</a><br>No personal data collection · No sign-up/login · No ads/tracking · All photo processing runs on-device only · No face recognition data sent to servers · Works offline</p>

<p style="font-size:0.8em; opacity:0.7;">🌐 The Indonesian (Bahasa Indonesia) translation within the app is machine-translated and has not been reviewed. It may contain inaccuracies.</p>

<div style="font-size:0.8em; opacity:0.7; margin-top:2em; line-height:1.6;">
<p>📜 <b>License · Credits</b><br>
<b>Akai Haato (赤井はあと)</b> is a VTuber belonging to <b>hololive JP 1st Generation</b>, operated by <b>COVER Corporation</b>.<br>Derivative works guidelines: <a href="https://hololivepro.com/en/terms/" target="_blank" rel="noopener">hololive Derivative Works Guidelines</a><br>
<b>ChamaOptics Desktop Core</b>: Dual-licensed under <b>MIT / NON-AI-MIT</b> · <a href="https://github.com/pmnxis/chama-optics" target="_blank" rel="noopener">GitHub</a><br>
<b>ChamaOptics Mobile</b>: Closed-source software license<br>
<b>Application Icon</b>: Licensed under <b>NON-AI / CC-BY-NC</b>, with permission from illustrator <b>シエミカ</b> (<a href="https://x.com/shiemika324" target="_blank" rel="noopener">@shiemika324</a>)<br>
<b>Cheki Tab Icon</b>: Modified to monochrome from <a href="https://www.irasutoya.com/2020/08/blog-post_978.html" target="_blank" rel="noopener">いらすとや</a> (みふねたかし) · <a href="https://www.irasutoya.com/p/terms.html" target="_blank" rel="noopener">Terms</a><br>
※ The NON-AI license applies to parts of this software's source code, and prohibits the use of related resources (including icons, illustrations, and documentation) for AI training, dataset creation, indexing, analysis, retrieval, or any form of derivative generation by artificial intelligence.</p>
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
      if (linuxVerEl) linuxVerEl.textContent = ver + ' \u00B7 No distro-specific guarantees \u00B7 Tested on Debian 12 amd64';
    })
    .catch(function(err) {
      console.warn('Chama Optics release fetch failed:', err);
      ['dl-macos-ver', 'dl-windows-ver', 'dl-linux-ver'].forEach(function(id) {
        var el = document.getElementById(id);
        if (el) el.textContent = 'Download from GitHub Releases';
      });
    });
});
</script>
