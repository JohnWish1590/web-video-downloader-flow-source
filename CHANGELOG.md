# Changelog · Web Video Downloader Flow

All notable changes to this product. The extension version matches `manifest.json`.

Product model, unchanged since 1.1.0:

- **Free to install. No account needed.** An unregistered user gets **3 completed downloads**, then a **1 hour cooldown**, then 3 more. This is enforced server-side (`FREE_DOWNLOAD_LIMIT = 3`, `FREE_COOLDOWN_SECONDS = 3600`).
- **Registration is optional** and removes the cooldown. One-time payment: `$6.99` via Stripe, or `¥28` via WeChat.

---

## 1.1.7 — 2026-10-19

TikTok and Douyin quality lists, and the fixes found while wiring them up.

### Added

- **TikTok support** (`video-bridge.js`, `video-content.js`). The page's own `__UNIVERSAL_DATA__` player state carries `bitrateInfo`; the extension lists every resolution the platform already handed the page, merges duplicate encodes of the same size, and pairs the separate music track so the download keeps its audio.
- **Douyin support** (`video-bridge.js`, `video-content.js`). The bridge reads the detail response the page itself requested, so the quality list comes from the platform instead of a guessed single stream.
- **A download button on the TikTok/Douyin player**, with the platform's real quality list in the menu, plus an audio track that the executor muxes into the output file. When the page has not fetched its data yet, the menu says so and offers a retry instead of silently offering one stream.
- `npm run test:douyin-bridge` — end-to-end check for the Douyin path.

### Fixed

- **The TikTok/Douyin quality list was never visible.** The bridge runs at `document_start` and the page UI at `document_idle`, so every record parsed during page load was posted before anything was listening. The bridge now keeps its latest records and re-sends them when the content script asks for a replay.
- **The bridge never recovered.** `installVideoBridge` existed but was never called, so a tab opened before the extension loaded (or after a service worker restart) had no bridge at all. `check_tab` and `onInstalled` now re-inject per platform.
- **Douyin direct media was filtered out.** The platform filter in `visibleVideos` treated every TikTok/Douyin entry as page-reported-only, which discarded the CDN response the generic path had already captured. X/Weibo stay snapshot-only; TikTok/Douyin keep the direct path as a fallback until the bridge speaks.
- **Free-quota errors on the in-page button showed server text.** The TikTok/Douyin button now translates `FREE_LIMIT_REACHED`, `COOLDOWN`, `LICENSE_KEY_INVALID`, `DEVICE_LIMIT_REACHED` and the access-service failures the same way the panel does.
- **The free-quota accounting could release a live download's reservation.** `releaseExpiredReservations` reset `reserved_count` to zero instead of subtracting only the reservations it expired, so sweeping one abandoned reservation also released every in-flight one — which allowed more than three concurrent downloads. It now subtracts exactly the rows it expired.

### Documented

- Added `CHANGELOG.md` (this file) and rewrote the public README with the full feature matrix, the free/paid model, and the corresponding-source instructions.
- `npm run test:quota` — free-quota accounting driven through the real Worker and the real `schema.sql`, with `node:sqlite` standing in for D1.
- `npm run test:douyin-bridge` — end-to-end check for the Douyin quality list and download path.

### Verified

- `npm run test:baseline` 17/17, `npm run test:replacement` 6/6, `npm run test:boundary` PASS (42 runtime files), `npm run test:generic-page` PASS, `npm run test:quota` PASS.
- Real TikTok page: 1,967,308 bytes downloaded, FFprobe confirms HEVC 720×1280 + AAC, 24.4 s, full decode.
- Douyin: covered by the synthetic bridge test (two qualities listed, non-highest quality selected, byte count matches the served file). A real Douyin page still needs a manual check — headless Chrome hits the platform's captcha page, and bypassing it is not a goal.

---

## 1.1.6 — 2026-10-19

### Fixed

