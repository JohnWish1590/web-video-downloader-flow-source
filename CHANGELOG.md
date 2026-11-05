# Changelog · Web Video Downloader Flow

## 1.1.23 — 2026-11-05（本地候选）

- 修复 YouTube 首页多视频画质及音轨混合：原分组只包含所在网页，同一首页解析的不同视频会被合并。现按播放器返回的视频 ID 隔离记录，并将未核实网络捕获与权威播放器格式列表分开，避免预览流和旧的整页聚合记录污染列表。
- 每条记录只组合同一视频的视频流、音轨、时长与大小；有该视频播放器格式列表时，详情页不再额外显示捕获流形成的重复/异源选项。
- 不按文件大小删除画质。高分辨率小文件可以是合法编码结果；文件大小由对应视频流及选中音轨的字节数计算，无法精确获知时沿用估算标记。
- 先复现后修复：1.1.22 在两视频独立记录断言中实际得到 1 条而非 2 条。新版验证不同长宽比、同分辨率不同大小、不同时长和音轨隔离；同一视频首页/详情页画质、大小、音轨及时间完全一致。合法的高分辨率小文件仍保留。
- 验证：完整基线 28/28，YouTube/B站页面专项通过，发布文件边界 PASS（54 文件、17 引用）。证据：build-records/youtube-quality-1.1.22-expected-failure.log、youtube-quality-1.1.23-source.log、youtube-quality-1.1.23-baseline.log。
- 限制：针对真实链接 E0ux_as0N-s 的完整播放器元数据提取未成功，不能逐档断言用户截图中的每条记录属于哪个视频。修复依据真实代码缺陷及隔离浏览器复现；真实网站新版端到端仍待加载后确认。
- 固定 chrome-extension/ 更新为 1.1.23；没有改变视频保存、命名、队列和额度授权，没有部署 Worker、发布 GitHub 或更新商店。

## 1.1.22 — 2026-11-05（本地候选）

- 补齐 YouTube 首页入口生成：普通封面和 Shorts 卡片不再依赖预览 video 节点或已经捕获的媒体记录。真实首页 ytLockupViewModelContentImage 类名已通过用户浏览器只读核对，兼容旧版缩略图入口。
- 每个入口绑定该封面的准确视频 ID；仅用户点击时读取可用画质，复用原有解析、队列、命名、保存和额度链路，不批量预取首页视频。
- 防止同尺寸卡片串视频：校验请求 ID、响应 ID、下载前卡片身份、解析期间页面导航；卡片复用或不可用时清除旧选项，不回退其他视频。首页有封面入口时不再重复给浮层预览挂第二个入口。
- 纠正前轮验收不足：1.1.21 只验证了带明确媒体地址的视频夹具，未覆盖真实首页封面尚未播放的状态。新测试无 video、无预置媒体地址，使用实际首页 aria-hidden 缩略图结构；旧 1.1.21 在入口等待处失败，新版普通视频/Shorts 的悬停、菜单、对应下载、换卡和不可用提示通过。
- 验证：源码专项 PASS；完整基线 28/28。日志见 build-records/youtube-home-1.1.22-source-generic.log、youtube-home-1.1.22-baseline.log，旧版失败证据 youtube-home-1.1.21-expected-failure.log。真实用户浏览器可读取封面结构，但脚本诊断被 YouTube CSP 拒绝；不声称已在用户浏览器完成新版扩展端到端验收。
- 固定加载目录 chrome-extension/ 更新至 1.1.22。未部署 Worker、发布 GitHub 或提交商店；此前 1.1.18 商店提交不变。

## 1.1.21 — 2026-11-05（本地候选）

- 修复 1.1.20 引入的 YouTube 普通首页预览入口回归：普通横向预览恢复使用视频原有几何位置，Shorts/竖向预览保留可见裁切区域计算。
- 根因：裁切算法无差别与所有 overflow 祖先相交；绝对定位的首页浮层可跨过零高度静态包装层而仍然可见，旧算法却把入口判成零高度。
- 回归先在 1.1.20 实际失败：可见视频能命中，但 wvdf-download[data-active]:not([hidden]) 等待超时。修正后，同一夹具的悬停与真实鼠标点击通过，Shorts 裁切点击、B站两张 BV 卡片/节点复用/原生控件避让及详情页/同页导航下载继续通过。
- 本轮仅修改 generic-content.js 的 YouTube 定位分支，未修改后台、B站卡片解析、视频保存、命名或额度；固定加载目录仍为 chrome-extension/。
- 验证边界：真实 YouTube 首页已只读核对浮层预览祖先结构；新版完整扩展交互使用隔离 Chromium 和受控站点测试，未声称用户真实首页已验收。未部署 Worker、更新 GitHub 或商店。

