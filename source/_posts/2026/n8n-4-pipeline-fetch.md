---
title: "[實作筆記] 個人自動化平台(四) n8n 實作：收集層，RSS 資料來源"
date: 2026/04/20 18:15:17
tags:
  - 實作筆記
---

## 前情提要

上一篇定好了「收集、處理、輸出」三層架構，這篇開始實作第一層：收集。

目標是從四個 AI 新聞來源拉資料，輸出統一格式，交給處理層處理。

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

收集層有兩種做法：

| | RSS Feed Trigger + 暫存層 | RSS Read + Filter |
| -- | -- | -- |
| 抓取方式 | 增量，只抓新的 | 全量，每次重抓 |
| 重複抓取 | 不會 | 會（Filter 擋掉舊的） |
| Token 消耗 | 較省 | 稍多 |
| 架構複雜度 | 高（需外部暫存） | 低（無狀態） |
| 適合 sub-workflow | 不適合（Trigger 節點限制） | 適合 |

### 現況的選擇：RSS Read + Filter

現階段優先走通整條流程，不想多依賴一個外部暫存服務。

RSS Read 是普通節點，可以被 sub-workflow 呼叫，搭配 Filter 過濾近 7 天的文章，邏輯乾淨：

```text
Schedule Trigger → RSS Read → Filter（只保留近 7 天文章）
```

等之後有需要優化 token 消耗，再評估加暫存層。

---

## 實作步驟

### 步驟一：建立 Workflow

新增 Workflow，命名 `[收集] The Verge AI`。

觸發節點選 **Schedule Trigger**，預設每天午夜執行即可（之後調整為週報頻率）。

### 步驟二：加入 RSS Read 節點

在 Schedule Trigger 後加 **RSS Read** 節點，填入 URL：

```text
https://www.theverge.com/rss/ai-artificial-intelligence/index.xml
```

### 步驟三：Filter 節點過濾日期

在 RSS Read 後加 **Filter** 節點，條件：

```text
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

四個步驟完成，收集層跑通。每次執行會輸出統一格式的文章列表，準備交給處理層處理。

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

- [個人自動化平台(一) n8n & GCP VM](/2026/n8n-1-in-gcp-free-tier-vm/)
- [個人自動化平台(二) Cloudflare Access & Cloudflare Tunnel](/2026/n8n-2-cloudflared-and-tunnel/)
- [個人自動化平台(三) 收集、處理、輸出：三層可插拔管道設計](/2026/n8n-3-pipeline-overview/)
- [個人自動化平台(番外) 拆掉重建 GCP VM & Cloudflared Tunnel & Cloudflare Access](/2026/n8n-3.1-rebuild-infromation/)
- [個人自動化平台(五) n8n 實作：處理層，Aggregate + Gemini 週報整理](/2026/n8n-5-pipeline-process/)
- [個人自動化平台(六) n8n 實作：輸出層，Instagram API 取得 Token 全記錄](/2026/n8n-6-pipeline-output/)

## 小結

- 收集層用 RSS Read + Filter 過濾近 7 天，四個來源統一輸出格式
- Credential 的 Allowed HTTP Request Domains 是縱深防禦，主防線還是 Cloudflare Access
- RSS Read 是「簡單優先」的選擇，之後有需要再加暫存層優化

(fin)
