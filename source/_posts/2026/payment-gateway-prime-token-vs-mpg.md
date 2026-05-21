---
title: "[工具筆記] Prime Token vs MPG：兩種台灣電商金流模式的本質差異"
date: 2026/05/21 11:36:44
tags:
  - 工具筆記
---

## 前情提要

最近在電商專案要把虛構金流換成真實串接，比較了 TapPay、綠界、藍新三家。
比著比著發現一件事：與其問「哪家好」，不如先搞懂兩種模式的差別。

- **Prime Token 模式**：TapPay、Stripe、Braintree
- **MPG 模式**：綠界 ECPay、藍新 NewebPay

兩種模式選錯，後面整個訂單系統的設計都會錯。

---

## 本質差異就一個問題：卡號從哪走？

兩種模式的根本問題只有一個：**敏感卡號（PAN + CVV）的傳遞路徑**。

### Prime Token

```text
使用者 → [前端 SDK iframe 直送金流方] → 拿 token
                                       ↓
你的 server 用 token + 商家 key → 打金流方 charge API → 同步拿結果
```

### MPG

```text
你的 server 產表單 + 簽章 → 前端 POST 到金流方頁
                              ↓
                  使用者離站填卡號（在金流方頁面）
                              ↓
              金流方非同步 callback 回你的 server
```

Prime Token 把「拿卡號」**SDK 化**，UX 由你掌握。
MPG 把「拿卡號」**頁面化**，工程量降低但 UX 受限。

---

## 安全性：兩種都不用 PCI-DSS，但風險點不同

兩種模式對開發者都**不需要 PCI-DSS 認證**，因為卡號從未進你的伺服器。
但要擔心的事情不一樣：

| 風險 | Prime Token | MPG |
| --- | --- | --- |
| 前端 SDK 被換掉（供應鏈攻擊、CSP 設錯） | ⚠️ 卡號可能外流 | 不受影響 |
| Callback 偽造 | 不存在 | ⚠️ 沒驗章就被偽造付款成功 |
| Replay attack | 不存在 | ⚠️ 需做冪等 |
| HTTPS 不夠用嗎？ | 夠 | 必須再加簽章 |

MPG 最容易出包的是這三件事：

1. **簽章寫錯**（綠界用 CheckMacValue / 藍新用 AES + SHA256）
2. **沒做冪等**（同一筆 callback 多次到結果重複出貨）
3. **相信 ReturnURL 而不等 NotifyURL**（ReturnURL 是給使用者看的，可以被竄改）

---

## UX 差異：設計語言會不會中斷

Prime Token 的卡號欄位內嵌結帳頁（實際是 iframe，能改邊框字體但限制多）。
整個結帳流程在你自己的頁面，設計語言一致。

MPG 直接跳到金流方頁面，**你的品牌設計到那一刻全部斷掉**。
但好處也很實際：同一個金流頁面能直接接信用卡 + ATM + 超商代碼 + LINE Pay。

簡單講：

- 想做品牌感、UX 要順 → Prime Token
- 想接齊台灣常見付款方式 → MPG

---

## 工程量反直覺：MPG 比 Prime Token 大

很多人以為「使用者離站填卡號 = 我不用做」，這是錯的。

Prime Token 你寫的東西：

- 前端 SDK 載入 + mount 欄位
- 後端一支 charge API
- 一個結果頁

MPG 你多寫的東西：

- 表單產生器（含簽章演算法）
- 自動 submit 的轉址頁
- **NotifyURL endpoint**（驗章 + 冪等 + 更新訂單，這支是核心）
- **ReturnURL endpoint**（使用者回站只看結果，**不可以**當付款依據）
- 訂單狀態機要多 `awaiting_payment`
- 對帳邏輯（callback 沒到怎辦？要定時 query 嗎？）

MPG 看似簡單，實際工程量比 Prime Token 大，**主要是 callback 的 edge case 多**。

---

## 同步 vs 非同步的訂單狀態機

這是最常被低估的差異。

### Prime Token（同步）

```text
[pending] --charge API--> [paid] / [failed]
```

單一請求週期內完成，UI 邏輯簡單。

### MPG（非同步）

```text
[pending] → [awaiting_payment]
              ↓
        使用者離站
              ↓
   ┌──────────┼──────────┐
   ↓          ↓          ↓
NotifyURL  使用者關頁面  NotifyURL 沒到
到（驗章 OK）(ReturnURL 沒到) (網路斷)
   ↓          ↓          ↓
[paid]/    [awaiting   [awaiting
 [failed]   _payment]   _payment]
                        ↓
                    需要對帳補完
```

`awaiting_payment` 這個狀態在 Prime Token 模式不存在，
是 MPG 必須補的領域概念。沒處理就會出現「使用者跳走後訂單卡住」的情況。

---

## 選擇心法

| 情境 | 建議 |
| --- | --- |
| 只賣信用卡、要做品牌感、UX 優先 | Prime Token（TapPay / Stripe） |
| 台灣 C2C、想接 ATM/超商、客單低 | MPG（綠界 / 藍新） |
| B2B 大額付款、年費月費 | Prime Token + 卡號 tokenize 存起來定期扣 |
| 開發者個人專案、要快 | 綠界 MPG（零註冊，公開測試憑證） |
| 想學金流原理 | 兩個都做一遍 |

---

## 混合策略：兩種一起上才是正解

認真的台灣電商常常**兩種都接**：

- **信用卡**走 Prime Token（自家品牌、UX 順）
- **ATM / 超商 / LINE Pay** 走 MPG（方便、台灣使用者熟悉）

不是因為要備援，而是**不同付款方式天生適合不同模式**。

要做成這樣，金流抽象層的 `PaymentService` port 必須能同時表達兩種流程：

```ts
type PaymentIntent =
  | { kind: 'sync-result'; result: PaymentResult }              // Prime Token
  | { kind: 'redirect'; url: string; formFields: Record }       // MPG
```

上層 use case 不用知道是哪家，但能正確處理「同步拿結果」與「轉址等 callback」兩種分支。

---

## 小結

- 兩種模式的本質差異是卡號的傳遞路徑：Prime Token 走 SDK iframe，MPG 走表單轉址
- 兩種都不需 PCI-DSS，但 MPG 多了驗章、冪等、ReturnURL 不可信三件事要顧
- 工程量反直覺：MPG 比 Prime Token 大，貴在 callback 的 edge case
- 訂單狀態機 MPG 多一個 `awaiting_payment`，這個概念漏了就會出包
- 真正的解法不是二選一，而是設計能同時支撐兩種模式的抽象層

(fin)
