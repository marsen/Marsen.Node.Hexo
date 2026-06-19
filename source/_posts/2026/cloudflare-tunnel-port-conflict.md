---
title: "[實作筆記] Cloudflare Tunnel 測試金流時遇到 404：Port 衝突雷點"
date: 2026/06/19 20:02:49
tags:
  - 實作筆記
---

## 前情提要

用 Cloudflare Tunnel 測試藍新金流 webhook 時，
付款完成後被 redirect 回來一直 404。
紀錄一下這個雷點。

## 問題

Cloudflare Tunnel 設定指向 `localhost:3000`。

但啟動 `pnpm dev` 時，3000 port 已被其他 process 佔用，
Next.js 自動改用 3001：

```text
⚠ Port 3000 is in use by process 65277, using available port 3001 instead.
▲ Next.js 16.1.6
- Local: http://localhost:3001
```

tunnel 不知道這件事，還是把流量打到 3000。
3000 的 process 沒有 `/api/newebpay/result` 這個路由，所以 404。

## 為什麼難發現

本機開瀏覽器 `http://localhost:3001` 完全正常，
app 功能也沒問題，只有走 tunnel 的路徑才壞。

金流 ReturnURL 是藍新打回來的，不是自己瀏覽器直接連，
所以測試到最後一步才踩到。

## 解法

先砍掉佔用 3000 的 process，再重啟 dev server：

```bash
lsof -ti:3000 | xargs kill -9
pnpm dev
```

dev server 就會正常跑在 3000，tunnel 不用動。

## 參考

- [本地 webhook 測試：用 cloudflared，不要用 localtunnel](/2026/cloudflared-vs-localtunnel/)

## 小結

tunnel 的 port 是靜態設定，Next.js 的 port fallback 是動態行為，
兩個各自獨立運作，對不上就 404。

開始測試前先確認 dev server 跑在哪個 port，tunnel 再對齊它。

(fin)