## 1.1.20 — 2026-11-05（本地候选）

- B站首页：封面卡片也可悬停显示入口；点击后按该卡片的 BV 号读取可用画质，继续使用原有视频队列、保存和额度授权。不会用其他卡片的视频兜底。
- B站原生控件：识别 div 形式的“稍后观看”，预留右上操作空间，避免下载按钮重叠和悬停出现后的跳动。
- YouTube Shorts：按视频与裁切容器、视口的交集定位，按钮点击区域留在可见画面内；移向按钮和操作菜单时保持可用。
- 回归：源码专项通过两张独立 BV 下载、卡片复用与不可用卡片拒绝、原生按钮不重叠、Shorts 裁切后真实鼠标点击，以及原有详情页和同页导航下载。完整基线 28/28 通过。
- 验证边界：专项为隔离 Chromium 和受控站点接口；真实B站首页结构已只读核对，尚未在用户浏览器验证新版完整交互。未部署 Worker、发布 GitHub 或上传商店。

## 1.1.19 — 2026-11-05（本地候选）

- B 站和 YouTube：视频下载图标改为移入播放器显示，移出隐藏；画质菜单打开或键盘焦点仍在控件内时保持可操作。
- 保留原有画质、命名、保存、队列和额度链路。固定加载目录为 chrome-extension/。
- 验证：源码及装配版本的两站浏览器专项均通过（初始隐藏、移入/移出、菜单保持、原有下载及同页导航）；文件边界 PASS。测试为隔离受控页面，真实网站完整体验及完整基线本轮未验证。
- 固定 chrome-extension/ 已刷新至 1.1.19；本轮未部署 Worker、上传商店或发布 GitHub。

## 1.1.18 — 2026-11-04（本地候选）

- 微博/X 的批量原图入口统一移到图片组上方，独立占行，避免覆盖评论等操作。
- ZIP 使用作者、帖子的发布日期、正文前12字命名；缺失日期省略，无正文使用“图片”。
- 保留单图悬停、原图字节、ZIP 保存与视频原有下载行为。

### 近两日问题汇总

- **授权与下载稳定性（1.1.10）**：修复配额并发完成重复计数、过期预留重放、重复打开首次须知，以及最终保存等待无期限的问题；B02 的产品边界仍是按本地安装身份计数，未增加账号或设备指纹。
- **页面扫描告警（1.1.13）**：移除受 Permissions Policy 拒绝的 `unload` 监听，保留视频变化扫描和 Blob 清理。
- **图片下载链路（1.1.14–1.1.16）**：补上微博来源请求头和实际 GET 字节校验，拒绝把 HTML 错误页保存成图片；修复微博裁切图片定位、X/微博批量入口重复和图标漂移。批量目录页面随后被撤回，改为一帖一次系统另存为 ZIP。
- **界面与视频回退（1.1.17–1.1.18）**：X/微博统一为单图悬停、每帖唯一的上方批量文字入口；视频恢复原有自动命名和默认保存逻辑，不再出现新增的文件名/目录输入框。

### 验证与边界

- 完整基线 **28/28**，发布边界检查通过；旧 1.1.17 在同一受控布局夹具上可复现图片入口覆盖，新 1.1.18 通过元素命中、鼠标点击、窄窗口、微博九宫格和 ZIP 字节/CRC 检查。
- 视频下载模块与 1.1.17 逐字节一致；真实 CDN 的 X/微博 JPEG 已通过原始字节和格式检查。
- Worker 已部署并在线复核：`af34a739-e38f-4b83-b7bc-de9e20cfac6b`（2026-11-04）。健康端点、3 次额度递减、冷却拒绝、取消不扣额度、完成/取消竞争、数据库终态回读、过期预留重放拒绝和安全重试均通过；冒烟测试数据已从线上 D1 清理。


## 1.1.17 — 2026-11-04（本地候选）

- X/微博统一为悬停显示单图按钮、每帖唯一的“下载全部原图”文字入口。
- 批量原图保存为 ZIP，仅调用系统另存为，移除运行包中的目录授权页面。
- 视频恢复自动命名与默认保存位置，移除新增的文件名、目录输入框。


All notable changes to this product. The extension version matches `manifest.json`.

Video pricing model, unchanged since 1.1.0:

