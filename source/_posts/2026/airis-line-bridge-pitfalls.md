---
title: "[實作筆記] 幫自己的 AI 祕書接上 LINE：一個下午踩的五個坑"
date: 2026/07/08 18:01:01
tags:
  - 實作筆記
---

## 前情提要

我在打造一個叫 AIris 的個人 AI 祕書：接住情緒、記下想法、守住主線，不讓自己偏離目標也不讓自己逃回工作慣性。

腦（Notion 記憶 + Claude）已經在跑了，但只能在電腦前用。這次要做的是把她接上 LINE，變成隨身可以講話、會主動早安簡報的樣子。

流程照 SDD 走：先開 PBI、拆 story、寫技術設計（Clean Architecture + port/adapter，抽換式設計，記憶/溝通管道/LLM 引擎/主機都可以換），再實作。設計本身沒什麼好講的，值得記的是接真實服務時踩到的幾個坑。

## 坑一：Claude 的 Notion 連接跟後端程式的 Notion API 是兩回事

我在 Claude.ai 裡已經連過 Notion（用來讀資料庫、查頁面），一開始以為後端服務可以蹭這個授權。

錯的。那個連接是 Claude.ai 這個產品跟我 Notion 之間的關係，只在我跟 Claude 聊天、用 MCP 工具時有效。AIris 的橋接服務是一支獨立跑在自己主機上的程式，跟 Claude.ai 是兩個完全不同的執行環境，不會繼承這組授權。

要另外去 notion.so/my-integrations 建一個 integration token，再把要用的頁面分享給它。

## 坑二：GitHub Project 的「進行中」不是 issue 的 open/closed

晨間簡報要抓「進行中的 PBI」，我一開始以為查 issue 的 state 就好。

實際上「進行中」是 GitHub Project（Personal Backlog Project #2）裡一個叫 Status 的自訂欄位，用 GraphQL 的 `ProjectV2ItemFieldSingleSelectValue` 管理，跟 issue 本身的 open/closed 完全無關。

更麻煩的是查詢預設只抓前 100 筆，Backlog 裡當時有 115 個項目，結果我自己這個正在做的 issue（#223）因為排序落在後面，直接被漏掉，本地測試時完全沒察覺——查出來的清單「看起來合理」，就是少了一筆。加上 cursor 分頁才修好：

```ts
const nodes: ProjectItemNode[] = []
let cursor: string | null = null

do {
  const page = await this.fetchPage(cursor)
  nodes.push(...page.nodes)
  cursor = page.pageInfo.hasNextPage ? page.pageInfo.endCursor : null
} while (cursor !== null)
```

這種「資料量小的時候測試都過，量大才爆」的 bug 最陰險，因為 code review 光看邏輯完全看不出問題，一定要拿真實資料跑過一次。

## 坑三：Claude Agent SDK 不是 API client，是完整的 coding agent

原始設計想用 `@anthropic-ai/claude-agent-sdk`（走訂閱額度，不額外計費）。裝起來一看：`query()` 這個函式會實際 spawn 一個 `claude` CLI 子行程，帶著 Bash、Read、Edit 等等完整的 Claude Code 工具集。

這是為「寫程式的 agent」設計的框架，不是我要的「丟 prompt 進去、拿文字出來」的輕量 LLM client。跟原本設計的 `LlmPort` 介面（回傳 tool call 讓呼叫端執行）也對不上——Agent SDK 是自己把 tool 執行完才回傳最終結果，兩種模型不一樣。

MVP 階段先把 `allowedTools` 設空陣列，不給任何內建工具，只支援不帶 tool 的單輪呼叫（晨間簡報用得到的全部）。tool-calling 怎麼接，留給雙向對話那階段再解。

## 坑四：訂閱額度不是「免費」，是跟自己平常用量搶

原本想法是：用 `claude login` 就不用另外花 API 費用。查證後發現沒那麼美好——目前 Agent SDK 的用量是跟我平常在 Claude App 聊天的額度**共用**的，Anthropic 原本要把它獨立成一包月費額度的計畫也被暫停了。

也就是說 AIris 傳的每一則簡報、每一次對話，都在吃我自己平常用 Claude 的份額。這不是「不花錢」，是「用同一個額度做兩件事」，用多了會兩邊一起卡。

這個取捨我知道後還是選了訂閱額度（比起每月多付幾美元，更在意額度別浪費在按量計費上），但代價是主機不能是 Cloud Run 這種無狀態 serverless——登入狀態留不住，必須換一台持久式主機。

## 坑五：LINE 的 webhook 開關 API 打不到

要拿自己的 LINE userId（push 訊息一定要指定收件人），本來想用「取得好友列表」API 走捷徑，結果 403——這個 API 這個方案用不了。

改用暫時的 webhook：本機開一個小 HTTP server，用 `cloudflared tunnel` 開一個公開網址，設成 LINE 的 webhook endpoint，傳一則訊息進來抓 `event.source.userId`。

設定網址之後查狀態，發現 `active: false`——原來 webhook 的網址跟「是否啟用」是兩件事，啟用開關只能在 LINE Developers Console 手動點，沒有公開 API 可以打。抓到 userId 之後記得把暫時的 tunnel 跟 webhook 都關掉，不然 LINE 會一直對著一個死掉的網址重試。

## 最後一個小坑：composition root 忘了載入 .env

adapter 一個一個單獨測都過，最後把它們組裝進真正的進入點（`morningBriefingTrigger.ts`）直接跑，直接炸——所有環境變數都是 undefined。

前面測試都是我手動寫腳本讀 `.env` 塞進 `process.env`，正式的進入點從來沒做過這件事。補一行 `import 'dotenv/config'` 就解決了，正式主機上環境變數會由平台直接注入，這行在那邊會是無害的 no-op。

## 小結

五個坑沒有一個是「寫錯程式」，全部是「對外部系統的假設不成立」：以為授權能共用、以為欄位是原生狀態、以為 SDK 是輕量 client、以為訂閱額度是免費、以為開關能用 API 控制。

抽換式設計（port/adapter）在這種情境下的價值很實際：每個坑都被隔離在單一個 adapter 裡，Domain 跟 Application 完全沒受影響，改完重新接上就過了。晨間簡報最後真的推了一則訊息到我的 LINE 上，整個 L1 端到端跑通。

雙向對話還沒做——Agent SDK 的 tool-calling 怎麼接，是下一個要解的問題。

(fin)
