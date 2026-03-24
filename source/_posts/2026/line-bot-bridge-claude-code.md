---
title: "[實作筆記] 用 Cloudflare Tunnel 讓 LINE Bot Webhook 打到本機"
date: 2026/03/19 06:11:45
tags:
  - 實作筆記
---

## 前情提要

在本機開發時需要 Webhook 來接受第三方的回呼，

例如「LINE Bot 的 Messaging API 」 通常需要真實的 HTTPS endpoint 的 Callback Url，

第一反應是用 ngrok，指令一行就能跑：

但是每次重開 URL 都會換不一樣，需要再去設定一次。

所以我決定換個做法。

選 **Cloudflare Named Tunnel** 的原因：

- URL 固定（綁自己的 domain），設定一次就好
- 已有 Cloudflare 管理 domain，沒有額外成本
- 搭配 launchd 開機自啟，幾乎感覺不到它的存在

前兩項是必要前置條件，剛好我都有，沒有的小朋友可能需要設定一下

或是找其它解法，文末有提供一些別的方法給你參考

## 前置條件

- 已有 Cloudflare 帳號
- 要使用的 domain 已托管在 Cloudflare（NS 已指向 Cloudflare）
- 本機 LINE Bot 已可正常啟動並監聽某個 port

---

## 為什麼選 Named Tunnel，不用 Quick Tunnel

Quick Tunnel 每次啟動 URL 都會變，要一直去 LINE Console 更新 Webhook URL，很麻煩。

Named Tunnel 設定一次，URL 固定（綁自己的 domain），之後只管跑就好。

---

## 設定步驟

本文以 tunnel 名稱 `mybot`、domain `mybot.example.com`、本機 port `3000` 為例。

### 1. 安裝 cloudflared

```bash
brew install cloudflare/cloudflare/cloudflared
```

### 2. 登入

```bash
cloudflared tunnel login
```

瀏覽器會開啟授權頁面，選擇要綁定的 domain。

### 3. 建立 Named Tunnel

```bash
cloudflared tunnel create mybot
```

會產生一組 tunnel ID 跟 credentials 檔，路徑類似 `~/.cloudflared/<tunnel-id>.json`。

### 4. 設定 config

寫入 `~/.cloudflared/config.yml`：

```yaml
tunnel: <tunnel-id>
credentials-file: /Users/<user>/.cloudflared/<tunnel-id>.json

ingress:
  - hostname: mybot.example.com
    service: http://localhost:3000
  - service: http_status:404
```

`<tunnel-id>` 從 Step 3 的輸出取得，或執行 `cloudflared tunnel list` 查看。

### 5. 設定 DNS

```bash
cloudflared tunnel route dns mybot mybot.example.com
```

Cloudflare DNS 會自動新增一筆 CNAME 指向 tunnel，不用手動加。

### 6. 跑起來

```bash
cloudflared tunnel run mybot
```

這樣 `https://mybot.example.com` 的流量就會轉進 `localhost:3000`。

---

## LINE Console 設定

**注意：Verify 前要確認 bot 已在本機跑起來，** LINE 會實際打一個請求過來，bot 沒跑就會 Verify 失敗。

LINE Developers Console → Messaging API → Webhook URL 填：

```text
https://mybot.example.com/webhook
```

勾選「Use webhook」→ 點「Verify」，回傳 200 就完成了。

---

## Port 選擇

LINE 打來的 Webhook 是外部流量，建議選高號碼 port（49152–65535），
比常見的 3000、8080 不容易被掃描到。
在 `config.yml` 的 `service` 欄位對應好本機 port 即可。

---

## 其他常見做法與沒選的原因

| 方案 | 為什麼沒選 |
| ------ | ----------- |
| **ngrok** | 免費版 URL 每次重啟都變，跟 Quick Tunnel 一樣的問題 |
| **localtunnel** | 開源，URL 可指定但不保證穩定，偶有被佔用的情況 |
| **smee.io** | Webhook 轉發專用，需要額外的 client 程式，多一層複雜度 |
| **VS Code Dev Tunnels** | 綁定 VS Code，換編輯器就失效 |
| **Tailscale Funnel** | 要先建 Tailscale 網路，對沒在用的人成本高 |
| **直接部署到雲端** | URL 穩定，但每次改 code 都要重部署，開發迭代太慢 |
| **Cloudflare Quick Tunnel** | URL 每次重啟都換，手動更新 Webhook 很煩 |

## 小結

Cloudflare Named Tunnel 解決了「本機服務需要公開 HTTPS endpoint」的問題，
對 LINE Bot 開發來說特別實用——URL 固定，不用每次重啟都更新 Webhook 設定。
搭配 launchd 讓 cloudflared 開機自動啟動，幾乎可以忘記它的存在。

(fin)
