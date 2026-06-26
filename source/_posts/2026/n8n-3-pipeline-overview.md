---
title: "[實作筆記] 個人自動化平台(三) 收集、處理、輸出：三層可插拔管道設計"
date: 2026/04/20 14:20:23
tags:
  - 實作筆記
---

## 前情提要

前兩篇把 GCP VM + n8n + cloudflared tunnel 都設好了，環境就緒。

這篇不急著動手，先把架構想清楚。

目標是一個可以自由擴充的個人內容自動化平台：新增資料來源、換 AI 模型、改輸出管道，都不需要重寫整個流程。

---

## 核心設計：三層分離

把整個管道拆成三層，每層只負責一件事：

```text
收集（Fetch）   →   處理（Process）   →   輸出（Sink）
```

- **收集**：從外部拿資料，不做任何處理
- **處理**：理解資料，產生新資訊
- **輸出**：把結果推出去

每層都是介面，實作可以自由替換或組合。

---

## 每層的邊界

### 收集（Fetch）

負責從各種來源拉資料：RSS、API、Webhook、爬蟲……

不管來源是什麼，輸出格式統一：

```json
{
  "title": "文章標題",
  "url": "https://...",
  "content": "原文內容或摘要",
  "source": "來源名稱",
  "published_at": "2026-04-20T00:00:00Z"
}
```

這個格式就是「收集」和「處理」之間的契約。只要每個來源都輸出這個 schema，「處理」完全不需要知道資料從哪來。

新增來源：建一個新的 sub-workflow，輸出同樣格式。
停用來源：把那個 sub-workflow 關掉。

### 處理（Process）

負責理解資料，產生新資訊。

目前規劃的功能：

| 功能 | 說明 |
| --- | --- |
| 摘要歸納 | 把長文濃縮成重點 |
| 事實查核 | 驗證內容可信度（未來） |
| 創意發想 | 從資料延伸新觀點（未來） |

AI 是「處理」層的實作細節，不是介面的一部分。

今天用 Claude，明天換 Gemini，或跑地端模型（Ollama）——主流程不在意，只呼叫「處理」的 sub-workflow，不管裡面用的是什麼模型。

「處理」層的輸出格式：

```json
{
  "title": "文章標題",
  "url": "https://...",
  "summary": "AI 整理的重點",
  "tags": ["AI", "LLM"],
  "source": "來源名稱",
  "published_at": "2026-04-20T00:00:00Z"
}
```

### 輸出（Sink）

負責把結果推出去。

兩個維度都可以擴充：

**格式**：
- 純文字
- 簡報（未來）
- 影音（未來）

**管道**：
- Email
- LINE
- YouTube / Instagram / Facebook（未來）

從「處理」到「輸出」可以有多條路線，同一批摘要可以同時走不同路線，互不干擾：

```text
處理（摘要結果）
  ├─ → Email 週報（長版純文字）
  ├─ → LINE 推播（短版）
  └─ → 存檔（JSON，備用）
```

---

## 在 n8n 的實作方式

n8n 的 sub-workflow 機制很適合做這件事：

- 每個「收集」來源 → 一個 sub-workflow
- 每個「處理」功能 → 一個 sub-workflow
- 每個「輸出」管道 → 一個 sub-workflow
- 主流程：用 **Execute Workflow** 節點串起來，用 **Merge** 合併多個來源，用 **Switch / If** 分流到不同輸出

新增或停用任何環節，只需要在主流程加減節點，不動其他邏輯。

---

## 小結

- 三層分離（收集、處理、輸出），每層依賴介面不依賴實作
- 各層之間用固定的 JSON schema 溝通，是可替換性的基礎
- 「處理」層的 AI 模型是實作細節，抽成 sub-workflow 就能自由替換
- 「輸出」層支援多路線同時輸出，格式和管道各自獨立擴充
- n8n 的 sub-workflow 機制天然適合這個設計

下一篇開始動手實作第一條 pipeline：AI 新聞週報。

## 參考

- [個人自動化平台(一) n8n & GCP VM](/2026/04/17/2026/n8n-1-in-gcp-free-tier-vm/)
- [個人自動化平台(二) Cloudflare Access & Cloudflare Tunnel](/2026/04/17/2026/n8n-2-cloudflared-and-tunnel/)
- [個人自動化平台(番外) 拆掉重建 GCP VM & Cloudflared Tunnel & Cloudflare Access](/2026/04/18/2026/n8n-3.1-rebuild-infromation/)
- [個人自動化平台(四) n8n 實作：收集層，RSS 資料來源](/2026/04/20/2026/n8n-4-pipeline-fetch/)
- [個人自動化平台(五) n8n 實作：處理層，Aggregate + Gemini 週報整理](/2026/04/27/2026/n8n-5-pipeline-process/)
- [個人自動化平台(六) n8n 實作：輸出層，Instagram API 取得 Token 全記錄](/2026/04/22/2026/n8n-6-pipeline-output/)

(fin)
