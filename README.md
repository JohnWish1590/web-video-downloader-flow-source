# Web Video Downloader Flow

A Chrome extension that finds video media on the page you are viewing, lets you pick a quality, and saves it to your own computer. This repository is the **corresponding-source offer location** for the extension and holds the public documentation, the privacy policy and the per-version source delivery archive.

Current version: **1.1.7** · [Changelog](CHANGELOG.md) · [Privacy policy](PRIVACY.md)

## How it is priced

- **Free to install. No account required.**
- An unregistered user gets **3 completed downloads**. After the third one, a **1 hour countdown** starts; when it reaches zero the counter resets and you get 3 more.
- **Registration is optional.** It removes the countdown entirely. One-time payment: **$6.99** through Stripe, or **¥28** through WeChat. Prices are shown in the extension's settings page and are the authoritative values.
- The counter is enforced by the license service, not by the extension alone, so it is not something you can reset by reinstalling. A download is only counted when it actually completes; a cancelled download does not consume quota.
- The extension works without ever contacting the license service if you never start a download — media detection happens locally in your browser.

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
| **YouTube** | Adaptive streams via the player's own format list, with the real title, channel and thumbnail taken from the player response; separate video and audio are muxed into one file. |
| **Bilibili** | Complete quality list for the signed-in account, filled in from the page's own same-origin `playurl` result. Same-size `avc1`/`hvc1`/`av01` encodes collapse into one row. Sizes come from real probes. |
| **X / Twitter** | Videos from posts already loaded on the page, with a download entry in the top-right corner of each post. The main video and comment videos are distinguished. |
| **Weibo** | Videos from posts already loaded on the page, same top-right entry, main post first and comments collapsed by default. |
| **TikTok** | Every resolution the platform handed the page in its own player state (`bitrateInfo`), with the separate music track muxed in so the file keeps its audio. |
| **Douyin / 抖音** | Same idea, read from the detail response the page itself requested. |

For TikTok and Douyin an empty quality list means the page has not fetched its data yet — the menu says so and offers a retry rather than inventing a single stream.

### HLS and DASH

- HLS: segments, AES-128 encrypted segments, init segments, separate audio tracks, quality switching on a shared timeline, signed and redirected addresses.
- Static, unencrypted DASH: separate video/audio AdaptationSets, quality selection, safe range validation.
- Explicitly refused rather than attempted: DRM, paywalls, login-only media, dynamic/live manifests, and `SegmentBase` timelines that cannot be determined safely.

### Output format

HLS, DASH and adaptive sources are muxed locally and saved as **MKV**. Direct MP4 sources are saved as **MP4**. Remuxing runs in an isolated worker from LibAV.js / FFmpeg built from the sources recorded in the source package; nothing is downloaded at runtime.

## What it does not do

- It does not bypass DRM, paywalls, login restrictions or any access control.
- It does not download private, paid, or account-restricted media you would otherwise not be able to watch.
- It does not execute remote code. All JavaScript, WASM and UI ship inside the package.
- It does not collect analytics, build a browsing history, or send page content or downloaded media to the developer.

Save only media you have the right to save. Site structure, sign-in state and network conditions change over time and affect detection.

## Install

Chrome Web Store listing: *coming for this version*. Until it is published, load the released package as an unpacked extension:

1. Open `chrome://extensions` and enable Developer mode.
2. Choose **Load unpacked** and select the unpacked extension directory from the release archive.
3. Reload the page you want to download from.

Do not load two builds of this extension at the same time; they share in-page element names and their buttons may collide.

## Corresponding source

Each release ships the exact source used by that runtime. Download the archive from the immutable GitHub Release:

- [WebVideoDownloaderFlow-corresponding-source-1.1.7.zip](https://github.com/JohnWish1590/web-video-downloader-flow-source/releases/download/v1.1.7/WebVideoDownloaderFlow-corresponding-source-1.1.7.zip)
- SHA-256: `93251178f87162562296f86e82a04f60ac60e288263ca2e2f16b708029c8c773` — the published asset was downloaded back and matches this value.

Every release also carries an `ARCHIVE-HASHES.json` record. The archive embeds `README.md`, `CHANGELOG.md` and `PRIVACY.md` as of this version. The archive contains the application source used by the runtime, recorded third-party source materials, build records, license notices and the corresponding-source delivery record. It is not the Chrome Web Store upload package and it is not the private development workspace.

Rebuilding from that archive is documented in `BUILD-FROM-SOURCE.md` inside it.

## Privacy

- [Privacy policy](PRIVACY.md)

Summary: media discovery and download processing happen in your browser. If you use the free quota or register, the extension sends a random client identifier, a random device identifier, an authorization token and a short operation identifier to the license service over HTTPS. It does not send page content or downloaded media. Stripe handles card details on its own checkout page.

## License and third-party components

Application code in this repository and in the release archives is provided for the corresponding-source obligation and for transparency. Third-party components (LibAV.js, FFmpeg, emfiberthreads, `m3u8-parser` and others) remain under their own licenses; see the `LICENSES/` directory and `THIRD-PARTY-NOTICES.md` inside each source archive.

This repository does not contain secrets, private credentials, the development workspace or the Chrome Web Store upload package.

The release record is an engineering delivery record, not legal advice.
