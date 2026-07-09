---
title: "[實作筆記] Tool Calling 是什麼：從一次介面改版學到的事"
date: 2026/07/09 08:16:36
tags:
  - 實作筆記
---

## 前情提要

AIris 的 LINE 雙向對話（讓她能記事、能查資料）需要 LLM 呼叫兩個自訂動作：`create_record`（記一筆事）、`query_records`（查資料）。

一開始設計 `LlmPort` 介面時，假設的是「LLM 說出想做什麼，我的程式碼自己執行」這種單次來回模式。真的要接 Claude Agent SDK 才發現：它不是這樣運作的，介面得改。

第一反應是抗拒的——**介面不是應該保持抽象嗎？因為一個 SDK 的實作細節就要改介面，這不就是洩漏實作細節嗎？** 這篇記錄想清楚這件事的過程。

## Tool 到底是什麼

LLM 本質上只做一件事：吃文字進去，吐文字出來。它沒有手，碰不到資料庫，沒辦法自己寫 Notion。

Tool 就是告訴 LLM：「這裡有幾件事你可以『開口要求』別人幫你做，我先把它們的名字跟說明給你」。

比喻成餐廳：LLM 是客人，不會下廚。Tool 定義是菜單，客人只能「點餐」（說出要呼叫哪個 tool、附上什麼參數），真正把菜端出來的是服務生——也就是我們自己寫的程式碼。

一個 tool 定義長這樣，四個欄位各自的工作完全不同：

| 欄位 | 給誰看 | 做什麼 |
| --- | --- | --- |
| `name` | LLM | 點餐時用的名字 |
| `description` | LLM | 白話說明「這道菜是幹嘛的」，讓 LLM 自己判斷什麼時候該點 |
| `parameters` | LLM | 規格書，點這道菜要附哪些資訊 |
| `handler` | 我們的程式碼 | LLM 點餐**之後**真正被執行的動作 |

前三個都只是「說明書」，LLM 讀了自己做決策，不會執行任何東西。`handler` 才是唯一真的會動的部分。

## 兩種模式：誰負責「執行完再繼續問」這個迴圈

原生的 Messages API（單次呼叫）長這樣：

```text
呼叫一次 → 拿到「LLM 想呼叫 create_record，附上這些參數」
→ 我自己執行 → 把結果餵回去 → 再呼叫一次 → 拿到最終文字
```

這個「執行完、餵回去、再問一次」的迴圈，是**呼叫端自己寫的**。我原本設計 `LlmPort` 就是照這個模式：`llm()` 回傳 `{ text, toolCalls? }`，呼叫端自己檢查 `toolCalls`、自己執行。

Claude Agent SDK 不是這樣。你把 tool 的「說明書 + 真正會執行的程式碼」一次交給它，它自己在內部跑完整輪，只吐出最終文字。這個迴圈**在 SDK 內部**，外部沒有介面可以攔截「LLM 想呼叫 X」然後自己接手執行。

證據很直接，改介面之前的程式碼裡，只要 `tools` 有內容就直接拋錯：

```ts
if (tools && tools.length > 0) {
  throw new Error('ClaudeAgentSdkAdapter 尚未支援帶 tools 的呼叫')
}
```

不是還沒寫，是原本的契約在這個 SDK 上沒有合法的寫法。

## 改介面到底有沒有洩漏實作細節

關鍵問題：`llm(prompt, tools) → { text }` 這個外部形狀，改版前後有沒有變得比較不抽象？

沒有。變的只是 `ToolDefinition` 多了一個 `handler` 欄位（真正會執行的函式），`LlmResponse` 拿掉了 `toolCalls`（不再需要呼叫端自己執行）。呼叫方式、回傳型別的「意圖」完全沒變。

真正該問的是：**「執行完 tool、繼續問、直到拿到最終答案」這個迴圈邏輯，本來就該放在哪一層？**

- 用 Agent SDK：這個迴圈它自己包辦
- 假設換成 Gemini 的原生 API（不是 agent 框架）：它只會「說」要呼叫哪個 tool，不會自己執行——這時候 `GeminiAdapter` 就得自己把這個迴圈寫出來：呼叫、看到 function call、執行、餵回去、再呼叫，直到沒有更多呼叫為止

兩種引擎，`ProcessIncomingMessageUseCase`（呼叫端）看到的介面完全一樣，一行都不用改。差別永遠關在 adapter 內部。

反過來想，如果堅持照原本的設計把 `toolCalls` 传回呼叫端，那個「執行、餵回去、再問一次」的迴圈就會被迫寫進 Application 層——這才是真正把「這個引擎需要幾輪來回」這種 SDK 專屬細節，洩漏到不該知道的地方。改介面不是妥協，是把一個本來放錯位置的責任歸位。

## 具體長什麼樣

```ts
const createRecordTool: ToolDefinition = {
  name: 'create_record',
  description: '把交辦的一件事記到 AIris 的記憶裡',
  parameters: {
    type: 'object',
    properties: {
      content: { type: 'string' },
      type: { type: 'string', enum: ['follow-up', 'idea', 'decision', 'reflection', 'inbox'] },
      isEmotionalMoment: { type: 'boolean' },
      waitingOn: { type: 'string' },
      dueDate: { type: 'string' },
    },
    required: ['content', 'type', 'isEmotionalMoment'],
  },
  handler: async (args) => {
    const input = args as { content: string; type: RecordType; isEmotionalMoment: boolean; waitingOn?: string; dueDate?: string }
    await memory.record({
      content: input.content,
      type: input.type,
      isEmotionalMoment: input.isEmotionalMoment,
      capturedAt: new Date(),
      meta: { waitingOn: input.waitingOn, dueDate: input.dueDate ? new Date(input.dueDate) : undefined },
    })
    return { saved: true }
  },
}
```

`parameters` 裡的欄位名字是我們自己取的，不是 LLM 供應商規定的。`handler` 收到的 `args`，形狀對應的是 `parameters.properties`（規格書列出的欄位），不是 `parameters` 這個物件整體——規格書跟真實資料是兩件事。

`handler` 內部做的事很薄：把 `args` 翻譯成 AIris 內部的 `RawRecord` 格式，呼叫 `memory.record()`。真正「存去哪裡」的邏輯，在 `MemoryPort` 介面背後的那個具體實作（目前是 `NotionMemoryAdapter`）——`handler` 完全不知道底層是 Notion 還是 SQL，也不需要知道。

## 小結

一開始會抗拒改介面，是因為直覺把「介面因為某個實作而調整」當成壞味道。但抽象該不該動，看的不是「有沒有因為某個具體東西而改」，而是**改完之後，介面外部看到的形狀有沒有變窄、變得只服務單一實作**。

這次沒有——`llm(prompt, tools) → text` 還是一樣通用，只是把「誰負責跑執行迴圈」這個責任，從「假設呼叫端會寫」改成「adapter 自己決定要不要寫」。換一個引擎，Application 層完全不用動，這才是抽象該有的樣子。

(fin)
