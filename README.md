# Web Video Downloader Flow

A Chrome extension that finds video media on the page you are viewing, lets you pick a quality, and saves it to your own computer. It also saves the original photos from loaded X and Weibo posts. This repository is the **corresponding-source offer location** for the extension and holds the public documentation, the privacy policy and the per-version source delivery archive.

Current corresponding source release: **1.1.24** · Previous published source: **1.1.23** · [Changelog](CHANGELOG.md) · [Privacy policy](PRIVACY.md)

## How it is priced

- **Free to install. No account required.**
- An unregistered user gets **3 completed video downloads**. After the third one, a **1 hour countdown** starts; when it reaches zero the counter resets and you get 3 more.
- **X and Weibo photo downloads are free and do not use video quota**, including during a video cooldown.
- **Registration is optional.** It removes the countdown entirely. One-time payment: **$6.99** through Stripe, or **¥28** through WeChat. Prices are shown in the extension's settings page and are the authoritative values.
- The license service enforces the free quota against a random client ID stored locally by the extension. Clearing extension data or reinstalling may create a new ID and reset the quota; cross-reinstall prevention is not promised. A download counts only when it completes; cancelling a download does not consume quota.
- Media detection happens locally in your browser. Photo downloads do not contact the license service.

## What it does

### Works on any page

- Detects media elements and media requests on the page you are on, for ordinary HTML5 video and audio.
- Lists the qualities that were actually observed, with real resolution and size when the server provides them. Missing data is shown as unknown rather than guessed.
- Saves directly to Chrome's download location through the extension's own offscreen executor — no intermediate save dialog, no half-written file left behind if the download fails or is cancelled.
- Pause, resume and cancel from the panel; task state survives closing the popup and reloading the page.
- One download button on the page player, with the quality menu right next to it, and a short confirmation when the download starts.
- Re-clicking a video that is already downloading is refused instead of creating a second task.

### Platform support

| Platform | What you get |
| --- | --- |
| **YouTube** | Not supported: no page controls or downloads, including Shorts, embedded players and media CDN sources. |
| **Bilibili** | Complete quality list for the signed-in account, filled in from the page's own same-origin `playurl` result. Same-size `avc1`/`hvc1`/`av01` encodes collapse into one row. Sizes come from real probes. |
| **X / Twitter** | Videos from posts already loaded on the page, with a download entry in the top-right corner of each post. The main video and comment videos are distinguished. Post photos and the foreground photo viewer have a one-click original-size save button. One text entry above the photo group saves the whole post as a ZIP; single-photo and batch photo downloads are free. |
| **Weibo** | Videos from posts already loaded on the page, same top-right entry, main post first and comments collapsed by default. Post photos use the same hover single-photo button and one text batch entry above the photo group as X. |

### 1.1.24 policy remediation

Removed YouTube player extraction and page/card downloads. Background checks reject YouTube sources and stored task retries. Historical 1.1.23 is separately preserved and is not the current store candidate. Bilibili and X/Weibo features remain.

### HLS and DASH

- HLS: segments, AES-128 encrypted segments, init segments, separate audio tracks, quality switching on a shared timeline, signed and redirected addresses.
- Static, unencrypted DASH: separate video/audio AdaptationSets, quality selection, safe range validation.
- Explicitly refused rather than attempted: DRM, paywalls, login-only media, dynamic/live manifests, and `SegmentBase` timelines that cannot be determined safely.

### Output format

HLS, DASH and adaptive sources are muxed locally and saved as **MKV**. Direct MP4 sources are saved as **MP4**. Remuxing runs in an isolated worker from LibAV.js / FFmpeg built from the sources recorded in the source package; nothing is downloaded at runtime.

X and Weibo photos retain the image format and original response bytes. “Original” means the largest version the platform currently provides, not a guarantee of the author's untouched camera file. An unavailable original is reported instead of silently saving a preview. Single photos use the normal Chrome download path; a post's batch entry creates one ZIP and asks for the save location once. The ZIP name uses the author, publication date and the first 12 characters of the post text; missing dates are omitted and empty text becomes `图片`. Video downloads keep their automatic filenames and normal save behavior.

## First run and your responsibility

On first install the extension opens a notice that has to be acknowledged before any file is written. It covers what the extension reads, the free quota, and this:

- **You start the download, and the download is your responsibility.** The extension only writes a file when you click; it does not decide who owns the content and does not obtain any permission for you.
- Only download what you are entitled to save: your own work, content that is explicitly offered for download, or a use the law permits.
- You are responsible for following the terms of service of the sites you visit and the copyright law that applies to you. Saving, sharing or republishing someone else's work without permission can infringe their rights.
- The developer accepts no liability for the content you download or for how you use it.

The same text is available again from the settings page at any time.

## What it does not do

- It does not bypass DRM, paywalls, login restrictions or any access control.
- It does not download private, paid, or account-restricted media you would otherwise not be able to watch.
- It does not execute remote code. All JavaScript, WASM and UI ship inside the package.
- It does not collect analytics, build a browsing history, or send page content or downloaded media to the developer.

Save only media you have the right to save. Site structure, sign-in state and network conditions change over time and affect detection.

## Install

1.1.24 is the remediation candidate after the 1.1.23 Blue Zinc rejection. Source publication and store approval are separate. For local testing, load the fixed directory:

1. Open `chrome://extensions` and enable Developer mode.
2. Choose **Load unpacked** and select the fixed local `chrome-extension/` folder from the development workspace.
3. Reload the page you want to download from.

Do not load two builds of this extension at the same time; they share in-page element names and their buttons may collide.

## Corresponding source

Each release ships the exact source used by that runtime. The corresponding source for this candidate is v1.1.24; links become available on publication:

- [WebVideoDownloaderFlow-corresponding-source-1.1.24.zip](https://github.com/JohnWish1590/web-video-downloader-flow-source/releases/download/v1.1.24/WebVideoDownloaderFlow-corresponding-source-1.1.24.zip)
- SHA-256: recorded in the release asset `ARCHIVE-HASHES.json` and verified after downloading the public asset.

Historical v1.1.23 contains YouTube functionality removed in 1.1.24 and is not the current store build. Earlier archives stay available for the versions they were built for. v1.1.7 and earlier still contain the Douyin and TikTok implementation that was removed in 1.1.8. The v1.1.24 source archive is the corresponding-source snapshot for the current runtime and includes the recorded archive hashes.

Every release also carries an `ARCHIVE-HASHES.json` record. The archive embeds `README.md`, `CHANGELOG.md` and `PRIVACY.md` as of this version. The archive contains the application source used by the runtime, recorded third-party source materials, build records, license notices and the corresponding-source delivery record. It is not the Chrome Web Store upload package and it is not the private development workspace.

Rebuilding from that archive is documented in `BUILD-FROM-SOURCE.md` inside it.

## Privacy

- [Privacy policy](PRIVACY.md)

Summary: media discovery and download processing happen in your browser. If you use the free quota or register, the extension sends a random client identifier, a random device identifier, an authorization token and a short operation identifier to the license service over HTTPS. It does not send page content or downloaded media. Stripe handles card details on its own checkout page.

## License and third-party components

Application code in this repository and in the release archives is provided for the corresponding-source obligation and for transparency. Third-party components (LibAV.js, FFmpeg, emfiberthreads, `m3u8-parser` and others) remain under their own licenses; see the `LICENSES/` directory and `THIRD-PARTY-NOTICES.md` inside each source archive.

This repository does not contain secrets, private credentials, the development workspace or the Chrome Web Store upload package.

The release record is an engineering delivery record, not legal advice.
