---
title: "[實作筆記] LINE Messaging API 用量查詢：免費額度怎麼看，以及為什麼通知會停止"
date: 2026/05/06 19:00:49
tags:
  - 實作筆記
---

## 前情提要

用 LINE Bot 做系統通知，結果某天發現通知完全停了，重啟也沒用。
查了一圈才發現根本原因不是程式問題，這篇記錄怎麼查用量、以及為什麼停的。

---

## 問題現象

LINE Bot 的 push 通知在某個時間點之後完全靜默。
程式沒報錯，因為程式碼裡的 push 用了 `.catch(() => {})` 吃掉所有錯誤：

```ts
async function push(text: string) {
  await lineClient.pushMessage(ALLOWED_USER_ID, msg).catch(() => {})
}
```

靜默失敗，完全看不出來哪裡出問題。

---

## 怎麼查用量

LINE 的用量統計不在 Developers Console，要去 **LINE Official Account Manager**：

1. 打開 [manager.line.biz](https://manager.line.biz)
2. 選你的 Official Account
3. 左側 → **分析** → **訊息**

可以看到：

| 欄位 | 說明 |
|---|---|
| 合計（所有訊息）| Reply + Push 總和 |
| Push | 主動推播的則數 |
| Reply | 回覆訊息的則數 |

可以選日期範圍，最長 60 天。

---

## 免費額度是多少

LINE Messaging API 免費方案每月有 **500 則**免費訊息（Push + Reply 合計）。

超過之後每則需要付費，或者訊息會被擋下來（依帳號設定而定）。

500 則聽起來很多，但如果 Bot 跑 AI 對話，Claude 回應很長，常常一次就切成好幾則 push（每則上限 5000 字），很快就會耗光。

---

## 這次的真正原因

查完用量才發現：5/1 用了 141 則，5/2 用了 59 則，5/3 之後全部是 0。

5/3 之後不是「發了但被擋」，是根本沒有觸發 webhook。

原因是 LINE Bot 用 **Webhook 模式**，需要一個對外的 HTTPS URL。
我的架構是：

```text
LINE 伺服器 → HTTPS Webhook URL → Cloudflare Tunnel → 本機 Express server
```

Cloudflare Tunnel 的 launchd service 在 5/3 掛掉，LINE 打不到 webhook，所以什麼都沒發生。

---

## Telegram 為什麼更穩

Telegram 用 **Long Polling** 模式，bot 主動去 Telegram 伺服器拉訊息，不需要對外開放任何 URL。

| | LINE | Telegram |
|---|---|---|
| 連線模式 | Webhook（需要 tunnel）| Long Polling（不需要）|
| 斷線風險 | tunnel 掛、IP 變、PORT 未開 | 幾乎沒有 |
| 免費額度 | 500 則/月 | 無限制 |

如果是用來做系統通知（不可中斷），Telegram 比 LINE 穩定很多。

---

## 小結

- LINE 用量在 LINE Official Account Manager → 分析 → 訊息
- 免費方案 500 則/月，用完靜默失敗
- Webhook 模式依賴 tunnel，tunnel 掛掉通知就停
- 對穩定性要求高的通知場景，Telegram Long Polling 更適合

(fin)
