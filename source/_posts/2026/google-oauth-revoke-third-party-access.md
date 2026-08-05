---
title: "[實作筆記] Google OAuth Refresh Token（三）：第三方應用權限撤銷入口與自動失效條件"
date: 2026/07/26 19:23:29
tags:
  - 實作筆記
---

## 前情提要

我想要為我的 AI 私人祕書加上「標記信件已讀/封存」的功能，但是查詢官方文件後，要達到這個能力的權限，比我想像的大的多

也就是說授權出去的 token，技術上就是有寄信能力，即使我的程式碼永遠不會呼叫寄信的 API。

這種情況下，「我知道怎麼把這個授權收回來」就變成必要的，本篇記錄一下我學到的事。

## 去哪看、去哪撤銷

Google 帳號 → **安全性** → **第三方應用程式和服務**，網址直接是：

```text
https://myaccount.google.com/permissions
```

進去會看到每一個曾經授權過的 App（用 OAuth Client 名稱顯示），點進去可以看到：

- 這個 App 實際拿到的**確切 scope 清單**（不是猜的，是 Google 記錄的真實授權範圍）
- **「移除存取權」**按鈕——按下去，這個 App 手上所有 access token 跟 refresh token 立刻全部失效，之後它想再打 API 一律拿到 `invalid_grant`，要重新走一次完整的授權流程才能恢復

查找一下 `AIris` 這是我的 App 名稱，可以使用確定我們使用的權限，也可以移除存取權。

## Refresh token 會不會自動過期

會，但條件很明確，Google 官方文件列了幾種情況，整理成表：

| 情況 | 說明 |
| --- | --- |
| 6 個月沒被換發 | 不是「沒被呼叫 API」，是 refresh token 拿去跟 Google 換新 access token 這個動作 6 個月沒發生過。正常運作中的服務每次呼叫底層都會自動換發，不會踩到 |
| 使用者主動撤銷 | 就是上面那個「移除存取權」按鈕 |
| 改密碼 | 如果 token 帶 Gmail scope，帳號密碼一改，該 token 就失效 |
| 超過 100 組上限 | 同一個 OAuth Client 對同一個帳號核發超過 100 組 refresh token，最舊的自動作廢 |
| OAuth Client 還在 Testing 發布狀態 | 不管有沒有用，7 天強制過期——這個坑之前踩過一次，見〈[Google OAuth Refresh Token（一）：Testing 模式卡住，只活 7 天](/2026/google-oauth-testing-mode-7-day-refresh-token/)〉，AI 私人祕書這個專案已經發布成 Production，不會再犯 |

前三種是正常使用下該知道的行為；後兩種是個人專案容易忽略的邊界情況。

## 參考

- [Using OAuth 2.0 to Access Google APIs — Refresh token expiration（官方文件）](https://developers.google.com/identity/protocols/oauth2)
- [Google OAuth Refresh Token（一）：Testing 模式卡住，只活 7 天](/2026/google-oauth-testing-mode-7-day-refresh-token/)
- [Google OAuth Refresh Token（二）：Desktop Client 用 loopback 位址手動換 token](/2026/google-oauth-desktop-client-loopback-refresh-token/)

(fin)
