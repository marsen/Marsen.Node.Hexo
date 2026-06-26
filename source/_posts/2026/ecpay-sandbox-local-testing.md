---
title: "[實作筆記] ECPay 綠界沙盒串接：本機測試完整流程"
date: 2026/06/18 05:12:57
tags:
  - 實作筆記
---

## 前情提要

串接 ECPay MPG（多元收款）時，本機測試有幾個坑需要提前知道。
整理一下從申請沙盒帳號到 webhook 打進來的完整流程。

## 申請沙盒帳號

到 ECPay 開發者後台申請測試帳號：

```text
https://vendor.ecpay.com.tw
```

登入後進入「廠商後台」→「系統開發」→「系統介接測試」，
可以拿到沙盒專用的三個憑證：

```ini
ECPAY_MERCHANT_ID=3002607
ECPAY_HASH_KEY=pwFHCqoQZGmho4w6
ECPAY_HASH_IV=EkRm7uxTO9nv1tog
```

這三組是 ECPay 官方的公開測試值，可以直接用。

## 兩個重要 URL 的差別

ECPay MPG 有兩個容易搞混的 URL 參數：

| 參數 | 觸發時機 | 方向 |
|　------　|　---------　|　------　|
| `ReturnURL` | 付款完成後 ECPay 主動通知 | ECPay → 你的 server（POST） |
| `OrderResultURL` | 付款完成後瀏覽器跳回 | ECPay → 使用者瀏覽器（POST redirect） |

兩個都要設，缺一個就會有問題：

- 沒有 `ReturnURL`：訂單狀態永遠不會更新，使用者付了錢系統不知道
- 沒有 `OrderResultURL`：付款完成後使用者停在 ECPay 頁面，不會跳回你的網站

## 本機測試的問題

ECPay 的 `ReturnURL` 是 server-to-server 的 webhook，
ECPay 的機器要能打到你的 server，所以 `localhost` 根本不行。

解法是用 tunnel 工具讓本機有一個公開 URL。
推薦用 **cloudflared**，不要用 localtunnel（理由另篇說明）。

```bash
npx cloudflared tunnel --url http://localhost:3000
```

拿到 URL 之後更新 `.env.local`：

```ini
NEXT_PUBLIC_BASE_URL=https://your-tunnel.trycloudflare.com
```

重啟 dev server，這個 URL 就會被用來組 `ReturnURL` 和 `OrderResultURL`。

## CheckMacValue 驗證

ECPay 打來的 webhook 會帶一個 `CheckMacValue`，
這是用 HASH_KEY 和 HASH_IV 算出來的簽章，用來確認請求是真的來自 ECPay。

驗證流程：

1. 把收到的 POST body 解析成 key-value
2. 移除 `CheckMacValue` 欄位本身
3. 依 key 字母排序，組成 `key=value&key=value` 字串
4. 前後加上 `HashKey=...&` 和 `&HashIV=...`
5. URL encode（小寫）後做 SHA256，轉大寫
6. 比對結果與收到的 `CheckMacValue`

驗證不過要回 `0|FAIL`，成功要回 `1|OK`。
不管成功失敗，HTTP status 都回 200，這是 ECPay 的協定要求。

## 測試信用卡

測試卡號請參考官方「測試介接資訊」頁面，卡號較多且會更新，不在這裡複製。

ECPay 沙盒付款頁網址：

```text
https://payment-stage.ecpay.com.tw/Cashier/AioCheckout/index
```

這個頁面上的所有付款方式（包括 Apple Pay）都是沙盒，不會真實扣款。

## 常見錯誤

### 付款失敗

通常是 CheckMacValue 算錯。
確認 HASH_KEY、HASH_IV 正確，URL encode 用的是 `encodeURIComponent` 後轉小寫（不是 `encodeURI`）。

### 付款成功但訂單沒更新

`ReturnURL` 沒收到 webhook callback。

1. 地端測試時，可以檢查tunnel 有沒有在跑
2. `ReturnURL` 有沒有帶到正確的 tunnel URL
3. dev server log 有沒有收到 POST

### 付款完成後使用者停在 ECPay 頁面

`OrderResultURL` 沒設或設錯，ECPay 不知道要跳回哪裡。

**redirect 跑到 `https://localhost:3000`**

在 API Route Handler 裡用 `request.url` 拿 origin 時，
cloudflared 會加 `X-Forwarded-Proto: https`，導致 Next.js 認為 origin 是 `https://localhost:3000`。
解法是直接讀 `NEXT_PUBLIC_BASE_URL` 環境變數，不要從 request 推。

## 參考

- [ECPay 官方：測試介接資訊（含測試卡號）](https://developers.ecpay.com.tw/2856/)
- [本地 webhook 測試：用 cloudflared，不要用 localtunnel](/2026/cloudflared-vs-localtunnel/)

## 小結

ECPay 沙盒串接的關鍵點：

1. `ReturnURL` 和 `OrderResultURL` 都要設，職責不同
2. 本機測試一定要用 tunnel，推薦 cloudflared
3. CheckMacValue 驗證不能省，這是確認 webhook 來源的唯一機制
4. API Route 裡的 redirect URL 要從環境變數讀，不要從 request 推

(fin)
