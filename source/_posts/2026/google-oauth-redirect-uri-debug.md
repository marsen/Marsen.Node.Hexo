---
title: "[實作筆記] Google OAuth redirect_uri_mismatch 除錯記錄"
date: 2026/04/27 23:28:15
tags:
  - 實作筆記
---

## 前情提要

`demo.marsen.me` 的 Google 登入突然失敗，報 `redirect_uri_mismatch`。

表面上「什麼都沒改」，但往前追才發現是另一個修復動作的連帶效果。

這篇記錄完整的前因後果。

---

## 事件時間線

### 起點：Instagram OAuth callback 回錯誤的 URL

在設定 IG Token 工具時，Instagram 授權完成後，callback 沒有導回 `https://demo.marsen.me/admin/tools/ig-token`，而是跑到 Vercel 內部 URL。

原因：Instagram callback route 的 `baseUrl` 沒有設定，fallback 讀取的是 Vercel 執行環境的 host，不是對外的 production domain。

### 修法：設定 `NEXT_PUBLIC_BASE_URL`

在 Vercel 環境變數加上：

```text
NEXT_PUBLIC_BASE_URL=https://demo.marsen.me
```

因為專案用 `vercel build --prebuilt` 方案，直接 Redeploy 不會重新 build，必須 push 新 commit 觸發 GitHub Actions：

```bash
git commit --allow-empty -m "chore: trigger redeploy"
git push origin main
```

Instagram OAuth callback 正常了。

### 連帶損傷：Google 登入壞了

`NEXT_PUBLIC_BASE_URL` 同時影響 Google OAuth 的 redirect URI（`src/app/api/auth/google/route.ts`）：

```ts
const google = new Google(
  env.googleClientId,
  env.googleClientSecret,
  `${env.baseUrl}/api/auth/google/callback`,
)
```

原本 `NEXT_PUBLIC_BASE_URL` 是 `http://localhost:3000`，Google Cloud Console 只登記了：

```text
http://localhost:3000/api/auth/google/callback
```

設成 `https://demo.marsen.me` 之後，app 送出的 redirect_uri 變成：

```text
https://demo.marsen.me/api/auth/google/callback
```

Google Console 沒有這條，報 `redirect_uri_mismatch`。

---

## 症狀

點「Google 登入」後，Google 跳出錯誤頁面：

```text
已封鎖存取權：這個應用程式的要求無效
發生錯誤 400：redirect_uri_mismatch
```

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

- Instagram OAuth callback URL 錯誤 → 設 `NEXT_PUBLIC_BASE_URL` → Google 登入壞掉，是一連串的連帶效果
- `redirect_uri_mismatch` 代表 app 送出的 redirect URI 和 Google Console 登記的不符
- 改了 `NEXT_PUBLIC_BASE_URL` 要同步更新所有用到這個變數的 OAuth 服務的 Console 設定
- Google Console 可以同時登記多條 URI，本機和 production 各一條，互不影響
- 用 `vercel build --prebuilt` 的專案，env var 更新後要 push 空 commit 才會生效

(fin)
