---
title: "Chama Optics 開発記"
date: 2026-02-14T00:00:00+09:00
draft: false
categories: ["Chama Optics"]
tags: ["Rust", "iOS", "Cross-Platform", "EXIF"]
description: "EXIF ベースの写真フレーム＋顔自動認識アプリ、Rust コアでデスクトップからモバイルまで"
---

> EXIF ベースの写真フレーム＋顔自動認識アプリ、Rust コアでデスクトップからモバイルまで

このブログにオタク全開の記事を堂々と書くのは、おそらく今回が初めてだと思います。
正直、この記事を書き始めてからかなり経つのですが、読者を開発者に向けるべきかVTuberオタクに向けるべきか掴めませんでした。
結局、意識の流れに任せて開発・貢献してきたことを並べていくことにしました。

そこで、このブログでは主にRust Embeddedを扱っており、以前 [billmock-app-rs](https://github.com/pmnxis/billmock-app-rs) というRust Embedded量産プロジェクトを手がけたことがあります。

この記事では [Chama Optics](https://github.com/pmnxis/chama-optics) の開発過程を紹介します。

2026年2月最終週に **0.2.0** で iOS / Android / macOS / Linux / Windows の正式リリースを予定しており、App Store・Google Play 承認前の開発過程を記述しています。

> 🌐 [한국어 아티클](/ko/posts/chama-optics-dev-story/) | [English Article](/en/posts/chama-optics-dev-story/)

---

## プロジェクト紹介

![Chama Optics](haachama-optics.webp)

**Chama Optics** は、DSLR/ミラーレスカメラで撮影した写真のEXIFデータを解析し、さまざまなテーマフレームを適用して、ウォーターマーク・モザイク・ステッカーなどのエフェクトを追加できる写真後処理プログラムです。「Chama」という名前は、旅系VTuber Akai Haato（赤井はあと）のニックネームに由来しています。

数多くのモバイル端末を使ってきましたが、自分の関心はいつも電子機器を**作る**側にあり、スマホアプリ開発とは縁遠かったです。そんな自分がHololive JP 1期生の赤井はあと（HAACHAMA）のファンとしてオタク生活をする中で、組み込みではないソフトウェア開発領域で初めてのプロジェクトを始めることになりました。

このプログラムを最初に構想したのは2025年3月からです。
当時はWebアプリとして動作させたいと考えており、ライブラリのテスト、WASM環境のテスト、libheifのポーティングなどを進めていました。
2025年8月、東京での天音かなたソロライブ LOCK-ON と、米国ニューヨークでのAnimeNYC World Tour + EN Concert（All for one）に行った後、写真を素早く整理してWEBPに圧縮して投稿する必要性を痛感しました。
同時に赤井はあとも最近写真を撮ることが好きで、メンバーシップ限定投稿で自分のカメラを見せたり、推し活はあとん日記（[#推し活はあとん日記](https://x.com/hashtag/%E6%8E%A8%E3%81%97%E6%B4%BB%E3%81%AF%E3%81%82%E3%81%A8%E3%82%93%E6%97%A5%E8%A8%98)）で写真投稿を促していたので、はあと（HAACHAMA）の名前でアプリを一つ開発してあげたかったです。

| 最近の3D Live | Akai Haato X(twitter) |
| ----------- | --------- |
| {{< x user="akaihaato" id="1954294114519851374" >}} | {{< x user="akaihaato" id="1916078150804799563" >}} |

プログラムを開発する際には、以下の鉄則に従いました。
- デスクトップ環境においていかなるアーキテクチャ差別があってはならない。
- MSやAppleの開発エコシステムに最小限しか縛られないこと。
- リソースを多く使ってはならず、高速でなければならない。

目標は大層なもののように見えますが、単に自分がRustマニアだからこうなっただけです。

---

## 始まり：EXIFフレームをもっと手軽に

最初から大掛かりなクロスプラットフォームアプリを計画していたわけではありませんでした。

カメラ好きの間では、写真を投稿する際にEXIF情報をもとにカメラ機種名、レンズ、シャッタースピードなどをフレームに入れて共有する文化があります。自分もこの方法を愛用しており、既存の [exif-frame](https://github.com/jeonghyeon-net/exif-frame) というWeb上のツールを参考にしていました。しかしHEIFフォーマットに対応していない点や高解像度画像出力に制限がある点が惜しく、自分で作ろうという考えからChama Opticsが始まりました。

最初は**デスクトップだけを想定**していました。モバイルへの漠然としたイメージはありましたが、旅先でもいつもMacBookを持ち歩いていたのでモバイルは全く念頭にありませんでした。カメラからSDカードを抜き、MacBookで写真を整理し、フレームをかぶせてアップロードする――そのワークフローが当たり前だったからです。

---

## 方向転換：「会場でMacBookは開けないだろ」

方向が変わったのには二つのきっかけがありました。

### イベント文化の違い ―― AnimeNYCで感じたこと

2025年8月、**AnimeNYC** と **Hololive World Tour / EN Concert** のために米国を訪れました。その時面白い違いに気づきました。米国ではイベント会場で撮った写真を投稿する際、**他人の顔をモザイクしない傾向**が強かったです。しかし韓国や日本のイベントでは、**他人の顔を必ずモザイク処理する**のがマナーであり暗黙のルールでした。

![AnimeNYC イベント写真の例](P1090028-OPTICS.webp)

「他人の顔のモザイク」は毎回手作業でやるにはあまりにも面倒な作業です。特に写真が数十枚、数百枚になればなおさらです。**顔の自動認識＋モザイク/ステッカー機能**が必要だという思いがこの時から強くなりました。

### 間もなく開催のホロライブエキスポ

もう一つの動機は **2026年3月のホロライブエキスポ/フェスティバル** でした。会場ですぐ写真を撮り、その場でフレームをかぶせ、モザイクまで処理してSNSに上げられたら？　でも会場でMacBookは開けません。**スマートフォンですぐ処理できなければなりませんでした。**

### 周囲からのリクエスト

さらに周囲から **iOS版開発へのリクエスト** が加わりました。そこでiOS版を開発し始めたら、今度は **Android版へのリクエスト** も来ました。

こうしてデスクトップ専用だったプログラムがiOS、さらにAndroidまでサポートする方向へ拡張されました。モバイルではミラーレスカメラユーザーよりも**一般ユーザーの体験**をより重視する設計方針としました。EXIFフレームという出発点はそのままですが、「イベント現場で素早く写真を処理して共有する」という新しいユースケースが加わったわけです。

---

## アーキテクチャ：デスクトップからモバイルへ

最初はRust + eguiでデスクトップアプリだけを作るつもりでした。だからコアロジックをすべてRustで書いたことが、結果的に良い選択となりました。iOS/Androidへ拡張する際に、**画像処理、EXIFパース、テーマレンダリング、エンコード/デコードといったコアコードをそのまま再利用**できたからです。

![ChamaOptics Architecture](architecture_layers.svg)

デスクトップではRustコアを**直接リンク**して使い、iOSでは **C FFIを通じてSwiftから呼び出し**、Androidでは **JNA（Java Native Access）を通じてKotlinから呼び出す** 構造です。Rustコアはgit submoduleで管理され、EXIF解析、画像オーバーレイ（テキスト、EXIF、余白、スケーリング、エンコード/デコード）といったコア機能をすべてのプラットフォームで共有します。

ただし、顔認識だけはプラットフォームごとに異なる戦略を使います。
- **デスクトップ（macOS/Windows/Linux）**: ONNX Runtime + InsightFace（SCRFD det_10g）モデル。Speed Modeに応じて640×640固定入力サイズのスライディングウィンドウを多段階（2560/1280/640）に適用して小さな顔まで検出し、NMSで重複を除去するパイプライン
- **iOS**: Apple Vision Frameworkをネイティブで使用。ONNXモデルなしでも高速・高精度で、プライバシー面でも有利
- **Android**: Google ML Kit（`com.google.mlkit:face-detection`）を活用 ―― Googleが提供するオンデバイス顔認識ライブラリ、Rustコアのspeed_modeをFAST/ACCURATEパフォーマンスモードにマッピング

---

## Web版を断念した理由

自分はWeb Appやブラウザの動作の仕組みに詳しくありません。それでもChama Opticsは当初、**Web（WASM）での動作を念頭に置いていました。** eguiがWASMをサポートしているから「デスクトップとWebが同時にいけるだろう」という漠然とした期待がありました。しかし次の二つの機能を実装しようとして断念しました。

- **HEIFデコード**
- **egui WebでのDrag & Drop**

### HEIF：WASMの上にWASM、その間にJS

libheifをブラウザで動かすのが難しいこと以前に、根本的な構造が腑に落ちませんでした。libheifはすでにWASMにコンパイルされた状態で、eguiアプリもWASMです。この二つの間の通信を**JavaScriptを介したFFIで何度も経由しなければならない**ということが理解できませんでした。ほとんどの言語間FFIはCベースで行うのに対し、なぜJSエコシステムではこうしなければならないのか理解できませんでした。

### Drag & Drop：デスクトップ開発者の期待と現実

Drag & Drop以外にも、WASMがブラウザからイベントを受け取る際にJSではなくDOMをバイナリ形態で受け取るとか、もっと従来のデスクトップ/組み込み開発で使われるような方法が提供されると思っていましたが、そうではありませんでした。

### 正直、CとRustしか使えない

自分は**CとRustしか使えません。**　つまり、Web開発そのものに対して非常に無知であるか、過去にやったとしても変な方法で開発していました。

以前、大量のデータをWebに並べる必要があったとき、どうすればいいか分からず、データをCSVにして `,` と `\n` を `<div>` などのHTMLタグに置き換えるのを**hexエディタで一括置換**してstatic webを作りデプロイしたことがあります。21世紀が25%も過ぎた現代のプログラム開発において、自分でも「これは何だ」と思いました。

もちろんWASMはWebなので、Webエコシステムと開発者のやり方に従うのが一般的でしょう。しかし自分はWeb開発者ではないため理解できませんでした。自分は**JS/Webエコシステムとはかけ離れた存在**で、こうした開発環境自体の方向性の違いを克服するよりも、ネイティブモバイルアプリを作る方がよほど自然でした。v0.1.9-betaで正式にWASMサポートを削除し、そのエネルギーをiOS/Androidネイティブに注ぐことにしました。

---

## タイムラインで見る開発の旅

### v0.1.0\~v0.1.1 (2025-10-19\~21) ―― 初のプレリリース

![v0.1.0 スクリーンショット](https://github.com/user-attachments/assets/2471db65-8b0b-44e4-9d2b-31701184878e)

macOS/Windowsバイナリ初配布。Filmテーマフレーム、日本語翻訳、一括保存、ファイル名プレフィックス/サフィックス設定。macOSコード署名DMG配布および日/英/韓インストールガイドWiki作成。

### v0.1.2\~v0.1.6 (2025-10-27\~11-24) ―― テーマ拡張とウォーターマーク

| Film Date テーマ | Strap テーマ | Monitor テーマ | Lightroom テーマ |
|:---:|:---:|:---:|:---:|
| ![Film Date](https://github.com/user-attachments/assets/a6bf0e51-d3b1-4779-9d65-080b225958f4) | ![Strap](https://github.com/user-attachments/assets/039ab49f-85b1-414b-95e3-2da166cea27f) | ![Monitor](https://github.com/user-attachments/assets/e92b81a0-4465-4dad-9097-7e8b4814fc15) | ![Lightroom](https://github.com/user-attachments/assets/ce1022cd-aea3-4260-9d5f-5e15997388de) |

Film Date/Film Glow/Just Frame/Strap/Monitor/Lightroom テーマ追加。ウォーターマーク（9箇所の位置、透明度、ブレンドモード）、フォント選択（内蔵＋OSフォント）、内蔵カメラメーカーロゴ自動適用、HEIF方向修正、可変フォント初期サポート、Longsideスケールオプション。

### v0.1.7 (2025-11-26\~12-19) ―― CJKレンダリング改善とオープンソース貢献

| One Line テーマ | Shot On Two Line テーマ |
|:---:|:---:|
| ![One Line](https://github.com/user-attachments/assets/337337f3-7c17-467a-b965-06481cba98c8) | ![Shot On 2](https://github.com/user-attachments/assets/a9ba4540-6540-418c-8e21-4f9961d7bff7) |

| Nikon PhotoStyle | Lumix Photo Style + LUT |
|:---:|:---:|
| ![Nikon](https://github.com/user-attachments/assets/28016fd2-2d4d-4043-88e1-c29f7577a32a) | ![Lumix](https://github.com/user-attachments/assets/62219e6a-c6cb-49ac-9803-cda29321b998) |

One Line/Two Line/Shot Onテーマ。CJKグリフレンダリングの大幅改善およびSourceHanSans fallback内蔵。Lumix LUT名・Nikon PhotoStyle名をEXIFから抽出するため [exif-rs](https://github.com/kamadak/exif-rs) にPRを提出し先行反映。

### v0.1.8 (2025-12-25\~27) ―― UIリニューアルとパフォーマンス改善

| 画像リストタブ | テーマ設定タブ |
|:---:|:---:|
| ![Tab1](https://github.com/user-attachments/assets/798d9c93-833a-4e9f-876d-ee3fe7182dab) | ![Tab2](https://github.com/user-attachments/assets/4dd969fa-6f75-4f73-8c8a-f89ac66e1b1b) |

タブベースインターフェースへの切り替え（4タブ）、EXIF変数オートコンプリート、画像自動グループ化、2MP MPFプレビューベースのテーマプレビュー、Rayonマルチコア並列処理、システムフォント読み込みメモリ問題の修正。[egui](https://github.com/emilk/egui) にもPRを提出。

### v0.1.9 (2026-01-18\~02-04) ―― 顔認識、LUT、iOS初配布

| 顔認識（デスクトップ） | モザイク適用 |
|:---:|:---:|
| ![Face Detection](https://github.com/user-attachments/assets/a038cdaa-f755-4d8f-97c9-71f57004e739) | ![Mosaic](https://github.com/user-attachments/assets/6ed05280-1980-448a-9243-aff0836d2470) |

| カラーグレーディングUI | LUT適用結果 |
|:---:|:---:|
| ![LUT UI](https://github.com/user-attachments/assets/554f96b2-217a-4603-93f5-1df41309f77e) | ![LUT Result](https://github.com/user-attachments/assets/4d49174c-2624-4b3c-a8a1-fc2b56d357c1) |

| iOS ギャラリー | iOS 編集 |
|:---:|:---:|
| ![Gallery](previews-d2OPTICS.webp) | ![Editor](color-themes-d2OPTICS.webp) |

デスクトップ単独リリースの最終版であり、iOSアプリ初配布バージョンです。ONNX（InsightFace）顔検出＋モザイク/ストローク/ステッカーオーバーレイ。1D/3D LUTカラーグレーディング（[wagahai-lut](https://github.com/pmnxis/wagahai-lut)）。iOSはSwiftUI + Vision Frameworkネイティブ顔認識、Rust FFIブリッジ（`ffi_ios.rs` + `RustBridge.swift`）。インドネシア語翻訳追加。

---

## 技術的チャレンジと解決策

プロジェクト全般にわたって適用されたパフォーマンス戦略をまず整理すると以下の通りです。

- **Rayon並列処理** ―― 大量画像の一括エクスポート時にマルチコアを活用。ただし、色補正などピクセル単位の処理では10万ピクセル以上の場合のみ `par_chunks_exact_mut()` で並列化し、小さい画像はコンテキストスイッチのオーバーヘッドを避けるため逐次処理します。
- **`fast_image_resize` ベースのリサイズ** ―― `image` クレートのデフォルトリサイズの代わりにSIMD最適化された `fast_image_resize` を使用し、サムネイル生成やプレビューリサイズの速度を大幅に改善
- **Lazyロードとキャッシュ** ―― LUTファイルは `lut_cache: HashMap<Uuid, CubeLut>` に初回使用時にパースしてキャッシュし、EXIFサムネイルも `thumbnail_cache` に遅延ロードします。バックグラウンドスレッド用の複製（`clone_for_thread()`）時にはキャッシュを除外し、不要なメモリ複製を防止します。
- **パーセプチュアルハッシュベースの画像グループ化** ―― 画像ロード時に8×8グレースケール平均ハッシュ（64-bit）を事前計算し、その後の類似画像グループ化をハミング距離O(1)比較で実行。元画像を再ロードせずメタデータだけでグループ化します。
- **ビルドプロファイル最適化** ―― Releaseビルドで `opt-level = 3`、`lto = "fat"`、`codegen-units = 1` を適用し、Devビルドでも `fast_image_resize`、`mozjpeg`、`ab_glyph` などパフォーマンスに敏感な依存関係は `opt-level = 3` で個別設定して、デバッグ中でも画像処理パフォーマンスを維持します。

### 1. クロスプラットフォームFFIの複雑さ

Rustコアを3つのプラットフォーム（デスクトップ/iOS/Android）で使うために、それぞれ異なるFFI戦略を採用しました。

| プラットフォーム | FFI方式 | 特徴 |
|:---|:---|:---|
| デスクトップ (egui) | 直接リンク | Rust → Rust、FFI不要 |
| iOS (SwiftUI) | C FFI (`@_silgen_name`) | SwiftからC関数を直接呼び出し |
| Android (Compose) | JNA (Java Native Access) | KotlinからJNA経由で.soを呼び出し |

このブリッジレイヤーをメンテナンスしつつも安定したメモリ管理（文字列の割り当て/解放、不透明ポインタハンドルパターン）を保証することが主な課題でした。

### 2. EXIFパースの尽きない変数

カメラごとにEXIFの記録方式が異なります。
- **シャッタースピード/F値の浮動小数点問題** ―― 1/125秒が `0.008000000` のような汚い値で記録される場合の自動補正
- **HEIF/HEIC方向情報の誤り** ―― 一部の画像で方向がずれる問題
- **レンズ情報のないカメラ** ―― Nikon Coolpixのようなコンパクトカメラへの対応
- **MakerNoteに隠された情報** ―― Lumix LUT名、Nikon PhotoStyle、Sony Creative Lookなどメーカー別非公開EXIFフィールドのパース

これらのために [exif-rs](https://github.com/kamadak/exif-rs) ライブラリに直接PRを送り、必要な機能を追加しました。

### 3. MakerNoteパース：メーカー別撮影設定の抽出

最近のミラーレスカメラは独自の色味を合わせる機能が非常に優れています。LumixのPhoto Style、NikonのPicture Control、SonyのCreative Lookなどがそれです。写真家の間では「どの色味設定で撮ったか」がカメラ機種やレンズと同じくらい重要な情報であり、この情報をフレームに一緒に入れられたらいいなと思いました。

[EXIF標準](https://www.cipa.jp/std/documents/download_e.html?DC-008-Translation-2023-E)の **MakerNote**（Tag 0x927C）は、カメラメーカーが自由に使える非標準領域です。フォーマットはメーカーごとに、さらには同じメーカーのモデルごとに異なり、ドキュメントも乏しいです。しかしここには**「どの色味設定で撮ったか」**のような、写真家にとって重要な情報が隠れています。

![MakerNote パースフロー](makernote_parsing.svg)

Chama Opticsでは `exif.maker_note_vendor()` でメーカーを先に識別し、各メーカー専用パーサーに分岐します。

**Nikon** ―― `PictureControlData` / `PictureControlData2` タグからPicture Control名を抽出します。`"VitalityFilm_Pmango"` のようなユーザー定義プロファイル名や、`"Flat"`、`"Vivid"` のようなプリセット名がここに格納されています。

**Panasonic (Lumix)** ―― 最も豊富なデータを提供します。`PhotoStyleName` から基本のPhoto Style名（`"NostalgicKintex"`）を、`LutPrimaryFile`/`LutSecondaryFile` から適用されたLUTファイル名（`"KintexYellow33.CUBE"`）とGain値まで抽出します。この情報はChama OpticsのLUTカラーグレーディング機能と直接連携します。

**Sony** ―― `Sony_0x9416` タグからCreative Style/Creative Look情報（`"Vivid"`、`"Standard"`、`"Portrait"` など）を抽出します。

このMakerNoteパース機能は既存のexif-rsになかったため、直接実装して [PR #57](https://github.com/kamadak/exif-rs/pull/57) として提出しました。

#### EXIF IFDエントリ構造とMakerNoteのoffset問題

MakerNoteをパースするには、まずEXIFのIFD（Image File Directory）構造を理解する必要があります。EXIFデータはTIFFフォーマットを基盤としており、各IFDエントリはちょうど **12バイト** で構成されます。

- **Tag**（2バイト） ―― フィールド識別子（例：`0x927C` = MakerNote）
- **Type**（2バイト） ―― データ型
- **Count**（4バイト） ―― 値の個数
- **Value/Offset**（4バイト） ―― データが4バイト以下なら値そのもの、超過なら**データ位置を指すoffset**

標準EXIFではこのoffsetは **TIFFヘッダー開始点からの距離** です。シンプルで明快です。ところがMakerNote内部のIFDではこのルールが崩れます。

![EXIF IFDエントリ構造とMakerNote offset方式](exif_makernote_structure.svg)

**問題はこれです**：MakerNote内部にもIFDと同一構造のエントリが存在しますが、ここでのoffsetが「どこを基準にした距離か」が**メーカーごとに異なります。**

- **TIFF-Relative方式**（Panasonic、Canon、Sony、Leica、Sigma） ―― MakerNote内のoffsetが元のTIFFヘッダー開始点基準。MakerNote自体にTIFFヘッダーがなく、offsetから `tiff_offset`（TIFFスタートからMakerNoteまでの距離）を引くことで実際のデータ位置を見つけられます。
- **MakerNote-Relative方式**（Nikon、Olympus、Fujifilm、Samsung、Apple、Pentax） ―― MakerNote内のoffsetがMakerNote開始点基準。自己完結型の構造（self-contained）であり、Nikonの場合はMakerNote内に独自のTIFFヘッダーまで持っています。

さらにバイトオーダー（エンディアン）もメーカーごとに異なります。Nikonは独自TIFFヘッダーから、Olympus/Appleはプロプライエタリヘッダー内の `"II"`/`"MM"` バイトから、SamsungはIFDタグ番号のパターンで自動判別します。

結局、10メーカー（Panasonic、Nikon、Sony、Canon、Olympus、Fujifilm、Samsung、Apple、Sigma、Pentax）それぞれのヘッダーフォーマット、offset補正式、バイトオーダー判別ロジックを実装し、合計23ファイル約5,900行のPRとなりました。

### 4. カメラメーカーロゴシステム：CSV → build.rs → バイナリ埋め込み

Strapテーマ、Filmテーマなどで写真フレームに**カメラメーカーのロゴ**を自動挿入するには、二つのことが必要です。(1) EXIFからメーカーを認識すること、(2) 該当メーカーのSVGロゴをレンダリングすること。

#### コンパイルタイム：CSVからSVGダウンロード＆埋め込み

以前のRust Embedded量産プロジェクトで [`const fn`/`const impl` でコンパイルタイムに最大限任せるアプローチ](/ko/posts/my_first_commerical_rust_embedded_product_3/) と [`build.rs` を活用したビルドスクリプト手法](/ko/posts/my_first_commerical_rust_embedded_product_4/) を扱ったことがあります。Chama Opticsのロゴシステムはこの経験の延長線上で `build.rs` + `include_bytes!()` を積極活用しています。

![コンパイルタイム ロゴパイプライン](logo_build_pipeline.svg)

`assets/logo_mnf.csv` に35メーカーのロゴ情報が定義されています。`cargo build` が実行されると `build.rs` がこのCSVを読み取り、以下を実行します。

1. **SVGダウンロード** ―― 各行の `url` カラムからSVGを取得します。Wikimedia Commons URLならHTTPでダウンロードし、ローカルパス（`assets/logo_mnf/contax.svg`）なら直接読み込みます。ネットワーク失敗時は最大3回、5秒間隔でリトライします。
2. **MD5ハッシュ検証** ―― ダウンロードしたファイルのMD5ハッシュをCSVの `expected_md5` 値と比較します。**ファイルがすでに存在しハッシュが一致すれば再ダウンロードをスキップします。** ハッシュが不一致なら `panic!` でビルドを中断します ―― Wikimedia側でSVGが変更された場合は意図的に確認する必要があるためです。
3. **Rustコード生成** ―― `assets/auto_generated/logo_assets.rs` を生成し、各SVGを `include_bytes!()` でバイナリに埋め込みます。ランタイムでのファイルロードは不要です。

```rust
// 自動生成されるコード例
pub const LOGO_ASSETS: &[ArtAsset] = &[
    ArtAsset {
        key: "canon.svg",
        data: include_bytes!(".../assets/download/canon.svg"),
        color_type: ColorType::Color,
        mnf: "canon", model: "",
        mnf_model_rel: MnfRelation::Any,
    },
    // ... 35メーカー
];
```

#### ランタイム：EXIF → ロゴマッチング → SVGラスタライズ

![ランタイム ロゴマッチング](logo_runtime_matching.svg)

写真がロードされるとEXIFの `Tag::Make`（メーカー）と `Tag::Model`（モデル名）を抽出し、`LOGO_ASSETS` 配列を巡回してマッチングします。

マッチングルールは二つです。
- **`MnfRelation::Any`** ―― メーカー名**または**モデル名のどちらか一方が一致すればよい（ほとんどの場合）
- **`MnfRelation::Both`** ―― メーカー名**かつ**モデル名の両方が一致しなければならない（特殊なケース）

`Both` が必要な実例：**Sigma** は2025年にロゴを変更しました。新ロゴを使うカメラは `SIGMA BF` モデルのみなので、CSVに `mnf="sigma", model="sigma bf", mnf_model_rel=Both` で `sigma2025.svg`（新ロゴ）を登録し、残りのSigmaカメラは `mnf="sigma", mnf_model_rel=Any` で `sigma.svg`（旧ロゴ）を使うように分離しました。

マッチングされたSVGは `usvg` でパース後 `resvg`+`tiny-skia` でラスタライズし、フレーム内の適切な位置とサイズで合成されます。`color_type`（Black/Color）と `fill_ops`（Default/Monochrome）に応じてレンダリング方式が変わり、背景色に合ったロゴ表現が可能です。

### 5. CJKフォントレンダリングと可変フォント（Variable Font）最適化

日本語・韓国語・中国語テキストを画像にレンダリングする際、多くの問題が発生しました。
- 一部のCJK漢字（ideograph）がレンダリングされない問題
- 可変フォントでグリフ幅が合わない問題

解決策として **SourceHanSansをビルトインfallbackフォント** として内蔵し、選択したフォントでサポートされないグリフを自動で代替レンダリングするようにしました。具体的にはテキストを文字単位で巡回し、主フォントで `GlyphId(0)`（グリフなし）が返された場合、SourceHanSans fallbackフォントに切り替えてレンダリングします。

#### 可変フォント weight リマッピング

Chama Opticsで使用する主フォント **BarlowGX.ttf** は可変フォント（Variable Font）ですが、内部のweight軸値が **22~188** という非標準範囲を使用していました。CSS標準やFreeTypeなどで使われる **100~900** の範囲と合わないため、`ab_glyph` で `set_variation(b"wght", 400.0)` でRegular weightを指定しても意図した結果になりませんでした。さらにデフォルトのwidthがwdth=300（Condensed）に設定されておりグリフ幅も合いませんでした。

単に `fvar`（Font Variationsメタデータ）だけ修正すればいいと思いましたが、実際のグリフ幅を保持する `hmtx` テーブルは依然としてCondensed基準でした。**メタデータだけ変えてもレンダリング結果は変わりません。** 結局BarlowGX.ttfから9つのweightインスタンスをwdth=500（Regular width）で抽出し、これをマスターソースとして `fontTools.varLib.build()` で可変フォントを丸ごとリビルドして解決しました。成果物が `Barlow-Variable-Remapped.ttf` と `Barlow-Variable-Remapped-Narrow.ttf` です。

#### 複数のフォントファイルを一つに統合 ―― ファイルサイズの絶対的な削減

可変フォントのもう一つの利点は、**複数のweightファイルを一つにまとめられる**ことです。従来Barlow-Thin.ttf、Barlow-Light.ttf、Barlow-Regular.ttf、Barlow-Bold.ttf、Barlow-Black.ttfなど9つ以上の静的フォントファイルが必要だったものを、可変フォント一つで置き換えられます。

CJKフォントも同様です。SourceHanSans（日中韓文字に最適な選択）は元々weight別に別ファイルが提供されますが、可変フォント版（`SourceHanSansVF`）を使えば一つのファイルで200~800範囲のweightをすべてカバーします。ただしこのフォントもBarlowGXと同じ問題があり、weight軸を標準範囲にリマッピングして `SourceHanSansVF-remapped.otf` を生成しました。

さらに `fontTools` を活用して、**異なる文字集合を持つフォントを一つに統合**する作業も行いました。ラテン文字フォント＋日本語フォント＋韓国語フォントを合わせて一つのファイルにでき、WOFF2解凍、TTC（Font Collection）処理、特定weightのインスタンス抽出、UTF-8ベースの文字サブセッティングなどを組み合わせて最終ファイルサイズを最小化しました。

最終的にChama Opticsに内蔵されるフォントファイルは：

| フォント | 静的フォント時の容量 | 可変フォント容量 | 削減 |
|:---|---:|---:|:---|
| `Barlow-Variable-Remapped.ttf` (100~900) | ~1.35 MB (9 weight) | **385 KB** | **~3.5x** |
| `Barlow-Variable-Remapped-Narrow.ttf` (100~900) | ~1.45 MB (9 weight) | **207 KB** | **~7x** |
| `SourceHanSansVF-remapped.otf` (200~800) | ~105 MB (7 weight) | **30 MB** | **~3.5x** |
| `DejaVuSansMono.ttf` (静的) | — | 327 KB | — |
| `digital-7.ttf` (静的) | — | 34 KB | — |
| **合計** | **~108 MB** | **~31 MB** | **~3.5x** |

Barlowの場合が特に劇的です。元のBarlowプロジェクトには 9 weight × 3 width × 2 (upright+italic) = 54個の静的TTFファイルがあり合計 **8.5 MB** ですが、Chama Opticsで必要な normal + narrow の2つの可変フォントは合わせて **592 KB** に過ぎません。モバイルアプリのバンドルサイズに敏感な環境ではこの差は決定的です。

#### eguiでの可変フォント weightセレクティブロード

デスクトップ版（egui）では可変フォントのweightを**ユーザーが自由に調整**できます。核心は `ab_glyph` クレートの `set_variation` APIです。

```rust
pub struct VariableFontPack {
    pub label: &'static str,
    pub font: ab_glyph::FontRef<'static>,
    pub default: u16,       // デフォルトweight（例：300）
    pub start: u16,         // 最小weight（例：100）
    pub end_include: u16,   // 最大weight（例：900）
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

ユーザーがテーマ設定でweightスライダーを調整すると、そのweight値で `set_variation(b"wght", weight)` を呼び出してランタイムでフォントの太さが変更されます。100（Thin）から900（Black）まで連続的な値を指定でき、350や450のような中間値も補間（interpolation）されてスムーズなweight遷移が可能です。

このロジックはデスクトップだけでなくiOS/Androidでも同様に動作します。iOSでは `FontSelectionView` で可変フォントの場合のみweightスライダーを表示し、選択されたweight値をFFI経由でRustコアに渡します。AndroidでもKotlinから `fontWeight` パラメータでRust FFIに渡す同一の構造です。

CJK fallbackもweightを反映します。主フォントがBarlow weight 700（Bold）でCJK文字が出現した場合、SourceHanSansも700に近いweightでレンダリングし、**ラテン文字とCJK文字の太さが一貫して**見えるようにしました。

#### ビルトインフォントとシステムフォント

Chama Opticsで使用するフォントは二種類に分かれます。**ビルトイン（builtin）フォント**と**システム（OS）フォント**です。

ビルトインフォントはアプリにデフォルト内蔵されるフォントで、Barlow（ラテン）、SourceHanSans（CJK fallback）、D2Coding（モノスペース）、Digital-7（セグメントディスプレイスタイル）などがあります。システムフォントはユーザーのOSにインストールされたフォントを取得してテーマに適用できるようにする機能です。EXIFフレームに表示されるテキストのフォントをユーザーが自由に選択できる必要があるため、ビルトインフォントだけでは不十分です。

**デスクトップでは `include_bytes!` でフォントをバイナリに内蔵**します。

```rust
pub(crate) const FONT_BARLOW: BuiltInFonts = BuiltInFonts {
    name: "Barlow",
    data: include_bytes!("../../assets/fonts/Barlow-Variable-Remapped.ttf"),
};
```

デスクトップは実行ファイル一つで配布するのが便利なため、フォントファイルをコンパイル時にバイナリに含めます。別途のフォントディレクトリなしに実行ファイルだけですぐ動作します。

一方、**iOS/Androidではフォントをファイルパスで動的ロード**します。モバイルアプリはバイナリサイズに敏感で、アプリバンドル内にリソースファイルとして分離するのがプラットフォームの慣例でもあります。Swift/KotlinからFFI経由でフォントディレクトリパスをRustコアに渡すと、Rust側で `std::fs::read()` で必要なタイミングにファイルを読み込んでロードします。

システムフォントはデスクトップでのみサポートします。`font-kit` クレートの `SystemSource` を使用してOSにインストールされたフォント一覧を列挙し、ユーザーが選択したフォントをロードします。この処理はUIをブロックしないようバックグラウンドスレッドで実行し、`Arc<RwLock<Vec<SystemFont>>>` でスレッドセーフに共有します。

#### font-kit macOS メモリ暴走のデバッグ

システムフォント列挙を実装した後、macOSで深刻な問題が発生しました。アプリ起動直後に**メモリ使用量が1.0GB、ピーク1.5GBまで跳ね上がる**現象でした。（[#5](https://github.com/pmnxis/chama-optics/issues/5)）

`MallocStackLogging` と `malloc_history` で追跡した結果、原因は `font-kit` のmacOSバックエンド（`core_text`）にありました。`font_kit::SystemSource::all_fonts()` がシステムフォント一覧を列挙する際、**各フォントのファイルデータ全体をメモリに読み込んでいました**：

```
435 calls for 2045941700 bytes:  ← 約2GB
  font_kit::sources::core_text::create_handles_from_core_text_collection
    font_kit::utils::slurp_file    ← フォントファイル全体をメモリに読み込み
      alloc::raw_vec::RawVecInner::try_allocate_in
```

macOSには数百のシステムフォントがインストールされており、CJKフォント（例：Apple SD Gothic Neo、Hiraginoなど）は個別ファイルが数十MBに達します。`slurp_file` がこれらのファイルをすべてメモリに載せ、435フォントに対して約2GBを割り当てたのです。（Windowsでは同じコードで約90MB程度でした。）

解決方法は `font-kit` をフォークして `all_fonts()` 呼び出し時に**フォントデータを読まずメタデータ（名前、パス）のみ収集**するよう修正することでした。修正後、メモリ使用量は **144.9MB**（ピーク389.4MB）に大幅削減されました。

### 6. LUTカラーグレーディング：wagahai-lutの最適化哲学

ライブラリ名の由来は [wagahaida_L（ラプラス・ダークネス）](https://x.com/wagahaida_L) のツイートから取りました。

| LaplusDarknesss | wagahaida_L |
| ---------- | ---------- |
| {{< x user="LaplusDarknesss" id="1940063453647147386" >}} | {{< x user="wagahaida_L" id="1992881206682423708" >}} |

> 余談ですが、v0.2.0で準備中のチェキ風（ポラロイド）画像自動生成機能もラプラス・ダークネスからアイデアを得ました。妖しく頭がいいと思います。

v0.1.9で追加されたLUTカラーグレーディング機能は、自作の [wagahai-lut](https://github.com/pmnxis/wagahai-lut)（[crates.io](https://crates.io/crates/wagahai-lut)）ライブラリを使用します。

#### CUBE LUTとは？

CUBE LUT（Look-Up Table）は [Adobeが定義した `.cube` ファイルフォーマット](https://web.archive.org/web/20220220033515/https://wwwimages2.adobe.com/content/dam/acom/en/products/speedgrade/cc/pdfs/cube-lut-specification-1.0.pdf) で、色変換情報を格納しています。1D LUTと3D LUTの2種類があります。

![CUBE LUT 概念](lut_cube_concept.svg)

**1D LUT** はR、G、B各チャンネルを独立して変換します。入力値をテーブルから探して出力値に置き換える単純な構造です。明るさ/コントラスト調整に適していますが、チャンネル間の相互作用（例：赤を青に変えること）は不可能です。テーブルサイズは通常1,024（10-bit）から65,536（16-bit）エントリで、隣接する2エントリ間の値は線形補間（linear interpolation）で計算します。

**3D LUT** はRGB 3次元色空間全体をマッピングします。入力 (R, G, B) がまったく異なる (R', G', B') に変換されうるため、映画/写真のクリエイティブな色味補正（film look、color grading）に使われます。キューブ内部の格子点（lattice point）が既知のマッピングを定義し、格子点間の値は周囲8つの頂点から**三線形補間**（trilinear interpolation）で計算します。一般的なサイズは17³（4,913点）、33³（35,937点）、65³（274,625点）です。

#### wagahai-lutの最適化戦略

既存のRust LUTライブラリは汎用性に重点を置いていました。wagahai-lutは「24MP写真数十枚を一括処理しても速くなければならない」というChama Opticsの要件に合わせ、メモリレイアウトからSIMDレベルまで最適化しました。ただしx86_64とARM64の両方をサポートする必要があるため、直接アセンブリを書く代わりに [`wide`](https://crates.io/crates/wide) クレートを使用して、アーキテクチャに依存しない汎用的なベクトル最適化を選択しました。

![wagahai-lut 最適化戦略](lut_optimization.svg)

**1) Structure of Arrays (SoA) メモリレイアウト**

一般的な3D LUT実装は `[Rgb, Rgb, Rgb, ...]` 形式のAoS（Array of Structures）レイアウトを使用します。しかし三線形補間は一度に1チャンネルずつ8頂点の値を読む必要があるため、AoSではキャッシュラインに不要なチャンネルデータが一緒にロードされます。

wagahai-lutは3D LUTを `r: Vec<f32>`、`g: Vec<f32>`、`b: Vec<f32>` の3つの分離された配列で格納します。このSoAレイアウトのおかげで、1チャンネルの補間に必要な8つの値がメモリ上で近くに位置し、CPUキャッシュヒット率が向上します。

**2) SIMD並列処理 (`wide::f32x4`)**

1D LUT処理では `wide` クレートの `f32x4` SIMDベクトルを使用し、R、G、B 3チャンネルの線形補間を**単一ベクトル演算**で実行します。4レーンのうち3つをR、G、Bに割り当て、乗算・加算が1命令で処理されます。

**3) 固定サイズ特殊化（Fixed-Size Specialization）**

1D LUTは `Bit10(1024)`、`Bit12(4096)`、`Bit14(16384)`、`Bit16(65536)` などの一般的なサイズに対して `Box<[Rgb; SIZE]>` 固定サイズ配列を使用します。コンパイルタイムにサイズが確定するため、境界チェック（bounds checking）をスキップして `get_unchecked()` で直接アクセスが可能です。3D LUTも17³、33³、65³のような一般的なサイズを別タイプで提供します。

**4) In-Place処理とループ最適化**

`apply_rgb_mut()` / `apply_rgba_mut()` 関数は画像バッファをその場で（in-place）修正し、追加メモリ割り当てが一切ありません。ホットループではドメイン範囲の逆数（`inv_domain_range`）をループ外で事前計算し、生ポインタ（raw pointer）演算で `get_pixel()`/`put_pixel()` 呼び出しのオーバーヘッドを除去し、バイトスライスを線形に巡回してCPUキャッシュプリフェッチを最大化します。

#### ベンチマーク結果

M4 Max（Stable Rust）基準の処理時間（JPEGデコード/エンコード時間含む）：

| 解像度 | 1D LUT | 3D LUT |
|:---|---:|---:|
| 1920×1080 (FHD) | 14.39 ms | 19.40 ms |
| 6000×4000 (24MP) | 159.91 ms | 223.15 ms |
| 8144×5424 (44MP) | 294.34 ms | 417.09 ms |

24MP写真基準で3D LUT適用が約0.22秒であり、Chama Opticsで数十枚の写真を一括処理する際にRayon並列化と組み合わせれば実用的な速度を達成できます。

### 7. デスクトップ顔認識：Speed Modeとスライディングウィンドウアルゴリズム

デスクトップではONNX Runtime + InsightFace（det_10g）モデルを使用します。このモデルの入力サイズは**固定640×640ピクセル**です。しかし実際の写真は24MP（6000×4000）以上であることがほとんどで、640×640に画像全体を縮小すると、人物が小さく写った集合写真では顔を見逃してしまいます。

この問題を解決するため、**Speed Modeに応じた多段階スライディングウィンドウアルゴリズム**を実装しました。

| モード | max_depth | Depth Loopウィンドウサイズ | 全体動作 |
|:---|:---:|:---|:---|
| Fastest | 0 | (なし) | 画像全体 → 640×640リサイズ → 単一推論 |
| Fast | 1 | (なし、depth loop未実行) | + 短辺（`min(W,H)`）サイズのスライディングウィンドウ |
| Normal | 1 | 640×640 | + 640×640精密ウィンドウ |
| Slow | 2 | 1280×1280 → 640×640 | + 1280→640多段階ウィンドウ |
| Slowest | 3 | 2560×2560 → 1280×1280 → 640×640 | + 2560→1280→640全多段階ウィンドウ |

> **Depth Loopウィンドウサイズの公式**: `window = 640 × 2^(max_depth - depth - 1)`
>
> 例：Slowest(max_depth=3) → depth 0: 2560, depth 1: 1280, depth 2: 640

アルゴリズムの流れは以下の通りです。

1. **第1段階（共通）**: 画像全体を640×640にリサイズして単一推論。大きな顔はこの段階で捕捉されます。
2. **第2段階（Fast以上）**: 画像の短辺（`min(width, height)`）サイズのスライディングウィンドウを10%オーバーラップで移動させ、各ウィンドウを640×640に縮小して推論。異常なアスペクト比（パノラマなど）での漏れを防止します。
3. **第3段階（Normal以上）**: `640 × 2^(max_depth - depth - 1)` サイズのウィンドウをdepthごとに巡回。Slowestは2560→1280→640、Slowは1280→640、Normalは640単一depth。
4. **最終**: NMS（Non-Maximum Suppression、IoU閾値0.4）で重複検出を除去します。

各Speed Modeの動作を視覚化したダイアグラム（6000×4000元画像基準）：

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

以下はSlowestモードで処理した実際のイベント写真の例です。大規模な集合写真で後列の隅にいる小さな顔まで漏れなく検出してモザイク処理した結果です。

![Slowest モード適用例](P1167220-OPTICS.webp)

> 2025年AGF 天音かなた ファン集合写真会

各モードの使用シナリオ：

| モード | 平均所要時間 | 適した状況 |
|:---|:---|:---|
| Fastest | ~0.5秒 | 1~2人のポートレート |
| Fast | ~0.6秒 | パノラマなど異常アスペクト比の1~2人写真 |
| Normal | ~7秒 | 約10人程度の集合写真 |
| Slow | ~13秒 | 40~50人規模の集合写真 |
| Slowest | ~28秒 | 50人以上の大規模集合写真 |

Fastestが画像全体を640×640一つに縮小して~0.5秒で終わるのに対し、Slowestは2560/1280/640の3段階のウィンドウをオーバーラップさせて探索するため~28秒かかります。しかしイベント会場の集合写真で後列の隅の小さな顔まで捕捉するにはこの程度の探索が必要です。

実行環境（Execution Provider）もプラットフォームごとに最適化されています。

- **macOS**: CoreML Execution Provider自動選択 ―― AppleのNeural Engine/GPUアクセラレーション活用
- **Windows/Linux**: CPUまたはOnnxAuto（自動検出）

macOSではユーザーがCPUを選択しても内部的にCorMLに自動切り替えされ、**Apple SiliconのNeural Engineを活用**します。これはCPU比で数倍のパフォーマンス向上をもたらします。

iOSでは、スライディングウィンドウアルゴリズムの構造（Fastest/Fast/ピラミッド深度）はデスクトップと同一ですが、推論エンジンとしてInsightFace ONNXの代わりに**Apple Vision Framework（`VNDetectFaceRectanglesRequest`）**を使用します。Visionが内部的にスケーリングを処理するため640×640リサイズ手順が不要で、ONNXモデルなしでも正確な顔認識が可能です。

Androidでは、Google ML Kit（`com.google.mlkit:face-detection`）を多段階累積構造で使用します。
- **Pass 1（全速度）**: 全体イメージを最大1024pxでデコード、`PERFORMANCE_MODE_FAST`、`minFaceSize=0.2`
- **Pass 2（Fast以上）**: 1024pxビットマップ上で`min(w,h)`サイズの正方ウィンドウを10%オーバーラップでスライディング
- **Pass 3+（Normal以上）**: ピラミッドマルチレベルデコード ―― `base = floor(min(minSide/2, maxSide/3) × 1.1)`を基準に、NormalはL0、SlowはL0–L1、SlowestはL0–L2まで`PERFORMANCE_MODE_ACCURATE`、`minFaceSize=0.1`で探索
- 全Passの結果をNMS（IoU 0.4）でマージして重複除去

### 8. iOSネイティブ統合

iOSアプリは単にRustコアをラップするだけでなく、プラットフォームの利点を最大限に活用しました。
- **Vision Framework** ―― 顔認識をiOSネイティブで処理し、ONNXモデルなしでも高速・高精度な認識
- **PhotosUI** ―― iOS写真ライブラリから直接画像を選択
- **Metalレンダリング** ―― GPUアクセラレーション画像処理
- **iPadサポート** ―― 広い画面に最適化されたレイアウト

### 9. MPFおよび内蔵プレビュー画像の抽出

JPEGファイル内に隠されたサブ画像を抽出する機能は、Chama Opticsのパフォーマンスに決定的な役割を果たします。この機能は [exif-rs PR #58](https://github.com/kamadak/exif-rs/pull/58)（[+1,364行](https://github.com/kamadak/exif-rs/pull/58)、PR #57ベース）で実装しました。

#### JPEGの中に隠された画像たち

1つのJPEGファイルの中には実際に複数の画像が入っている可能性があります。

![JPEG ファイル構造と MPF サブ画像](jpeg_mpf_structure.svg)

JPEG内に内蔵された画像は大きく3つのソースから抽出できます。

1. **EXIF IFD(1) サムネイル** ―― 標準EXIFサムネイル（通常160×120）
2. **APP2セグメント（MPF）** ―― [CIPA DC-007](https://www.cipa.jp/std/documents/download_e.html?DC-007-Translation-2021-E) 標準で定義されたMulti-Picture Format。メインEOI以降に別の完全なJPEGストリームとして格納される。
3. **MakerNote内部プレビュー** ―― メーカー別非標準プレビュー画像

#### なぜMPFプレビューが重要なのか：メモリとパフォーマンス

写真一覧でサムネイルを表示する際、最も単純な方法は**元画像をロードしてリサイズ**することです。しかしこれはひどく非効率的です。

| 方式 | メモリ使用量 | 処理時間 |
|:---|:---|:---|
| 元画像(24MP)ロード → リサイズ | ~72MB (24M × 3bytes) | 遅い |
| IFD(1)サムネイル使用 | ~76KB (160×120) | 速いが、小さすぎてぼやける |
| **MPFプレビュー(~2MP)使用** | **~8MB** | **速く、視覚的に十分** |

IFD(1)のサムネイルは小さすぎて一覧用には問題ありませんがプレビュー用にはぼやけます。元画像をロードすると24MP画像がメモリに~72MBを占有しデコード時間も長いです。**MPFに含まれる1~2MPプレビュー画像**はこの両者の間のスイートスポットです ―― 視覚的に十分鮮明でありながら、メモリとCPUオーバーヘッドが元画像の1/10以下です。

特にChama Opticsのように**数十枚の写真を同時にリスト表示し、テーマプレビューまで提供**するプログラムではこの差が決定的です。50枚の24MP写真を元画像でロードすると~3.6GB、MPFプレビューでロードすると~400MB ―― 約9倍の差です。

既存のexif-rsユーザーに影響を与えないよう `mpf` feature flagで提供するようにしました。

### 10. HEIF/HEICデコーダ：プラットフォーム別戦略

最近JPEG以外にHEIF（High Efficiency Image Format）で写真を保存するデバイスが増えています。特にiOSは撮影時にHEIFをデフォルトで使用し、写真を外部に転送する際にJPEGに変換するかHEIFのまま渡すかをOSが自律的に判断します ―― アプリからこれを制御するのは容易ではありません。一部のミラーレスカメラ（Sony、Canonなど）もHEIF撮影をサポートし始めました。互換性のためにJPEGだけを使うユーザーもいますが、HEIFで入ってくるファイルを処理できなければ写真アプリとして致命的です。問題はHEIFデコードのサポートがプラットフォームごとに大きく異なることです。

Chama Opticsは**可能な限りOSネイティブデコーダを使い、ネイティブサポートのないプラットフォームでのみlibheifを使う**戦略を取りました。

![HEIF デコーダ戦略](heif_decoder_strategy.svg)

**iOS/macOS** ―― AppleのImageIO フレームワークがHEIFをネイティブサポートします。外部ライブラリなしにOS APIだけでデコードが可能です。iOSではSwiftアプリレイヤーでデコードしたピクセルバッファをC FFIでRustに渡し、macOSではRustが直接macOS APIバインディング経由で呼び出します。

**Android** ―― API 26（Android 8.0）以上でBitmapFactoryとMediaCodecがHEIFをネイティブサポートします。Kotlinアプリでデコードした後JNA経由でRustに渡します。

**Windows/Linux** ―― ネイティブHEIFデコーダがないか制限的です。この場合 [libheif_rs](https://crates.io/crates/libheif-rs)（libheifのRustバインディング）を使用します。C FFIはlibheif_rsが内部的に処理するため、Rustコードからは安全なAPIのみ呼び出せばよいです。libheifは内部的にlibde265（HEVCデコーダ）とlibaom（AV1/AVIF）を使用します。

この戦略の核心は `#[cfg(target_os)]` 条件付きコンパイルです。ネイティブデコーダがあるプラットフォームでは外部依存なしに最適なパフォーマンスを得て、libheif_rsが必要なプラットフォームでのみリンクします。結果的にmacOSビルドではlibheif関連コードはそもそもコンパイルされません。

### 11. テーマパラメータシステム：Rust → JSON → プラットフォーム別UI

Chama Opticsのテーマには40以上の設定パラメータがあります ―― フォントweight、ウォーターマークの位置・透明度、フレームスタイル、ロゴ表示有無、色、余白など。これらのパラメータがデスクトップとモバイルで**同一の結果**を保証しなければならないことが核心的な要件でした。

![テーマパラメータシステム](theme_param_json.svg)

デスクトップ（egui）ではRust構造体を**直接参照**してUIウィジェットを描画します。`Slider::new(&mut config.font_weight, 100..=900)` のようなコードで構造体フィールドがそのままUI状態になります ―― JSONシリアライゼーションも中間変換もありません。

問題はモバイルです。iOS（SwiftUI）とAndroid（Jetpack Compose）はRust構造体に直接アクセスできません。40以上のパラメータそれぞれについてSwift/Kotlin側で手動でUIを作り、FFIで値をやり取りするコードを一つ一つ書くとしたら？　パラメータが一つ追加されるたびにRust、Swift、Kotlinの三箇所を同時に修正しなければなりません。

この問題を `proc_macro` で解決しました。Rust構造体の定義に `#[derive(ThemeParam)]` を付けると、コンパイルタイムに以下が自動生成されます。

- **JSONスキーマ**: 各フィールドのUI型（スライダー、トグル、enum選択、カラーピッカーなど）、範囲、デフォルト値を含むJSON
- **FFI関数**: `get_param_json()`、`set_param()` などモバイルから呼び出せるC ABI関数
- **デシリアライゼーションロジック**: JSONで受け取った値をRust構造体に反映するコード

モバイルアプリはこのJSONをパースして**ネイティブUI要素を動的に生成**します。`"type": "slider"` → SwiftUIの `Slider`、Jetpack Composeの `Slider()`。`"type": "toggle"` → `Toggle` / `Switch`。ユーザーが値を変更するとFFI経由でRustコアに渡され、Rustコアは同一のレンダリングパイプラインで結果を返します。

結果的にRust構造体一つが**UI仕様でありデータモデルでありシリアライゼーションフォーマット**の役割を同時に果たします。新しいパラメータを追加する際はRustにフィールド一つを追加してアトリビュートでUIヒントを付ければ、proc_macroがJSONスキーマを更新し、モバイルアプリは次回ビルドで自動的にそのUIを表示します。Swift/Kotlinコードを修正する必要がありません。

組み込みでの `build.rs` 乱用と `const fn` 執着がこのような形で応用されました。正直proc_macroでやるのがスマートかどうかは分かりません ―― 本人も「異様だ」と思っている部分です。しかし**40以上のパラメータを3つのプラットフォームで手動同期するよりは確実にマシです。** Rustの手続き型マクロ（procedural macro）についてもっと知りたければ [この記事](https://priver.dev/blog/rust/procedural-macros/) が良い参考になります。

### 12. 多言語翻訳システム：YAML一つで3プラットフォームの翻訳を自動生成

4つの言語（英語、韓国語、日本語、インドネシア語）をサポートしながら**翻訳文字列が3つのプラットフォームで同期されなければなりません。** 翻訳キーを一つ追加したり文言を修正するたびにiOSの `.strings`、Androidの `strings.xml`、デスクトップのRustコードをそれぞれ手で直さなければならないとしたら？　結局漏れたりずれたりします。

解決方法はシンプルです。**rust-coreのYAMLファイルを唯一の原本とし、ビルド時に各プラットフォームのフォーマットに自動変換する**ことです。

![i18n ビルドパイプライン](i18n_build_pipeline.svg)

#### YAML：翻訳の原本

`rust-core/locales/` ディレクトリに23個のYAMLファイルがあります。`common.yml`、`gallery.yml`、`theme.yml`、`face_detection.yml` など機能単位で分割されており、合計約3,900行です。

```yaml
# rust-core/locales/gallery.yml
gallery:
  empty_state_title:
    en: "No Images Yet"
    ko: "이미지 없음"
    ja: "画像がありません"
    id: "Belum Ada Gambar"
```

この構造は `rust_i18n` クレートが要求するフォーマットそのままです。デスクトップでは `rust_i18n::i18n!("locales")` でコンパイルタイムにYAMLを埋め込み、`t!("gallery.empty_state_title")` で呼び出します。別途の変換は不要です。`build.rs`で `cargo:rerun-if-changed=locales` を宣言してあるため、YAMLが修正されると自動的に再コンパイルされます。

問題はiOSとAndroidです。

#### iOS: generate_ios_strings.sh

iOSは `NSLocalizedString` と `.strings` ファイルを使用します。`generate_ios_strings.sh` はPython3 + PyYAMLでYAMLをパースし、各ロケール別の `Localizable.strings` を生成します。

```bash
# build_ios.sh から自動呼び出し
./generate_ios_strings.sh
```

YAMLの階層構造をドット（`.`）表記法で平坦化して `.strings` フォーマットに変換します。

```
/* Auto-generated from rust-core/locales - DO NOT EDIT */
"gallery.empty_state_title" = "이미지 없음";
"common.actions.save" = "저장";
```

iOS固有の要件もありました。同じキーでもiOSでは異なる文言を使うべき場合があります ―― 例えばデスクトップで「ファイル読み込み」と書く箇所をiOSでは「写真を選択」と書く方が自然です。これに対応するため **`_ios` サフィックスオーバーライド**を実装しました。YAMLで `import.label_ios` が定義されていればiOSビルドでは `import.label` の代わりにその値を使用します。デスクトップとAndroidには影響しません。

このスクリプトは `build_ios.sh` でRustクロスコンパイル前に自動的に呼び出されるため、YAMLを修正してXcodeビルドを実行すれば翻訳が自動反映されます。

#### Android: generate_android_strings.sh

Androidは `strings.xml` と `R.string.*` リソースシステムを使用します。核心的な違いが2つあります。

**第一に、キーフォーマットが異なります。** Androidリソース名にはドット（`.`）を使用できません。YAMLの `gallery.empty_state_title` をAndroidでは `gallery_empty_state_title` に変換する必要があります。

```python
def yml_key_to_android_key(yml_key):
    return yml_key.replace('.', '_')
```

**第二に、ロケールディレクトリ規則が異なります。** Androidはインドネシア語を `id` ではなく `in` で表記します ―― `values-in/strings.xml`。このマッピングをスクリプトで処理します。

```bash
ANDROID_LOCALE_MAP["en"]="values"
ANDROID_LOCALE_MAP["ko"]="values-ko"
ANDROID_LOCALE_MAP["ja"]="values-ja"
ANDROID_LOCALE_MAP["id"]="values-in"    # Android uses "in" for Indonesian
```

iOSスクリプトとのもう一つの違いは**diff基盤同期**だということです。iOSは毎回ファイルを丸ごと上書きしますが、Androidスクリプトは既存の `strings.xml` に既にあるキーには触れず**不足しているキーのみ追加**します。Android側で手動管理しているエントリ（アプリ名など）を保持するためです。`--check` モードで実行するとファイルを修正せず不足している翻訳のみ報告します。

Androidでこれらのキーを実際に使用する際は `ThemeI18n.kt` でYAMLドット表記法キーを `R.string.*` リソースIDにマッピングします。

```kotlin
object ThemeI18n {
    fun translate(context: Context, key: String): String {
        val resourceId = keyToResourceId(key)
        return if (resourceId != 0) context.getString(resourceId) else key
    }
}
```

#### 3プラットフォームのキー変換比較

| 要素 | Desktop (Rust) | iOS (Swift) | Android (Kotlin) |
|:---|:---|:---|:---|
| **原本** | `t!("gallery.empty_state_title")` | `NSLocalizedString("gallery.empty_state_title")` | `R.string.gallery_empty_state_title` |
| **キー区切り** | `.`（ドット） | `.`（ドット） | `_`（アンダースコア） |
| **生成方式** | コンパイルタイム埋め込み | ビルドスクリプト自動生成 | ビルドスクリプトdiff同期 |
| **プラットフォームオーバーライド** | — | `_ios` サフィックス | — |
| **インドネシア語コード** | `id` | `id` | `in` |

この構造のおかげで翻訳を追加または修正する際に**YAMLファイル一つを直すだけで** 3つのプラットフォーム全てに反映されます。23個のYAMLファイル、4つの言語、3つのプラットフォームを手動で同期するのは現実的に不可能です ―― 自分が考えた方法は自動化だけでした。

---

## オープンソース貢献活動

Chama Opticsの開発過程で依存するオープンソースプロジェクトにも積極的に貢献しました。

| プロジェクト | 貢献内容 |
|:---|:---|
| [exif-rs](https://github.com/kamadak/exif-rs) | MakerNoteパース ―― 10メーカーサポート ([PR #57](https://github.com/kamadak/exif-rs/pull/57), +5,946 lines) |
| [exif-rs](https://github.com/kamadak/exif-rs) | MPFおよび内蔵プレビュー画像抽出 ([PR #58](https://github.com/kamadak/exif-rs/pull/58), +1,364 lines) |
| [exif-rs](https://github.com/kamadak/exif-rs) | TIFFフィールドアクセス改善 ([PR #51](https://github.com/kamadak/exif-rs/pull/51), approved) |
| [font-kit](https://github.com/servo/font-kit) | macOSシステムフォント列挙時のメモリ暴走修正 ([PR #271](https://github.com/servo/font-kit/pull/271)) |
| [egui](https://github.com/emilk/egui) | variable fontロード時のmain weight設定機能改善 ([PR #7790](https://github.com/emilk/egui/pull/7790), approved) |
| [wagahai-lut](https://github.com/pmnxis/wagahai-lut) | 1D/3D LUTカラーグレーディングライブラリ ([crates.io](https://crates.io/crates/wagahai-lut)) |

---

## リリースまとめ

| バージョン | 日付 | 主な変更点 |
|:---|:---|:---|
| v0.1.0 | 2025-10-19 | 初プレリリース、macOS/Windows、Filmテーマ |
| v0.1.1 | 2025-10-19 | 日本語翻訳、一括保存、プレフィックス/サフィックス |
| v0.1.2 | 2025-10-27 | Glowエフェクト、Film Date/Glowテーマ |
| v0.1.3 | 2025-11-03 | ウォーターマーク（9箇所）、フォント選択 |
| v0.1.4\~5 | 2025-11-05\~12 | Just Frame、Strapテーマ、カメラロゴ |
| v0.1.6 | 2025-11-24 | Monitor、Lightroomテーマ、Longsideスケール |
| v0.1.7 | 2025-12-19 | One/Two Line、Shot Onテーマ、CJK修正、PhotoStyle |
| v0.1.8 | 2025-12-27 | タブUI、グループ化、テーマプレビュー、マルチコア |
| v0.1.9 | 2026-02-04 | 顔認識、LUTカラーグレーディング、**iOS TestFlight初配布** |

---

## AIと共にプログラミング（バイブコーディング？）

最初はAIコーディングに対して懐疑的でした。

そのためデスクトップ版の大半は今でも自分の手書きコード + `cargo fmt/clippy/check` に依存しています。Rust Embeddedでの習慣である `const` の乱用をデスクトップでも適用しようとする自分の意図を、AI（Claude）はまだ正しく把握できていません。

しかしモバイル版を開発しながら、**一人でこれを全部やるのは正気の沙汰じゃない**と思いました。既存のデスクトップ版と同一の結果をモバイルで提供することが最優先であり、ネイティブAPIとRust FFIを直接呼び出す形が多くなると予想されました。

そんな折、友人の結婚式の招待状の集まりに行く際、一緒に車に乗っていた友人が「**FlutterでiOS 26のLiquid UIが今まともに動かない**」と言いました。開発するとしてもネイティブでのみ開発すべきだという考えが固まり、同時にAIにモバイル用のコードを任せることに決めました。

結果として **Rustコアは自分が直接**、**モバイルUI（SwiftUI/Jetpack Compose）とFFIブリッジはAIと共に** 書くという分業体制が生まれました。Rust側は自分の意図したパターンとスタイルがあるのでAIがうまく合わせられませんが、Swift/Kotlinのように自分がよく知らない言語でプラットフォームネイティブコードを書くにはAIが大いに助けになりました。

この経験をした後、Flutterのようなクロスプラットフォームフレームワークの立ち位置について考えるようになりました。もちろんFlutterやReact Nativeが解決する問題 ―― 一つのコードベースで複数プラットフォームをカバーすること ―― は依然として有効です。しかしAIが各プラットフォームのネイティブコードを十分にうまく書ける時代になれば、「ネイティブを知らないからクロスプラットフォームを選ぶ」という動機は徐々に弱まっていくかもしれません。モバイル開発をまったく知らなかった自分がAIの助けだけでSwiftUIとJetpack Composeをそれぞれネイティブで書けたという事実が、その可能性を示す一つの事例ではないかと思います。

---

## 今後の計画

v0.1.9を起点にデスクトップ単独リリースは終了し、v0.2.0からはiOS、Androidモバイルアプリと一緒にリリースする予定です。追加機能よりは随時安定化とテーマ追加に集中するつもりです。

当面の目標は **2026年3月のホロライブエキスポ/フェスティバル** で実戦投入することです。会場でミラーレスで写真を撮り、iPhoneですぐフレームをかぶせ、顔を自動でモザイク処理してSNSに上げる ―― カメラユーザーでも一般のスマホユーザーでも、それぞれの環境で快適に使えるワークフローを提供するのが方向性です。

サイドプロジェクトとしてChama Opticsは「写真を撮る人が写真をより良く見せられるようサポートするツール」を目標に、もっと快適なワークフローを提供していきます。そして今回の開発で蓄積した手続き型マクロと最適化の経験をもとに、再びRust Embedded方面にももう少し力を入れていく予定です。

---

## 参考文献および引用

#### 標準文書
- CIPA DC-008 — [Exchangeable image file format (Exif) Version 3.0](https://www.cipa.jp/std/documents/download_e.html?DC-008-Translation-2023-E)
- CIPA DC-007 — [Multi-Picture Format (MPF)](https://www.cipa.jp/std/documents/download_e.html?DC-007-Translation-2021-E)
- Adobe — [Adobe Cube LUT Spec 1.0](https://web.archive.org/web/20220220033515/https://wwwimages2.adobe.com/content/dam/acom/en/products/speedgrade/cc/pdfs/cube-lut-specification-1.0.pdf)

#### ライブラリおよびフレームワーク
- [exif-rs](https://github.com/kamadak/exif-rs) — Rust EXIFパースライブラリ
- [egui](https://github.com/emilk/egui) — Rust即時モードGUIフレームワーク
- [font-kit](https://github.com/servo/font-kit) — クロスプラットフォームフォントロードライブラリ
- [wagahai-lut](https://github.com/pmnxis/wagahai-lut) ([crates.io](https://crates.io/crates/wagahai-lut)) — 1D/3D LUTカラーグレーディングライブラリ
- [libheif-rs](https://crates.io/crates/libheif-rs) — libheif Rustバインディング
- [wide](https://crates.io/crates/wide) — クロスプラットフォームSIMDベクトルクレート
- [ONNX Runtime](https://onnxruntime.ai/) — クロスプラットフォームML推論エンジン
- [InsightFace SCRFD](https://github.com/deepinsight/insightface/tree/master/detection/scrfd) — 顔検出モデル (det_10g)
- [exif-frame](https://github.com/jeonghyeon-net/exif-frame) — EXIFフレームWebツール（Chama Opticsの初期参考）

#### 参考資料
- [Rust Procedural Macros](https://priver.dev/blog/rust/procedural-macros/) — 手続き型マクロ参考
- [ExifTool TagNames](https://exiftool.org/TagNames/) — メーカー別MakerNoteタグ参考
- [Exiv2 MakerNote Documentation](https://exiv2.org/makernote.html) — MakerNote構造およびメーカー別フォーマット参考

---

## Special Thanks

- [SkuldNorniern](https://github.com/SkuldNorniern) — デバッグおよび顔認識関連の支援
- [miniex](https://github.com/miniex) — フォントシステムのデバッグおよび顔認識関連の支援
- [jcm7612](https://github.com/jcm7612) — デバッグおよびフィードバック
- [shiemika324](https://x.com/shiemika324) — イラストおよびアイコンイラスト提供
