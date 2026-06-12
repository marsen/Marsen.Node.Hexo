---
title: "[實作筆記] 前端靜態資源放哪裡：public/ vs src/assets"
date: 2026/06/12 19:24:23
tags:
  - 實作筆記
---

## 前情提要

做形象站，Hero 要放一支滿版自動播放的背景影片，第一個冒出來的問題是「影片放哪裡」。

這個問題帶出了前端靜態資源的兩種擺法：`public/` 和 `src/assets/`。對一般人看起來只是資料夾名字不同，但對工程師來說，兩者的機制差很多。

> 做形象站 Hero 背景影片的前置研究。完整的流量估算與播放器選型，見[下一篇：形象站滿版背景影片](/2026/hero-background-video-hosting/)。

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

## 參考

- [形象站滿版背景影片：流量、CDN 與播放器選型](/2026/hero-background-video-hosting/)

## 小結

- `src/assets/` 交給打包工具管，享有自動雜湊快取，但只能靜態 `import`、不適合大檔案。
- `public/` 繞過打包工具，直接 serve，URL 是字串、runtime 拿來用都沒問題。
- 影片、字型這類大媒體放 `public/`；icon、logo 等小圖用 `src/assets/`。
- 換 CDN 只需換 URL 字串，不用動程式碼。

(fin)
