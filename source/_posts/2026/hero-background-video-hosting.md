---
title: "[實作筆記] 形象站滿版背景影片：放哪、吃不吃流量、能不能用 YouTube"
date: 2026/06/08 00:27:00
tags:
  - 實作筆記
---

## 前情提要

做形象站，Hero 想放一支滿版自動播放的背景影片。

從「檔案丟哪」一路查到「會不會吃爆流量」「YouTube 能不能拿來當乾淨背景」，整理成這篇。

結論先講：**背景影片別綁死 host，渲染只認 URL；正式影片走影音 CDN 或 Vimeo 背景模式，不要硬塞 YouTube。**

---

## 素材放哪：public/ vs src/assets

先講結論：

- **動態 URL 字串型的媒體 → `public/`**
- **程式碼裡 import 的媒體 → `src/assets/`（或任何 src 下，靠打包工具處理）**

差別在「這個值從哪來、怎麼被引用」：

| | `src/assets/` | `public/` |
|---|---|---|
| 引用方式 | 程式碼 `import img from "..."` | 絕對路徑字串 `/x.mp4` |
| 經過打包工具 | 會（雜湊檔名、優化、可能 inline） | 不會，原樣提供 |
| 適合 | 寫死在元件裡的素材（logo、icon） | 以固定 URL 直接提供的靜態檔 |

關鍵案例：如果你的背景媒體是**存在資料庫、可後台編輯的字串**，渲染時走 `<video src={url}>`，那它**必須**放 `public/`。

因為 `src/assets` 要 `import` 才拿得到那個雜湊 URL，而 import 是編譯期的程式碼行為——你沒辦法把它當成一個「可編輯的字串」塞進資料模型，跟使用者貼進來的網址混在一起。

換句話說：**範例預設值跟使用者貼的網址要走同一條路徑，就只能是 `public/` 的字串 URL。**

> 反例：純粹寫死在某個元件的 logo，那才該放 `src/assets` 用 import，享受優化與雜湊快取。

---

## 自存影片：會吃多少流量？

會，而且**滿版自動播放的背景影片，是整頁最吃流量的東西**。

粗估一下：一支壓到 SD 的背景影片約 3.5MB，每個訪客每次載入首頁就抓一次（瀏覽器會快取，回訪者不重抓）。

```text
3.5MB × 10,000 次造訪/月 ≈ 35GB/月
```

對小型品牌站通常還在額度內，但要注意兩件事：

1. 行動裝置上，這 3.5MB 是吃**訪客自己**的行動數據，UX 不友善。
2. 影片越大支越可怕，HD 1080p 動輒 12MB 起跳。

### 重點觀念：走 CDN ≠ 不算流量

很多人以為「丟到平台、平台有 CDN」就沒事了。

以主流部署平台來說，`public/` 的靜態檔確實會由它的 Edge Network（CDN）快取分送——**但傳輸量仍然計入你的方案額度**。

CDN 解決的是「分送速度與快取」，不是「免費流量」。要真正卸載流量，得換 host。

---

## 要省流量：CDN / 影音 host 比較

好消息是，只要你的渲染**只認 URL**，換 host 就是換一個字串，不用改程式碼。

| 方案 | 特點 |
|---|---|
| Cloudflare R2 | **egress（流出）免費**，等於自己的低成本 CDN |
| Cloudflare Stream / Mux | 專業影音串流，自動轉檔、自適應位元率、自帶 CDN |
| Cloudinary | 媒體優化平台，轉檔 + CDN |
| Vimeo | 有官方「背景模式」（下節詳述） |
| YouTube 嵌入 | 流量完全由它吸收（免費），但有播放器外觀 |

形象站的滿版背景，**通常不會用 YouTube**，原因看下面。

---

## YouTube 當「無播放器」背景？hack 與代價

YouTube 這麼主流，能不能拿來當乾淨背景？

**沒有官方乾淨方案**，主流做法都是 hack，而且 YouTube 近年反而把「藏品牌」收得更緊。

做法是 IFrame Player API + 參數 + CSS：

```text
autoplay=1&mute=1&loop=1&playlist=VIDEO_ID&controls=0&playsinline=1&rel=0&disablekb=1&iv_load_policy=3
```

```css
/* 把 iframe 放大超出視窗、置中裁切，藏掉黑邊與殘留 UI */
.bg-wrap { position: absolute; inset: 0; overflow: hidden; }
.bg-wrap iframe {
  position: absolute; top: 50%; left: 50%;
  width: 150vw; height: 150vh;        /* 刻意超出 */
  transform: translate(-50%, -50%);
  pointer-events: none;                /* 防止點到跳出控制列 */
}
```

代價很實際：

- `modestbranding` 已被 YouTube 淡化／移除，載入、暫停、結束仍可能**閃一下 logo／標題**。
- 單片循環要用 `loop=1&playlist=ID` 的 trick，**接點會黑屏跳一下**（要用 API `seekTo(0)` 才順）。
- 16:9 硬裁成滿版 → **邊緣被切掉**。
- 比原生 `<video>` **重很多**（要載播放器 JS、追蹤 cookie，除非用 `youtube-nocookie`）。
- **手機（尤其 iOS）自動播放不穩**。
- 「完全藏品牌」本身遊走在 ToS 邊緣。

---

## 真正乾淨的主流解

### 1. Vimeo 背景模式

Vimeo 的嵌入支援 `background=1` 參數，這是**官方為背景影片設計**的模式：自動播放、循環、靜音、**全程無控制列、無品牌**。

```text
https://player.vimeo.com/video/VIDEO_ID?background=1&muted=1&autoplay=1&loop=1
```

要 iframe 又要乾淨，這才是該選的，比 YouTube hack 乾淨太多。

### 2. 影音 CDN 的 mp4 + 原生 video

把影片放 Stream / Mux / R2，拿到真正的 mp4／HLS URL，用原生 `<video>` 播。

最乾淨、最可控，也最好做手機降級（poster、`preload`、小螢幕只給圖）。

---

## 架構建議：媒體以 URL 為主

把這些湊起來，得到一個簡單原則：**渲染層不要綁死 host。**

- 媒體只存一個 URL 字串（哪來的都行：自家 `public/`、R2、Mux、Vimeo iframe）。
- 範例／預設素材放 `public/`，跟使用者貼的網址走同一條路徑。
- 正式上線要換成 CDN？**貼上新 URL 即可，不用改程式碼。**
- 要支援 iframe 型（Vimeo／YouTube）就多開一種「媒體類型」，渲染分流到 iframe，原生影片仍走 `<video>`。

這樣早期可以自存 demo、之後無痛換成專業影音 CDN，決策成本最低。

---

## 小結

- 動態字串型媒體放 `public/`；寫死 import 的才放 `src/assets`。
- 滿版自動播放背景影片最吃流量；CDN 加速 ≠ 免費流量，傳輸仍算額度。
- 省流量靠換 host：R2（egress 免費）/ Stream / Mux / Cloudinary。
- YouTube 無播放器背景只能 hack，缺點一堆；要 iframe 乾淨解請用 **Vimeo `background=1`**。
- 最乾淨：影音 CDN 的 mp4 + 原生 `<video>`。
- 架構上讓媒體「只認 URL」，換 host 就是換字串，不用動程式碼。

(fin)