- **Thumbnails still did not render.** The `<img>` used `loading="lazy"`, which in a popup often never starts loading: the element sat at `complete: false`, `naturalWidth: 0`, and never fired `error`, so even the fallback icon stayed hidden. Lazy loading is gone, and a decoded-empty or failed image now falls back to the play icon.
- **Brand icons used as covers.** Some pages expose a site icon as `og:image`. `content.js` now rejects obvious logos and favicons and treats the page as having no cover, so the panel shows the play-icon placeholder instead of a YouTube logo.

---

## 1.1.5 — 2026-10-19

### Fixed

- **Panel title showed the site name and the file was saved as `YouTube.mkv`.** `probeYoutubeFormats` called `changePage()` before registering tracks, which cleared the page title, so `captureTitle()` fell back to the media URL. YouTube's player response already contains `videoDetails.title`, the channel name and `videoDetails.thumbnail`; those are now carried through and outrank the URL fallback, and `watch`/`shorts`/`embed`/`live` were added to the generic title blocklist.
- Bilibili was unaffected: its title arrives from the page context first.

---

## 1.1.4 — 2026-10-19

A full read-through of the runtime (~7,400 lines). The fixes below are real defects, not style.

### Security

- **Removed four unauthenticated message branches** (`download_mkv`, `download`, `dl_complete`, `get_download_progress`) and their call chains. Any iframe's content script could make the extension download an arbitrary address with extension privileges, burn download quota, or rewrite another tab's media state. The `dl_video` compatibility branch in `content.js` and the now-unreachable save window (`social-save.html` / `social-save.js`, `get_social_save`, `openSaveWindow`) were removed with them.

### Fixed

- **Duplicate-download race.** The check and the write were separated by the authorization request and the offscreen start, so two fast clicks both passed. A synchronous in-memory reservation closes the window. `activeTaskForVideo` also used to delete terminal tasks for the same video while the social path called it twice, deleting the task it had just created; deletion was split out into `dropFinishedTasks`, called only before writing a new record. X/Weibo's direct-MP4 path bypassed the guard entirely and is now covered.
- **Cancel between the record write and the executor start** returned "task does not exist", the panel showed a failure, and the download then started anyway. The early cancel is now recorded and the executor stops on start, ending as `cancelled` rather than `error`.
- **Bilibili quality.** The post-probe filter dropped every variant without a resolution, which emptied the list when page context had not arrived yet and made the panel fall back to "quality unknown"; the filter moved into `adaptiveQualityOptions` and only drops entries once a size is known. `get_video_details` replaced the quality list with the probe result instead of merging sizes. `<video>` probes overwrote the declared size with the decoded size (852 → 853) and created duplicates; declared size now wins. The panel skipped detail probing for adaptive videos, so sizes were never known.
- **Resources and state.** `waitForDownload` had no timeout and never removed its listener (one leaked per call on a paused or stuck download) — now 10 minutes with cleanup. `bilibiliProbeByTab` and the playlist-probe set were never cleared on tab close, navigation or clear, and a reused tab id could skip probing; both are cleaned by `forgetTab`. `restore()` trusted stored entries for closed tabs and now verifies the tab exists. `onCurrentPage` returned true when page context was unknown, so a restarted worker showed the previous page's media; it now returns false. `mergePageContext` rewrote `sourcePage` on every entry, making iframe media look like top-level page media and bypassing filters; it now rewrites only top-level entries. The `persist()` quota fallback discarded the oldest entries including in-flight downloads; it now keeps every non-idle entry. `snapshotSocial` wiped the page title. `download-tasks.js` sweeps orphaned OPFS staging files at startup. `content.js` had no re-injection guard. The panel's timer was not cleared on `pagehide`. Error text duplicated its prefix.
- **Cleanup.** Removed the never-written `merging` status; `updateBadge` counts `task_` keys instead of reading all session storage; two third-party hardcoded site names in `filename.js` became generic rules.

---

## 1.1.3 — 2026-10-19

Reported from a real Bilibili page.

### Added

- **Bilibili quality completion.** The panel used to list only streams the player had already requested, so one video showed one quality. Bilibili now fills in the rest from the page's own same-origin `playurl` result (reusing the user's existing session; no login or permission bypass), listing every size the account can play and merging `avc1`/`hvc1`/`av01` copies of one size into a row. Sizes are real probe results, shown per row once the probe returns.

