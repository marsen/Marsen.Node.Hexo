---
title: "[實作筆記] 形象站滿版背景影片"
date: 2026/06/08 00:27:00
tags:
  - 實作筆記
---

## 前情提要

Hero 想放一支滿版自動播放的背景影片。

> Hero 區塊是網頁最頂部的全版區塊，通常含標題、說明文字與背景，是訪客進站看到的第一眼。

思考點有以下

1. **影片放哪裡**：自存到專案（`public/` 或 `src/assets`），還是丟外部 host（R2、Stream、Vimeo）？
2. **流量控管**：滿版自動播放背景影片是首頁最重頭的流量來源，怎麼估、怎麼省？
3. **UI 體驗**：背景影片要無控制列、自動播放、靜音、循環——原生 `<video>` 還是嵌入 YouTube／Vimeo？

## 素材放哪：public/ vs src/assets

對一般人來說看起來只是資料夾名字不同，但對這裡有蠻多的細節，我們來逐一分析。

首先要了解兩個時間點：

- **編譯期（Build Time）**：網站還沒上線前，可以將一些資源優化打包減少不必要的流量，或是使用雜湊命名來避免快取問題。
- **執行期（Runtime）**：網站上線後，使用者實際使用的期間。程式在這個階段才真正執行，才能動態取得資源、回應使用者的操作。

### src/assets/ — 打包工具接管

現代主流網站打包工具（Vite / Webpack）會在編譯期處理這些檔案：

- **雜湊檔名**：輸出 `bg.a1b2c3d4.mp4`，URL 跟著內容變，瀏覽器可以放心永久 cache（`Cache-Control: immutable`）——改了檔案雜湊就換，訪客自動抓新的
- **可能優化**：圖片壓縮、小檔案 inline 成 base64 等
- **追蹤依賴**：沒有被 import 的檔案，build 產出裡就不會出現

**限制一**：`import` 是編譯期的靜態行為，URL 在 build 時就決定了，沒辦法在 runtime 動態組字串——更不可能從資料庫讀一個 URL 出來用。

**限制二**：打包工具適合處理 JS、CSS、小圖，遇到影片這種大檔案，build 時間拉長，而且影片可壓縮或優化的效果有限，過一手只是浪費。

### public/ — 繞過打包工具

這裡的資源不會被打包工具處理，本質跟早期 Apache / Nginx 放一個靜態資料夾直接 serve 檔案是同一件事——收到請求，原樣回傳。快取責任落在「伺服器那層」：早期要自己設 cache header，現代部署平台（Vercel、Netlify、Cloudflare Pages）會自動推到 Edge Network 幫你卸載流量——但這是**平台**做的，不是框架。

URL 永遠是 `/你放的路徑`，開發時字串直接用不用 `import`：

```text
<video src="/hero.mp4" />     ← 寫死字串
<video src={dbValue} />       ← 從資料庫來的字串，也走得通
```

> Next.js 本身真正在 framework 層處理靜態資源的是 `next/image`：server side 做圖片壓縮、格式轉換、依裝置裁切，回傳正確的 cache header。`public/` 它不碰。

### 判斷比較

| | `src/assets/` | `public/` |
|　---　|　---　|　---　|
| 引用方式 | `import`，編譯期靜態 | 字串 URL，runtime 動態 |
| 快取 | 自動雜湊，可放心 `immutable` | 手動管理，或靠部署平台預設值 |
| 大檔案 | 不適合（影片、字型） | 適合 |
| 適合 | Icon、logo、小圖 | 後台可換的媒體、使用者貼的 URL |

> 我們的例子是背景影片，幾乎都是「後台可換、runtime 才知道 URL」的需求 → 判斷上選擇 `public/` 較為適合。

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

## 要省流量：CDN / 影音 host 比較

影片放在 `public/` 的話，流量算在部署平台的額度裡。

以我們用的平台　Vercel 免費方案只有 100GB/月，一支背景影片流量大的時候很快就吃光。

解法是把影片搬到外部 host，讓別人的伺服器去扛流量。

好消息是，只要程式碼只用 URL 字串引用影片，換 host 就是換那個字串，**不用改程式碼**：

```text
// 原本放 public/
<video src="/hero.mp4" />

// 搬到 R2 後，只改這一行
<video src="https://pub-xxx.r2.dev/hero.mp4" />
```

兩條路線：

### 路線一：自己存，外部 CDN 來送（Cloudflare R2

把影片傳到 R2，egress（流出流量）免費，只付儲存費（$0.015/GB/月）。一支 5MB 影片幾乎是 $0，部署平台那邊完全不計這筆流量。

### 路線二：讓別人的平台吸收（Vimeo/Youtube）

影片上傳到 Vimeo，你的伺服器完全不出流量，由 Vimeo 負責送給訪客。代價是多了第三方依賴，免費方案有上傳量限制。

| 方案 | 流量誰扛 | 備註 |
| --- | --- | --- |
| Cloudflare R2 | R2（egress 免費） | 自己管檔案，換 URL 即可 |
| Cloudflare Stream / Mux | 平台 | 專業影音串流，自動轉檔、自適應位元率 |
| Cloudinary | 平台 | 媒體優化 + CDN |
| Vimeo | Vimeo | 免費方案有上傳限制，背景模式乾淨 |
| YouTube 嵌入 | YouTube（完全免費） | 有播放器外觀問題（下節詳述） |

形象站的滿版背景，**通常不會用 YouTube**，原因看下面。

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

## 架構建議：媒體以 URL 為主

把這些湊起來，得到一個簡單原則：**渲染層不要綁死 host。**

- 媒體只存一個 URL 字串（哪來的都行：自家 `public/`、R2、Mux、Vimeo iframe）。
- 範例／預設素材放 `public/`，跟使用者貼的網址走同一條路徑。
- 正式上線要換成 CDN？**貼上新 URL 即可，不用改程式碼。**
- 要支援 iframe 型（Vimeo／YouTube）就多開一種「媒體類型」，渲染分流到 iframe，原生影片仍走 `<video>`。

這樣早期可以自存 demo、之後無痛換成專業影音 CDN，決策成本最低。


## 小結

- 動態字串型媒體放 `public/`；寫死 import 的才放 `src/assets`。
- 滿版自動播放背景影片最吃流量；CDN 加速 ≠ 免費流量，傳輸仍算額度。
- 省流量靠換 host：R2（egress 免費）/ Stream / Mux / Cloudinary。
- YouTube 無播放器背景只能 hack，缺點一堆；要 iframe 乾淨解請用 **Vimeo `background=1`**。
- 最乾淨：影音 CDN 的 mp4 + 原生 `<video>`。
- 架構上讓媒體「只認 URL」，換 host 就是換字串，不用動程式碼。

(fin)