- **Free to install. No account needed.** An unregistered user gets **3 completed video downloads**, then a **1 hour cooldown**, then 3 more. This is enforced server-side (`FREE_DOWNLOAD_LIMIT = 3`, `FREE_COOLDOWN_SECONDS = 3600`). X photo downloads are free and do not consume video quota (since 1.1.11).
- **Registration is optional** and removes the cooldown. One-time payment: `$6.99` via Stripe, or `¥28` via WeChat.

---

## 1.1.16 — 2026-11-04（本地候选）

- 按用户反馈纠正微博图片布局：信息流中的单张图标默认隐藏，只在对应图片悬停或键盘聚焦时显示。微博的长图在方形容器内被裁切，按钮改为锚定可见裁切容器，避免落在文字区、图片缝隙或错误位置。
- 移除每张图片上的批量图标，在整组图片下面、地点/互动栏上方仅放置一条小号“下载全部原图”文字；不改变页面图片尺寸或图片顺序。
- 修复只检查 HEAD 却让浏览器再次请求远程 URL 的保存缺陷：现在检查实际 GET 的状态、图片类型和文件签名，再保存已验证的 Blob 原始字节。HTML 内容即使伪装成 image MIME 也不落盘。X/微博单张共用此保存逻辑，图片不重编码，继续不占视频额度。
- 批量目录流程保留一帖一次授权；区分目录选择/授权失败与原图网络失败。File System Access 接口可能限制整个桌面等受保护目录，不能承诺桌面根目录可授权；可选择桌面下的图片子目录。不更改 Chrome 安全设置或用户的全局下载选项。
- 回归补齐九宫格真实裁切结构、默认隐藏/悬停、批量入口位置、HEAD 正常但 GET 返回 HTML、微博实际文件字节及 X 既有轮播行为。真实 CDN 的 X 和微博样本已通过新链路保存为 .jpg，文件签名为 JPEG；系统目录选择器的人工授权仍未验收。

验证：最终完整基线 **27/27**，发布边界 PASS（54 个文件）；真实 CDN 到文件落盘的 X/微博 JPG 样本经签名和 ffprobe 验证。前几轮出现的面板与 X 视频查看器固定时长等待不稳定，改为等待实际 UI 状态后保留原断言，最终同一轮全部通过。单张提示改为页面角落短暂显示，避免遮图。

---

## 1.1.15 — 2026-11-03（本地候选）

- 修复微博原图下载缺少来源信息导致 CDN 返回 403：在原图 HEAD 检查和实际 GET 下载前安装限定扩展来源的请求头规则；拒绝把 HTML 错误页当图片保存。
- 修复微博/X 的“全部”按钮隐藏失效及越出图片边缘的问题：每组仅第一张显示批量入口，图标统一并放在照片内，保留键盘标签和焦点反馈。
- 图片批量下载改为插件内保存页：每次点击一个帖子的全部下载，由用户选择一次目录，本批逐张写入原图；不保存目录句柄、不修改 Chrome 全局下载设置，下次批次重新选择。保留单张原生下载方式。
- 同名文件自动加序号，避免覆盖；关闭保存页会解除原页面按钮的等待状态，部分失败可只重试失败项。图片继续免费，不进入视频额度。
- 增加受来源保护的真实 HTTP 请求、真实目录句柄/文件写入、同名保护、关闭选择页清理、按钮边界的隔离 Chromium 回归。原生系统目录选择由测试替身提供，真实微博完整交互仍需另行核验。

验证：完整基线 **26/26**；补充批量测试覆盖部分失败后仅重试失败项、375px 布局、键盘焦点、小保存窗口完成自动关闭。原生目录选择器由测试替身提供，真实磁盘目录的手工授权尚未验收。

---

## 1.1.14 — 2026-11-03（本地候选）

本版本按已确认的开发清单完成 1–5 项；第 6 项 MP4/MKV 输出设置没有改动，抖音/TikTok 仍不属于运行时支持范围。