### Fixed

- **No feedback after choosing a quality.** The menu closed and the download ran in the background, which looked like nothing happened. The generic page button and the X/Weibo button now show "download started · 640×360" next to the button. The notice is bound to the video element so it survives floating-layer rebuilds when the background rotates a signed address.
- **Repeated clicks created several tasks.** Starting now checks whether that video already has an active task and refuses with `ALREADY_DOWNLOADING`; both entry points show "this video is already downloading". Finished tasks are replaced by the next download instead of piling up.
- **Adaptive entries split into two rows** when a request arrived before the page URL was known (grouped by `page|stream family`). Grouping is now by stream family and adopts the old format; the panel also de-duplicates by video at render time.

---

## 1.1.2 — 2026-10-19

### Changed

- **The intermediate save window is gone from every entry point.** The panel button and the in-page button both hand the task to the offscreen executor, which writes an OPFS staging file and then delivers it through `chrome.downloads` to Chrome's own download location. A failed or cancelled task deletes the staging file, leaving no half-written output.
- **Panel layout:** width 380 → 420 px, max height 560 → 600 px, tighter card spacing. Task cards no longer repeat the quality selector (the quality is fixed once the download starts); a single row is about 104 px, no horizontal scroll, and the quality menu stays inside the panel.
- Both entry points behave identically: save directly. If Chrome itself is set to "Ask where to save each file", the browser's native dialog still appears — that is a browser setting.

### Fixed

- **The task list lost entries.** `processing` was not counted as in-progress and completed tasks were filtered out. `processing` now counts as active, finished tasks stay with "saved", and `chrome.storage.onChanged` refreshes the panel.

---

## 1.1.1 — 2026-10-18

### Fixed

- Platform detection and release boundary corrections found during store preparation.

---

## 1.1.0 — 2026-10-18

### Added

- **YouTube and Bilibili adaptive media.** `src/dash.mjs` and `src/dash-transfer.mjs` handle static, unencrypted DASH manifests with separate video/audio AdaptationSets, quality selection, init/media segments and safe range validation; dynamic/live, DRM, `SegmentBase` and manifests whose timeline cannot be determined safely are refused explicitly.
- Adaptive grouping: YouTube `googlevideo` streams and Bilibili `da2/da2x` m4s streams from one video collapse into a single panel entry; ordinary MP4s keep the plain direct path.

### Changed

- **Douyin public media detection.** A narrow CDN fallback accepts only known Douyin media hosts, play paths and `media`/`xmlhttprequest`/`other` request types, and only when the response type is empty or an explicit generic binary type. Ordinary API responses are never treated as video.

---

## 1.0.34 — 2026-10-18

- First public store candidate after the rewrite: new page probing, social recognition, panel, offscreen save flow, background coordinator and download executor.

---

## Notes on the free/paid model

The quota is enforced by the Cloudflare Worker, not by the extension alone:

- `authorize` reserves one download against `completed_count + reserved_count < FREE_DOWNLOAD_LIMIT`. Reservations expire after 12 hours, and abandoning one releases it.
- `complete` increments `completed_count` and sets `cooldown_until = now + FREE_COOLDOWN_SECONDS` once the limit is reached; `cancel` releases the reservation without consuming quota.
- Every download entry point goes through `authorizeDownload`: the offscreen executor path (`startSavedTask`), the X/Weibo direct Chrome path (`startSocialDownload`), and the panel path. There is no download code path that skips authorization.
- A registered device resolves to `statusForLicensed` and never touches the free counters.

Reminder for development: the development `src/access-config.mjs` has an empty `ACCESS_API_BASE_URL`, so a locally loaded copy reports `development: true` and is unlimited. Only an assembled release injects the deployed Worker URL. To test real quota locally, assemble a release and load `chrome-extension/`, or set `WVDFLOW_ACCESS_API_URL` before `npm run release:assemble`.
