---
title: "[實作筆記] Google OAuth 卡在 Testing 模式，refresh token 只活 7 天"
date: 2026/07/16 01:37:22
tags:
  - 實作筆記
---

## 前情提要

在測試 AIris（自己做的 LINE 分身秘書）的晨間簡報功能時，撞到 Google Calendar/Gmail 的 API 突然回 `invalid_grant`。本機、正式環境用的都是同一組 `GOOGLE_REFRESH_TOKEN`，前幾天都還好好的，這次是真的過期了。查完發現原因很單純：OAuth consent screen 還停在 Testing 模式，跟有沒有在用完全無關，記錄一下排查過程跟解法。

## 症狀

直接拿 `.env` 裡的 `GOOGLE_REFRESH_TOKEN` 打 Google 的 token endpoint 測：

```bash
curl -s -X POST https://oauth2.googleapis.com/token \
  -d "client_id=$GOOGLE_CLIENT_ID" \
  -d "client_secret=$GOOGLE_CLIENT_SECRET" \
  -d "refresh_token=$GOOGLE_REFRESH_TOKEN" \
  -d "grant_type=refresh_token"
```

回應：

```json
{
  "error": "invalid_grant",
  "error_description": "Token has been expired or revoked."
}
```

不是網路問題、不是程式碼問題——Google 自己說這把 token 已經過期或被撤銷。

## 為什麼會過期

Google Cloud Console 的 OAuth consent screen（新介面叫 **Google Auth Platform**）有個「Publishing status」欄位，兩種狀態：

- **Testing**：refresh token 效期固定 **7 天**，不管有沒有呼叫過 API，時間到就是死
- **In production**：refresh token 效期正常，不會這樣莫名其妙過期

去 `console.cloud.google.com` → 選對的專案 → 左側選單「Google Auth Platform」→「Audience」，就能看到目前的 Publishing status。這個專案果然是 **Testing**——難怪 refresh token 活不過一週。

## 怎麼解

同一頁下面就有「Publish app」按鈕，點下去確認就會變成「In production」，refresh token 的 7 天限制就解除了。

幾個原本擔心、後來確認不成立的疑慮：

- **會不會收費？** 不會。發布狀態本身完全免費。只有用到 restricted scope、服務大量外部使用者時，才需要走 Google 的第三方資安審查（CASA），那個才要花錢，跟這裡無關。
- **要不要走完整審核？** 單一使用者、個人用途，不需要。發布後最多是每次授權畫面多一個「Google hasn't verified this app」的警示，點「Advanced → 繼續前往」就過去了，純粹是多一次點擊，不影響功能。
- **User cap 100 人的限制會不會卡到？** 那是 Testing 模式底下才有的限制，而且是算「整個專案生命週期」的累計人數。單一使用者用途完全用不到。

## 小結

Google OAuth 的 refresh token 過期，第一個該查的不是程式碼、不是網路，是 Cloud Console 裡這個專案的 Publishing status 是不是還卡在 Testing。個人專案很容易忘記這一步——建好 OAuth client 能動就沒再理它，7 天後才發現「怎麼又要重新授權」，一直重複同樣的坑。發布成 Production 一次到位，之後就不會再犯。

發布只解決「以後不會再 7 天過期」，舊的 refresh token 還是死的，要重新拿一組——這部分另外寫在下一篇。

## 參考

- [Desktop 類型的 Google OAuth Client，用 loopback 位址手動換 refresh token](/2026/google-oauth-desktop-client-loopback-refresh-token/)

(fin)
