---
title: "[實作筆記] 個人自動化平台(五) n8n 實作：處理層，Aggregate + Gemini 週報整理"
date: 2026/04/27 16:03:36
tags:
  - 實作筆記
---

## 前情提要

收集層跑通後，每次執行輸出統一格式的文章列表。

這篇記錄處理層：把多筆文章合併，送給 Gemini 整理成 AI 週報。

未來也可能抽換成不同的雲端或地端 AI 模型服務。

---

## 為什麼要 Aggregate？

收集層輸出的是多筆獨立資料（以 RSS 為例，一次可能拿到 10 筆）。

Basic LLM Chain 預設對每筆各送一次請求，10 筆 = 10 次 API call，超過 Gemini 免費版 5 RPM 限制。

用 **Aggregate** 節點先把多筆合成一筆，再送一次請求給 AI。這也更符合「週報」的本意——要的是一份完整報告，不是 10 篇獨立摘要。

```text
Edit Fields → Aggregate（All Item Data）→ Basic LLM Chain（Google Gemini）
```

---

## Prompt 設計

```text
週報日期：{{ $now.toFormat('yyyy年MM月dd日') }}

以下是本週 AI 新聞，請用繁體中文整理成一份週報。
每則新聞說明「是什麼」和「為什麼重要」，條列呈現。

{{ $json.data.map(i => `【${i.source}】${i.title}\n${i.content}`).join('\n\n') }}
```

執行結果：日期正確，內容有條理，每篇清楚說明「是什麼」和「為什麼重要」。

---

## 踩坑：Rate Limit

第一次直接把 10 筆送給 LLM，遇到 `429 Too Many Requests`：

```text
quota exceeded: limit 5 requests/minute (gemini-2.5-flash free tier)
```

解法：加 Aggregate 節點合併成一筆，從 10 次請求降為 1 次，問題消失。

---

## 參考

- [個人自動化平台(一) 用 GCP 免費 VM 跑 n8n](/2026/04/17/2026/n8n-1-in-gcp-free-tier-vm/)
- [個人自動化平台(二) Cloudflare Access & Cloudflare Tunnel](/2026/04/17/2026/n8n-2-cloudflared-and-tunnel/)
- [個人自動化平台(三) 收集、處理、輸出：三層可插拔管道設計](/2026/04/20/2026/n8n-3-pipeline-overview/)
- [個人自動化平台(番外) 拆掉重建 GCP VM & Cloudflared Tunnel & Cloudflare Access](/2026/04/18/2026/n8n-3.1-rebuild-infromation/)
- [個人自動化平台(四) n8n 實作：收集層，RSS 資料來源](/2026/04/20/2026/n8n-4-pipeline-fetch/)
- [個人自動化平台(六) n8n 實作：輸出層，Instagram API 取得 Token 全記錄](/2026/04/22/2026/n8n-6-pipeline-output/)
- [個人自動化平台(番外) Google OAuth redirect_uri_mismatch 除錯記錄](/2026/04/27/2026/n8n-6.1-google-oauth-redirect-uri-debug/)
- [個人自動化平台(七) n8n 備份自動化：workflow + credentials → GitHub](/2026/04/29/2026/n8n-7-backup-automation/)
- [個人自動化平台(番外) 更新自架在 GCP VM 上的 n8n（Docker 版）](/2026/05/28/2026/n8n-7.1-docker-update-on-gcp-vm/)
- [個人自動化平台(番外) n8n AI 週報踩坑記：一條接錯的線，與找免費生圖來源的旅程](/2026/06/05/2026/n8n-ai-weekly-ig-debug-free-image-gen/)

## 小結

- Aggregate 節點把多筆合成一筆，是處理層的關鍵，解決 rate limit 也符合週報語意
- Prompt 用 expression 動態帶入日期與新聞內容，輸出穩定

(fin)