- **微博原图与批量下载**：从受控微博帖子图片记录获取原图，支持单张和当前帖子的全部图片逐张保存；X 图片继续保持原图下载。图片下载免费，不占视频额度；批量失败只保留失败项供重试，不打包 ZIP，也不抓取头像、封面、评论或推荐图。
- **文件名和目录**：下载前可编辑文件名，并可指定 Chrome Downloads 下的相对目录。拒绝绝对路径、UNC、盘符、`..` 和非法字符；原有文件名清理规则继续生效，视频、X 图片和微博图片统一执行。
- **画质偏好**：设置中可选择最高画质、优先 1080p 或优先 720p；按实际可用选项选择，找不到目标时使用最近的可用档位。下载菜单的手动选择优先于全局偏好。
- **视频队列**：默认单槽位，等待任务保存在 `chrome.storage.session`，支持上移、下移、移除；只有真正开始的任务才占用视频额度。覆盖通用页面、社交页面、原生 Chrome 直链和执行器路径，任务完成、取消和启动失败都会推进队列。
- **失败重试与诊断**：失败任务按阶段和原因分类，重试重新获取来源并生成新的任务/操作 ID；诊断信息只包含脱敏后的状态、阶段、平台、画质和错误类别，不上传日志，不包含 Cookie、令牌、签名 URL、页面内容或本地路径。

验证：`npm run test:baseline` **25/25**、`npm run test:replacement` **6/6**、`npm run test:boundary` PASS、`npm run release:verify` PASS；扩展归档 SHA-256 `3d76db12db7be712453919ae8bb68f92e82977a2b66947afdf81528e576c9932`，源码归档哈希记录在同一版本的 `ARCHIVE-HASHES.json`。未部署 Worker，未提交或发布 Chrome Web Store。

---

## 1.1.13 — 2026-11-03（本地候选）

- 移除通用页面 content.js 中多余的 unload 事件监听，避免页面禁止 unload 时出现 Permissions policy violation 警告。
- 继续保留视频元素变化监听，不在页面进入历史缓存时主动断开；已有临时 Blob URL 的 pagehide 清理逻辑保持原样。
- 增加禁止 unload 的浏览器回归：旧实现在同一断言中复现警告，修复后无警告；刷新、前进后退后的播放列表识别，以及扩展重载恢复均通过。
- 验证使用真实隔离 Chromium 与本地受控网页；不代表已验证 DeepSeek 的完整实际页面。
- 验收：完整基线 21/21、发布边界通过；运行时代码对比确认仅移除该行监听。

## 1.1.12 — 2026-11-03（本地候选）

- 修复 X 多图查看器下载图标随横竖照片和左右切换漂移的问题：优先将按钮放进与照片边界一致的现有定位容器，固定在照片右上角，顶部与右侧内边距各 6px。
- 轮播布局变化和切换动画期间隐藏按钮，稳定后只显示当前照片按钮；监听图片尺寸变化，窗口缩放后仍保持相对位置。离开弹窗时清理已移除轮播的动画状态。
- 补充真实 Chromium 中的多图回归：横竖图往返切换、没有 popstate 的同页导航、动画期间隐藏、单按钮及 6px 相对位置、窗口缩放。新增定位断言在旧实现上复现失败，修复后通过。
- 本次不改原图保存、图片免费和视频额度逻辑；仅生成本地候选并同步固定加载目录。

## 1.1.11 — 2026-11-03（本地候选）

- X 帖子照片与大图查看器新增“下载原图”按钮。多图分别保存当前照片；头像、视频封面和链接预览不显示图片按钮。
- 直接请求 X 的 `name=orig` 最高可用尺寸，保留图片格式和响应字节，不截图、不重新压缩；原图不可用时明确报错，不自动降级到预览图。
- 图片免费下载，不请求视频额度授权，也不记入视频下载任务；首次使用须知仍需同意。文件名为作者、帖子 ID、图片序号。
- 覆盖并发重复点击、原图失败/非图片响应、同页导航、直接打开大图页、轮播旧帧拦截、脚本重复注入和窄窗口控件位置。
- 用户示例原图通过实际扩展保存验证：1290×1296 JPEG、362,335 字节，保存文件 SHA-256 与重新请求的原图响应一致：`adfcf302999bdfe2846aebdf25934f575604c27f7b6eb7d983d20545b6bd3cbe`。该保存验证使用受控帖子 DOM 与真实 X 图片 CDN；实际已登录 X 页面的图片链接结构另行核对。
- 验证：`test:baseline` 21/21，`test:access`、`test:access-browser`、`test:boundary` 通过。新增测试产物保存在项目内 `research/x-photo-tests/`，每次运行结束清理本次隔离浏览器配置，只保留样本和截图。

## 1.1.10 — 2026-11-01（本地候选；未部署、未提交商店）

本版本处理功能复查中的 B01、B03、B04、B05；B02 按用户确认保留“每次本地安装身份可获得 3 次免费额度”的产品边界。

### 修复

