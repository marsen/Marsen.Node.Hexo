---
title: "[實作筆記] 個人自動化平台(四) n8n 實作：取層，RSS 資料來源"
date: 2026/04/20 18:15:17
tags:
  - 實作筆記
---

## 前情提要

上一篇定好了「取、讀、寫」三層架構，這篇開始實作第一層：取。

目標是從四個 AI 新聞來源拉資料，輸出統一格式，交給讀層處理。

---

## 資料來源選擇

驗證過四個 RSS 來源都可用：

| 來源 | URL | 說明 |
| --- | --- | --- |
| OpenAI | `https://openai.com/news/rss.xml` | 官方，第一手產品更新 |
| Google Blog AI | `https://blog.google/innovation-and-ai/technology/ai/rss/` | 官方，涵蓋 Gemini 等產品 |
| The Verge AI | `https://www.theverge.com/rss/ai-artificial-intelligence/index.xml` | 綜合媒體，覆蓋廣 |
| HackerNews | `https://hnrss.org/newest?q=AI&points=50` | 社群篩選，工程師視角 |

> **Anthropic 目前無官方 RSS**，是個缺口，之後另找方案補上。

CLI 驗證方式：

```bash
curl -s -o /dev/null -w "%{http_code} %{url_effective}\n" <RSS_URL>
```

---

## n8n 節點選擇

取層用兩個節點組合：

- **Schedule Trigger**：控制執行時機（每週一次）
- **RSS Read**：拉取完整 feed 內容

為什麼不用 **RSS Feed Trigger**？

RSS Feed Trigger 會記住上次讀到哪，只推送新增項目。概念上正確，但它是 Trigger 節點，不適合被主流程呼叫（sub-workflow 架構）。

改用 **RSS Read + Filter 節點過濾日期**：

```text
Schedule Trigger → RSS Read → Filter（只保留近 7 天文章）
```

這樣取層不需要管狀態，邏輯乾淨，日期範圍由 Filter 節點統一控制。

---

## 實作步驟

### 步驟一：建立 Workflow

新增 Workflow，命名 `[取] The Verge AI`。

觸發節點選 **Schedule Trigger**，預設每天午夜執行即可（之後調整為週報頻率）。

### 步驟二：加入 RSS Read 節點

在 Schedule Trigger 後加 **RSS Read** 節點，填入 URL：

```text
https://www.theverge.com/rss/ai-artificial-intelligence/index.xml
```

### 步驟三：Filter 節點過濾日期

在 RSS Read 後加 **Filter** 節點，條件：

```
isoDate → is after → {{ $now.minus({days: 7}).toISO() }}
```

### 步驟四：Edit Fields 節點標準化輸出

在 Filter 後加 **Edit Fields（Set）** 節點，對應欄位：

| Name | Value |
| --- | --- |
| `title` | `{{ $json.title }}` |
| `url` | `{{ $json.link }}` |
| `content` | `{{ $json.contentSnippet }}` |
| `published_at` | `{{ $json.isoDate }}` |
| `source` | `The Verge AI` |

執行成功，每筆輸出格式如下：

```json
{
  "title": "Cloud development platform Vercel was hacked",
  "url": "https://www.theverge.com/tech/914723/vercel-hacked",
  "content": "Vercel, a major development platform...",
  "published_at": "2026-04-19T19:54:52.000Z",
  "source": "The Verge AI"
}
```

---

## API Credential 的資安決策

設定 Google Gemini credential 時，有一個值得討論的選項：**Allowed HTTP Request Domains**。

n8n 官方說明：

> Control which domains this credential can be used with in HTTP Request nodes

這個設定限制的是**被呼叫端**（destination）——也就是「這組 key 只能被送往哪些 domain」，不是「誰能在 n8n 裡使用這組 key」。

**能防護的範圍**：
- 攻擊者透過 HTTP Request 節點把 key 帶著打到外部惡意 server

**防護不到的範圍**：
- n8n 原生節點（如 Google Gemini Chat Model）
- 直接存取 n8n 資料庫
- 從 n8n UI 讀取 credential

**決策**：填 `generativelanguage.googleapis.com`，遵守最小權限原則。但主要防線是 Cloudflare Access（限制誰能登入 n8n），這個設定是額外的縱深防禦，不能過度依賴。

核心原則：**理解每層保護的邊界，不要以為設了就完全安全。**

---

## 參考

- [個人自動化平台(一) n8n & GCP VM](/2026/gcp-free-tier-n8n/)
- [個人自動化平台(二) Cloudflare Access & Cloudflare Tunnel](/2026/gcp-n8n-cloudflared/)
- [個人自動化平台(三) 取、讀、寫：三層可插拔管道設計](/2026/automation-pipeline-fetch-read-write/)
- [個人自動化平台(番外) 拆掉重建 GCP VM & Cloudflared Tunnel & Cloudflare Access](/2026/gcp-n8n-rebuild/)

## 小結

（待補）

(fin)
