---
title: "[工具筆記] 本地 webhook 測試：用 cloudflared，不要用 localtunnel"
date: 2026/06/18 05:02:58
tags:
  - 工具筆記
---

## 前情提要

在本機開發時，第三方金流（ECPay、TapPay）的 webhook 需要一個公開 URL 才能打回來。
我試了 localtunnel，然後換成 cloudflared，體驗差很多，記錄一下。

## localtunnel 的問題

localtunnel 安裝很簡單：

```bash
npx localtunnel --port 3000
```

但實際用起來有幾個痛點：

**不穩定**。tunnel 動不動就斷，斷了 URL 就換掉，要重新設定金流後台的 webhook URL。
測到一半付款成功，webhook 送過來的時候 tunnel 已經掛了，log 什麼都沒有，很難 debug。

**IP 驗證頁**。localtunnel 為了防濫用，第一次打這個 URL 會先跳出一個「請輸入你的 IP」確認頁面。
金流打 webhook 的時候是自動 POST，不是人在點，所以它打到的是那個驗證頁，根本進不來。

解法是在 header 加 `bypass-tunnel-reminder: true`，但第三方的 webhook request 你改不了，等於這個問題無解。

## cloudflared：免安裝，開箱即用

cloudflared 是 Cloudflare 官方出的 tunnel 工具。
最棒的一點：**不需要帳號，不需要安裝**，一行指令直接跑：

```bash
npx cloudflared tunnel --url http://localhost:3000
```

跑起來會拿到一個 `trycloudflare.com` 的 URL，例如：

```text
https://rolled-intro-operator-meat.trycloudflare.com
```

沒有 IP 驗證頁，webhook 直接打進來，不會被擋。
穩定度比 localtunnel 好很多，整個測試過程沒斷過。

## 快速比較

| | localtunnel | cloudflared |
|---|---|---|
| 安裝 | `npm i -g localtunnel` | 不需要 |
| 帳號 | 不需要 | 不需要 |
| 穩定度 | 容易斷 | 穩定 |
| IP 驗證頁 | 有（webhook 無法繞過） | 無 |
| URL 固定 | 否 | 否 |
| 免費 | 是 | 是 |

## 使用方式

```bash
npx cloudflared tunnel --url http://localhost:3000
```

拿到 URL 之後，把它設到需要公開 URL 的地方，例如 `.env.local`：

```ini
NEXT_PUBLIC_BASE_URL=https://rolled-intro-operator-meat.trycloudflare.com
```

然後重啟 dev server，金流後台的 webhook URL 也一起更新，就可以測了。

每次重新跑 URL 會換，這點跟 localtunnel 一樣，需要重設一次。
但只要 tunnel 不斷，URL 就是固定的，這點比 localtunnel 好太多。

## 小結

本機測 webhook，直接用 cloudflared，不要碰 localtunnel。
免安裝、沒有驗證頁擋路、穩定，完全沒有理由繼續用 localtunnel。

(fin)
