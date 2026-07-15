---
title: "[實作筆記] Desktop 類型的 Google OAuth Client，用 loopback 位址手動換 refresh token"
date: 2026/07/16 01:48:23
tags:
  - 實作筆記
---

## 前情提要

AIris 的 Google refresh token 過期了（見〈Google OAuth Testing 模式，refresh token 只活 7 天〉那篇），想說最快的方式是用 [OAuth 2.0 Playground](https://developers.google.com/oauthplayground/) 重新走一次授權，結果卡住——原因跟 Testing/Production 無關，是另一個坑：**這個 OAuth client 的類型是 Desktop app，不是 Web application**。記錄一下 Desktop 類型該怎麼手動拿 refresh token。

## 為什麼 OAuth Playground 用不了

OAuth Playground 的做法是把自己的網址（`https://developers.google.com/oauthplayground`）註冊成你的 OAuth client 的「Authorized redirect URI」，然後借用你的 client 走一次真實授權。

問題是：Google Cloud Console 裡的 OAuth client 分兩種類型，能不能自由登記 redirect URI 是不一樣的行為：

| Client 類型 | Redirect URI 規則 |
|---|---|
| Web application | 可以登記任意 HTTPS 網址（例如 OAuth Playground 那個） |
| Desktop app | 只能用 `http://localhost:任意port` 或 `http://127.0.0.1:任意port`，不能登記其他網址 |

AIris 這個 client 建立時選的是 Desktop app（合理，因為它本來就是跑在自己主機上的背景服務，不是網頁應用），所以沒辦法把 OAuth Playground 的網址加進去。

## Desktop app 的解法：loopback 位址流程

Google 對 Desktop/CLI 這類「沒有公開伺服器」的程式，設計了專屬流程：redirect_uri 直接指向 `localhost` 的任意 port，不用預先登記，Google 就是允許這樣做。

拿到新 refresh token 分三步：

### 1. 組出授權網址，瀏覽器打開

```text
https://accounts.google.com/o/oauth2/v2/auth?
  client_id=你的CLIENT_ID
  &redirect_uri=http://localhost:8080
  &response_type=code
  &scope=https://www.googleapis.com/auth/calendar.readonly https://www.googleapis.com/auth/gmail.readonly
  &access_type=offline
  &prompt=consent
```

幾個參數的用意：

- `scope`：只填程式碼實際會用到的唯讀範圍，不多要權限
- `access_type=offline`：預設（`online`）只給一小時就過期的 access token，沒有 refresh token；要拿 refresh token 一定要加這個
- `prompt=consent`：同一個 client + scope + 帳號，Google 通常只在「第一次」同意時發 refresh token，之後重複走流程可能會偷懶不給新的；強制加這個保證這次一定拿到新的

### 2. 從網址列複製 code，不用管頁面內容

登入帳號、按「允許」之後，瀏覽器會被導去 `http://localhost:8080/?code=xxxxx`。

這個頁面會顯示「無法連上這個網站」——完全正常，因為根本沒有東西在監聽 8080 這個 port。但沒關係，Google 是先把 `code` 塞進網址列、瀏覽器才嘗試連線，所以連線失敗不影響拿到 code：直接從網址列複製 `code=` 後面那一串就好。

### 3. 用 code 換正式的 token

```bash
curl -X POST https://oauth2.googleapis.com/token \
  -d "client_id=你的CLIENT_ID" \
  -d "client_secret=你的CLIENT_SECRET" \
  -d "code=剛剛複製的那串" \
  -d "grant_type=authorization_code" \
  -d "redirect_uri=http://localhost:8080"
```

`redirect_uri` 要跟第一步組網址時完全一致，Google 會拿來比對。回應的 JSON 裡 `refresh_token` 欄位就是新值，直接拿去換掉 `.env` 裡的舊值。

`client_secret` 全程只出現在這個 `curl` 呼叫裡——這是刻意的：`client_secret` 不能出現在瀏覽器網址列或任何前端看得到的地方，只能在「後端對後端」（這裡用 `curl` 模擬）的請求裡使用，這也是為什麼拿 code 換 token 一定要多這一步，不能讓瀏覽器直接拿到最終的 token。

## 小結

Desktop 類型的 OAuth client 拿 refresh token，不能套用 Web app 常見的「拿一個現成的第三方工具（像 OAuth Playground）代勞」這條路，因為 redirect URI 的規則不一樣。老實走一次 loopback 位址流程（開網址 → 從網址列複製 code → curl 換 token）三步就搞定，不需要真的架一個 server 去接那個 redirect。

## 參考

- [Google OAuth Testing 模式，refresh token 只活 7 天](/2026/google-oauth-testing-mode-7-day-refresh-token/)

(fin)