- **B01 配额重复计数**：完成下载时在一个 D1 batch 事务中条件更新额度和预留终态；并发完成只计一次，完成/取消竞争由数据库终态决定。数据库中途失败时 batch 回滚，安全重试仍只记一次。线上 D1 并发尚未实测。
- **B03 过期预留重放**：重放 pending reservation 时先执行过期清扫并重新读取记录；只有仍为 pending 才返回 `allowed: true`，已过期或已处理记录会被拒绝。
- **B04 须知页重复打开**：X 直链下载接入共享须知页复用逻辑；未接受前仍在授权前拦截。修复了该分支一处被截断的提示文本语法错误。
- **B05 最终保存等待无期限**：连续 30 分钟没有下载字节进度时请求取消 Chrome 下载；暂停期间停表，有进度则重新计时。等待 Chrome 最终状态后才返回成功或取消，避免界面先报失败而文件随后完成；所有终态清理监听器和计时器。

### 产品边界与验证范围

- **B02 安装身份**：额度绑定扩展本地生成的 `client_id`。用户确认卸载重装后重新获得 3 次免费额度可以接受；未增加账号或设备指纹，也未实际执行卸载重装验证。
- 已运行配额并发/失败注入/TTL 回归、须知页复用与门禁、下载等待超时/暂停/进度/完成竞态、任务取消终态和发布边界检查。
- 下载超时和任务状态使用 VM 模拟；真实 Chrome 下载、线上 D1 并发、真实网站兼容及支付均未验证。本版本只生成本地候选包，不部署 Worker、不发布商店。

## 1.1.9 — 2026-10-20

The first-run notice no longer opens another tab when a download is refused before acknowledgment. The generic page download test now accepts the notice before checking that downloads do not open an intermediate window.

### Fixed

- Reuse the already-open first-run notice when a download is attempted before acknowledgment, including after the service worker restarts.
- Added browser coverage that a refused download does not create another notice tab and that accepting the notice closes it before generic download checks.

### Verified

- `npm run test:baseline` 18/18, `npm run test:replacement` 6/6, `npm run test:generic-page` PASS, `npm run test:boundary` PASS, and `npm run release:verify` PASS.

### Released

- Submitted to the Chrome Web Store for review on 2026-10-26, replacing the published 1.0.31 build. Automatic publication after approval is enabled.
- The listing now declares in-app purchases instead of a free product, because a license is sold to remove the download cooldown.
- The store description was rewritten for this version, and the reviewer instructions were rewritten to fit the 500 character limit and to mention the first-run notice.
- Corresponding source published as GitHub Release `v1.1.9`, SHA-256 `84286eb890be3051eeaf04c4507eabd0f1090d6145b753a6d6b655938f1f3e27`, verified against the public asset.

---

## 1.1.8 — 2026-10-20

Douyin and TikTok downloads are removed from the extension runtime.

### Changed

- Removed their page-script matches, player controls, bridge injection, media registration, panel entries, and download routes. The generic page script now excludes both sites, and the background clears old cached entries and refuses stale download requests.
- Preserved the local 1.1.7 development implementation and probes under `research/retired-platform-support/1.1.7/` and in the corresponding-source archive only. This is a source snapshot, not a claim that every file matches the previously published archive. These materials are not included in the extension package or current test suite.
- Updated the store listing, privacy wording, and reviewer instructions to match the supported runtime.
- Added `npm run test:unsupported-platforms` to verify both sites produce no page control or downloadable media entry.

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

### Added

- **First-run notice before anything is downloaded.** Chrome Web Store expects a prominent disclosure before an extension handles page metadata, and the copyright point belongs in front of the user rather than buried in a policy. `welcome.html` opens on install and lists what the extension reads, the copyright and responsibility terms, and the free quota. Every path that writes a file — the offscreen executor and the direct Chrome download — refuses with `DISCLOSURE_REQUIRED` until the user ticks the box. Page inspection keeps working, so the panel still shows what it found.
- The notice is reachable again from the settings page footer.

### Documented

- Added `CHANGELOG.md` (this file) and rewrote the public README with the full feature matrix, the free/paid model, and the corresponding-source instructions.
- `npm run test:quota` — free-quota accounting driven through the real Worker and the real `schema.sql`, with `node:sqlite` standing in for D1.
- `npm run test:douyin-bridge` — end-to-end check for the Douyin quality list and download path.

### Verified

- `npm run test:baseline` 18/18 (adds `first-run-disclosure`), `npm run test:replacement` 6/6, `npm run test:boundary` PASS (44 runtime files), `npm run test:generic-page` PASS, `npm run test:quota` PASS, `npm run test:disclosure` PASS.
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
