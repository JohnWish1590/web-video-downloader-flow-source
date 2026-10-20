# Web Video Downloader Flow Privacy Policy

Effective date: 2026-10-18

Web Video Downloader Flow helps a user discover video media on the webpage they are viewing, choose an available quality, and save media that the user is authorized to save. This policy describes the extension and its optional license service.

## Information handled by the extension

The extension may process the following information to provide its visible downloader features:

- video elements, media URLs, titles, dimensions, sizes, thumbnails, and related request metadata on the current webpage;
- the current page URL or referrer when it is required to request media selected by the user;
- language/theme settings, short-lived task state, and temporary media metadata stored in Chrome storage;
- a random client identifier, a random device identifier, a license key entered by the user, an authorization token, and a short download operation identifier when the optional access service is enabled.

On TikTok and Douyin the packaged page script reads player data that the site already delivered to the page, in order to list the qualities the site itself offers. It does not sign requests, read cookies or account tokens, or bypass a login or paywall, and that page data is not sent to the developer.

Page media discovery and download task processing are performed for the user-facing downloader feature. The extension does not build an analytics history, sell browsing activity, or send page content or downloaded media to the developer's authorization service.

## Optional authorization service

When the user uses the free quota or enters an optional license key, the extension communicates over HTTPS with the Web Video Downloader Flow Cloudflare Worker. The service receives the product identifier, random client/device identifiers, the submitted license key during activation, an authorization token when present, and the minimum status/operation information required to enforce the free quota and registered access. It does not receive the media file downloaded by the user or the full contents of the webpage.

The extension stores the returned authorization token and cached status locally in Chrome storage. Removing the extension or clearing its site/extension data removes those local values. Server-side license and transaction records may be retained as needed to prevent reuse, provide support, secure the service, meet accounting obligations, or comply with law.

## Payments and manual support

Stripe processes payment details on its own checkout page. The extension does not request or store card numbers, bank credentials, or other payment credentials. A user choosing manual WeChat payment may voluntarily email a payment screenshot and an address for delivery of a license key; those materials are used only to confirm the purchase and deliver the requested key.

## First-run notice and user responsibility

Before the extension writes any file, it shows a notice that must be acknowledged. The notice explains what the extension reads (listed above), the free quota, and that the user starts each download and is responsible for it: only content the user is entitled to save may be downloaded, the user must follow the terms of service of the sites they visit and the copyright law that applies to them, and the developer accepts no liability for downloaded content or its subsequent use. The acknowledgement is stored locally as a timestamp and is not sent anywhere.

## Sharing and limited use

We do not sell user data, use it for targeted advertising, or use it to build user profiles. Data is handled only to provide, secure, maintain, and support the downloader and optional authorization feature. We do not share page content or downloaded media with third parties. Requests to the media host are made because the user explicitly selected a media download and are subject to that host's terms.

## Security

Data sent to the optional authorization service is sent over HTTPS. Secrets used by the authorization service remain on the server and are not bundled in the extension. The extension does not execute remote JavaScript, WASM, or other downloaded code.

## User choices and contact

The extension can be used without registration. Users may avoid the optional license service and may remove the extension at any time. For privacy questions or a request about server-side license records, contact `cheung.cn@gmail.com` and include only the information needed to locate the request.

## English summary

Web Video Downloader Flow processes webpage media metadata and user-selected download requests to provide its downloader UI. Optional authorization sends random client/device identifiers and license information to the configured Cloudflare Worker over HTTPS; it does not receive page content or downloaded media. Stripe handles card details on its own checkout page. We do not sell data, use it for targeted advertising, or execute remote code. Contact `cheung.cn@gmail.com` for privacy questions.
