---
title: "Chama Optics Privacy Policy"
date: 2026-02-16T00:00:00+09:00
draft: false
---

**Effective Date: February 25, 2026**

> This page is managed via GitHub Pages automatic deployment. Any changes to this Privacy Policy can be verified through the commit history of the [GitHub repository](https://github.com/pmnxis/pmnxis.github.io).

Chama Optics (hereinafter "the App") is a photo post-processing application developed and operated by Jinwoo Park (hereinafter "the Developer"). This Privacy Policy explains how your personal information is handled when using the App.

---

## 1. Information We Collect

**Chama Optics does not collect, store, or transmit any personal information.**

- The App does not require account creation, sign-up, or login.
- We do not collect any personally identifiable information such as names, email addresses, phone numbers, or location data.
- We do not collect information for analytics, advertising, or tracking purposes.
- We do not collect device identifiers, IP addresses, or cookies.

---

## 2. App Permissions and Purpose of Access

Chama Optics may request the following device permissions to provide its photo post-processing features.

| Permission | Purpose | Data Transmission |
|---|---|---|
| Photo Library Access | To load photos selected by the user for applying frames, mosaic, and other post-processing effects | None (on-device processing) |
| Storage Access | To save processed photos to the device | None (on-device storage) |

- Requested permissions are used solely for the App's core functionality (photo post-processing).
- The App does not request access to camera, microphone, contacts, or location.

---

## 3. Photos and Image Data

- Photos selected by the user are **processed entirely on the device**.
- Photo data is never transmitted to external servers.
- EXIF data (camera model, lens, shooting settings, etc.) is read locally on the device for frame generation only and is never transmitted externally.
- Processed photos are saved to the user's device storage and can be managed (deleted, etc.) directly by the user.

---

## 4. Face Detection Processing

Chama Optics provides a feature that automatically detects faces in photos and applies privacy-protection effects such as mosaic (pixelation), stickers, or border overlays.

### 4.1 What Face Data Is Collected

- The App collects **only face bounding box coordinates** (x position, y position, width, height) — the rectangular area indicating where a face is located in the image.
- **No facial landmarks** (eyes, nose, mouth positions) are collected.
- **No facial feature embeddings or biometric identifiers** are collected.
- **No identity information** is collected — the App performs face **detection** (locating faces) only, not face **recognition** (identifying individuals).

### 4.2 How Face Data Is Used

- Face bounding box coordinates are used **solely** to apply user-selected privacy-protection effects: mosaic/pixelation, border stroke, and sticker overlays.
- These effects help users obscure faces in photos before sharing on social media.
- Face detection results are **never** used to identify, profile, or track individuals.

### 4.3 Where Face Data Is Stored

- **All face detection processing is performed entirely on-device.**
- Face bounding box coordinates are stored **locally on the device** as part of the editing session state (in the app's local storage) so that the user can resume editing.
- Face data is **never transmitted** to any external server, cloud service, or third party.

### 4.4 Third-Party Sharing of Face Data

- Face data is **never shared** with any third party.
- The App contains no advertising SDKs, analytics SDKs, or any network code that could transmit face data.

### 4.5 Face Data Retention

- Face bounding box coordinates are retained locally on the device only for as long as the corresponding image remains in the App's gallery.
- Face data is **permanently deleted** when:
  - The user removes the image from the App's gallery, or
  - The user uninstalls the App.
- No face data is retained on any server at any time.

### 4.6 Technologies Used by Platform

- **iOS**: Apple Vision Framework (`VNDetectFaceRectanglesRequest`) — on-device processing
- **Android**: Google ML Kit Face Detection — on-device processing, no data sent to Google servers
- **Desktop (macOS/Windows/Linux)**: ONNX Runtime + SCRFD model — on-device processing

> **Note for Android users**: Google ML Kit Face Detection operates entirely on-device. Face images and detection results are not transmitted to Google servers. For more details, see the [Google ML Kit Terms of Service](https://developers.google.com/ml-kit/terms).

---

## 5. Data Security

- The App does not collect personal information, so no user data is transmitted externally.
- All photo processing is performed on the user's device, with no communication to external servers.
- The core libraries of the App are open source and available on [GitHub](https://github.com/pmnxis/chama-optics) for transparency.

---

## 6. Data Retention and Deletion

- Chama Optics does not store any user data on servers.
- Face bounding box coordinates are stored locally as part of the editing session and are deleted when the user removes the image from the App or uninstalls the App (see Section 4.5).
- Processed result images are saved to the user's device storage and can be deleted by the user through the device's file management features.
- Uninstalling the App removes all local data associated with it, including any stored face coordinates.

---

## 7. Network Usage

- Chama Optics does not require an internet connection for photo processing.
- The App operates offline by default.
- Network communication initiated by the platform (App Store, Google Play) for features such as update checks is governed by the respective platform's policies.

---

## 8. Third-Party Sharing

- Chama Optics does not sell, provide, or share user data with any third parties.
- The App does not include any third-party advertising SDKs, analytics SDKs, social login SDKs, or similar services.
- Google ML Kit used in the Android version operates on-device only and does not transmit user data to Google.

---

## 9. Children's Privacy

- Chama Optics is not directed at children under the age of 13.
- Since the App does not collect any personal information from any user, there is no separate collection process for children.

---

## 10. Changes to This Privacy Policy

- If this policy is updated, the changes will be posted on this page and the effective date will be updated.
- For significant changes, notice will be provided through app updates or the GitHub repository.

---

## 11. Contact

If you have any questions or concerns regarding this Privacy Policy, please contact us at:

- **Developer**: Jinwoo Park
- **GitHub**: [https://github.com/pmnxis/chama-optics](https://github.com/pmnxis/chama-optics)
- **Email**: pmnxis@gmail.com
