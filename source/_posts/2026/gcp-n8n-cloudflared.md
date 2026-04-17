---
title: "[實作筆記] 用 cloudflared tunnel 讓 n8n 安全對外，不開防火牆"
date: 2026/04/17 14:54:04
tags:
  - 實作筆記
---

## 前情提要

上一篇在 GCP 免費 VM 上把 n8n 跑起來了，但還沒辦法從外部存取。

這篇用 cloudflared tunnel 解決這個問題。

好處是：
- 不用改 GCP 防火牆，不暴露任何 port
- 自動 HTTPS
- 可以加 Cloudflare Access（Google 帳號驗證）擋在前面

---

## 命名決策

這台 VM 不只跑 n8n，之後可能還有其他自動化服務。
所以 tunnel 和 subdomain 都以機器用途命名，而不是以某個服務命名。

選 `butler`（管家），因為這台機器的定位是「幫你自動處理事情的後台」。

- Tunnel 名稱：`butler`
- 對外網址：`butler.marsen.me`

之後如果加新服務，在 config 裡多加一條 ingress 規則就好。

---

## 步驟一：登入 Cloudflare

在 VM 上跑：

```bash
cloudflared tunnel login
```

它會印出一個 URL，複製到瀏覽器，選你要授權的 domain（這裡是 `marsen.me`）。

授權完成後 cert 會自動存到：

```text
~/.cloudflared/cert.pem
```

---

## 步驟二：建立 Tunnel

```bash
cloudflared tunnel create butler
```

成功後會顯示 tunnel ID（UUID 格式），同時在 `~/.cloudflared/` 產生一個 credentials JSON 檔。

```text
Tunnel credentials written to /home/USER/.cloudflared/<tunnel-id>.json
Created tunnel butler with id <tunnel-id>
```

把 tunnel ID 記下來，後面要用。

---

## 步驟三：建立 config 檔

```bash
cat > ~/.cloudflared/config.yml << 'EOF'
tunnel: <your-tunnel-id>
credentials-file: /home/<USER>/.cloudflared/<your-tunnel-id>.json

ingress:
  - hostname: butler.marsen.me
    service: http://localhost:5678
  - service: http_status:404
EOF
```

注意最後的 `EOF` 必須從行首開始，前面不能有任何空格。

驗證 config 是否正確：

```bash
cloudflared tunnel ingress validate
```

看到 `OK` 就是正確。

---

## 步驟四：新增 DNS 記錄

```bash
cloudflared tunnel route dns butler butler.marsen.me
```

這會在 Cloudflare DNS 自動加一筆 CNAME，指向你的 tunnel。

---

## 步驟五：啟動 Tunnel（待補）

---

## 步驟六：設定開機自動啟動（待補）

---

## 步驟七：Cloudflare Access（待補）

加一層 Google 帳號驗證，讓 `butler.marsen.me` 不是誰都能開。

---

## 小結

（待補）

(fin)
