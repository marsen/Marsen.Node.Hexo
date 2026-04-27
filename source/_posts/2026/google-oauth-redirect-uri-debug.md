---
title: "[實作筆記] Google OAuth redirect_uri_mismatch 除錯記錄"
date: 2026/04/27 23:28:15
tags:
  - 實作筆記
---

## 前情提要

`demo.marsen.me` 的 Google 登入突然失敗，報 `redirect_uri_mismatch`。什麼都沒改，就壞了。

這篇記錄找到根本原因的過程。

---

## 症狀

點「Google 登入」後，Google 跳出錯誤頁面：

```text
已封鎖存取權：這個應用程式的要求無效
發生錯誤 400：redirect_uri_mismatch
```

---

## 找原因

### 看程式碼

Google OAuth 的 redirect URI 是這樣組的（`src/app/api/auth/google/route.ts`）：

```ts
const google = new Google(
  env.googleClientId,
  env.googleClientSecret,
  `${env.baseUrl}/api/auth/google/callback`,
)
```

`env.baseUrl` 來自環境變數 `NEXT_PUBLIC_BASE_URL`。

### 問題出在哪

原本 `NEXT_PUBLIC_BASE_URL` 設的是 `http://localhost:3000`（本機開發環境），Google Cloud Console 也只登記了：

```text
http://localhost:3000/api/auth/google/callback
```

後來為了修另一個問題，把 `NEXT_PUBLIC_BASE_URL` 改成 `https://demo.marsen.me`。

app 送出的 redirect_uri 就變成：

```text
https://demo.marsen.me/api/auth/google/callback
```

但 Google Console 裡沒有這條，直接 mismatch。

---

## 解法

進 Google Cloud Console → Credentials → OAuth 2.0 Client → Authorized redirect URIs，新增：

```text
https://demo.marsen.me/api/auth/google/callback
```

本機那條 `http://localhost:3000/api/auth/google/callback` 保留，本機開發還用得到。

存檔幾分鐘內生效，不需要重新 build 或 deploy。

---

## 小結

- `redirect_uri_mismatch` 代表 app 送出的 redirect URI 和 Google Console 登記的不符
- 改了 `NEXT_PUBLIC_BASE_URL` 就要同步更新 Google Console 的 Authorized redirect URIs
- Google Console 可以同時登記多條 URI，本機和 production 各一條，互不影響

(fin)
