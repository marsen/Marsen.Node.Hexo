---
title: "[實作筆記] 設定 Resend 自訂 Tracking Domain，開啟 Open & Click Tracking"
date: 2026/06/24 05:40:35
tags:
  - 實作筆記
---

## 前情提要

在 Arsenal 上做了一個聯絡表單，寄通知 email 用 Resend。
設定的過程中看到 Open and Click Tracking 這個功能，想開，但開之前要先設定自訂 tracking subdomain。

順手記一下為什麼要設、怎麼設。

## 為什麼不能直接用 Resend 預設的 tracking domain

Open Tracking 的原理是在 email 裡塞一個 1px 透明圖片，收件人開信時瀏覽器載入這張圖，Resend 就知道「被打開了」。
Click Tracking 則是把 email 裡的連結換成中繼 URL，收件人點了之後先到 Resend，記錄一筆，再跳到原始連結。

預設這兩個 URL 的 domain 都是 Resend 的（例如 `track.resend.dev`）。

問題有兩個：

1. **deliverability**：Gmail、Outlook 的垃圾信過濾會注意 email 內的 domain 跟寄件人是否一致。圖片跟連結指向一個陌生的 domain，可疑分數會上升。

2. **共用 reputation**：Resend 的 tracking domain 是所有用戶共用的。如果有人用 Resend 大量發垃圾信被封，你也可能受連帶影響。

設定自己的 subdomain 可以解決這兩個問題。

## 設定步驟

### 1. 在 Resend 找到 Domain 設定

進 Resend Dashboard → Domains，點開你的 domain（這個 domain 要已經驗證完成，也就是已經能寄信的那個）。

### 2. 開啟 Tracking

在 domain 設定頁面找到 **Tracking** 區塊，開啟 Open Tracking 和 Click Tracking。

開啟後 Resend 會給你一筆 CNAME 紀錄要加到 DNS：

```text
類型：CNAME
名稱：tracking
值：links1.resend-dns.com
```

最終效果是你的 `tracking.yourdomain.com` 會指向 Resend 的 tracking endpoint。

### 3. 在 DNS 加 CNAME 記錄

如果 DNS 是用 Cloudflare 管的，Resend 有直接整合——在 Resend 介面授權連結 Cloudflare，它會自動幫你加好那筆 CNAME，不需要手動操作。

其他 DNS 提供商就要自己去加。Cloudflare 的話記得把 Proxy 設成 DNS only（灰色雲），CNAME 才能正常傳遞。

### 4. 等 DNS 生效

一般 5 分鐘到幾小時，Cloudflare 通常很快。

回到 Resend 按 Verify，綠燈就是設定完成。

## 開啟後的行為

設定完之後，Resend 寄出去的 email 裡：

- 圖片 URL 會從 `track.resend.dev/open/xxx` 變成 `track.yourdomain.com/open/xxx`
- 連結會從 `track.resend.dev/click/xxx` 變成 `track.yourdomain.com/click/xxx`

都是你自己的 domain，deliverability 更好。

## 程式碼端不需要改

這是 Resend 的 infrastructure 層設定，你的程式碼不需要動任何東西。只要 domain 驗證完成，之後的每封信都自動套用。

## 小結

整個設定大概 10 分鐘。核心就一件事：加一筆 CNAME 到 DNS，把 tracking 的 subdomain 接到 Resend。

之後 open rate 和 click rate 就能在 Resend Dashboard 看到了。

(fin)
