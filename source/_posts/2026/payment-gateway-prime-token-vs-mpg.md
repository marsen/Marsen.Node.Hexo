---
title: "[工具筆記] Prime Token vs MPG：兩種台灣電商金流模式的本質差異"
date: 2026/05/21 11:36:44
tags:
  - 工具筆記
---

## 前情提要

最近在電商專案要把虛構金流換成真實串接，比較了 TapPay、綠界、藍新三家。

比著發現原來存在著兩種金流串接的模式。

- **Prime Token 模式**：TapPay、Stripe、Braintree
- **MPG(Merchant Payment Gateway) 模式**：綠界 ECPay、藍新 NewebPay

兩種模式選錯，後面整個訂單系統的設計都會錯。

---

## 本質差異就一個問題：卡號從哪走？

兩種模式的根本問題只有一個：**敏感卡號（PAN + CVV）的傳遞路徑**。

MPG 型金流（綠界、藍新跳轉）對消費者反而更透明——畫面明確跳到金流商的頁面，消費者看得出來自己在哪。

但是在使用者體驗上，會多一個斷點，以電商來說會有轉換率的問題

Prime 型（TapPay iframe）的取捨是體驗好但透明度低，信任靠的是商家品牌，不是技術可見性。

而這是一個取捨。

---

## 實作細節

Prime Token 的卡號欄位內嵌結帳頁（實際是 iframe，能改邊框字體但限制多）。

整個結帳流程在你自己的頁面，設計語言、外觀風格一致。

MPG 直接跳到金流方頁面，**你的品牌設計到那一刻全部斷掉**。

但好處也很實際：同一個金流頁面能直接接信用卡 + ATM + 超商代碼 + LINE Pay。

兩者的開發量比較

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

## 補充: PCI DSS 是什麼 ?

Payment Card Industry Data Security Standard，簡稱 PCI DSS。

信用卡組織（Visa、Mastercard 等）聯合制定的資安標準，規定任何「碰到卡號」的系統都要符合一堆安全要求。

核心概念：碰到卡號的範圍越小，你要過的關越少。

實務上只有金流商或大型電商會去取得這種認証，一般而言小型電商還是透過第三方金流來實現交易。

---

## 小結

- 兩種模式的本質差異是卡號的傳遞路徑：Prime Token 走 SDK iframe，MPG 走表單轉址
- 信任是最難的

(fin)
