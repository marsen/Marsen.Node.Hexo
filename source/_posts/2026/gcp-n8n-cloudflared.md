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

## 步驟五：啟動 Tunnel

```bash
cloudflared tunnel run butler
```

看到 `Registered tunnel connection` 就是連上了。

### 踩坑一：localhost 解析成 IPv6

第一次跑時 config 寫的是 `http://localhost:5678`，
cloudflared 把 `localhost` 解析成 IPv6 的 `[::1]`，但 n8n 只監聽 IPv4，所以連不到。

解法：把 config 裡的 `localhost` 改成 `127.0.0.1`：

```bash
sed -i 's|http://localhost:5678|http://127.0.0.1:5678|' ~/.cloudflared/config.yml
```

### 踩坑二：n8n connection reset by peer

tunnel 連上但瀏覽器開不了，先查 n8n log：

```bash
sudo docker logs n8n --tail 20
```

若出現 `EACCES: permission denied, open '/home/node/.n8n/config'`，問題在 volume 目錄的 owner。

**根本原因**：`~/.n8n` 是第一次啟動時由 Docker（root）自動建立的，
但容器裡的 `node` 用戶（UID 1000）不是 root，所以沒有寫入權限。

解法：把 host 上的 `~/.n8n` 改成讓 `node`（UID 1000）擁有，再重建容器：

```bash
sudo chown -R 1000:1000 ~/.n8n
sudo docker rm -f n8n
sudo docker run -d \
  --name n8n \
  --restart unless-stopped \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  -e N8N_SECURE_COOKIE=false \
  docker.n8n.io/n8nio/n8n
```

n8n 映像檔本來就用非 root 的 `node` 用戶執行，不需要加 `--user`。

---

## 步驟六：設定開機自動啟動

讓 cloudflared 在 VM 重開機後自動啟動，不用每次手動跑。

`cloudflared service install` 預設找 `~/.cloudflared/config.yml`，
但用 `sudo` 跑時 home 會變成 root 的，所以要明確指定 config 路徑：

```bash
sudo cloudflared --config /home/<USER>/.cloudflared/config.yml service install
sudo systemctl start cloudflared
sudo systemctl status cloudflared
```

把 `<USER>` 換成你的 OS Login 用戶名稱（OS Login 帳號格式是 Gmail 帳號把 `.` 和 `@` 換成 `_`，例如 `thisismysoul_gmail_com`）。

看到 `active (running)` 就完成了。之後 VM 重開機，cloudflared 和 n8n 都會自動恢復。

---

## 步驟七：Cloudflare Access

n8n 對外之後，`butler.marsen.me` 是全公開的，任何人都能看到登入頁面。
加一層 Cloudflare Access，讓驗證擋在 n8n 前面。

### 為什麼要在 n8n 設定前先做這步？

你一開 n8n 就會看到「Set up owner account」頁面。
如果這個頁面是公開的，任何人都能搶先填、搶走 owner 帳號。

正確順序：**先設 Access → 再設 n8n owner account**。

### 操作步驟

進 Cloudflare 主控台 → 左側找 **Zero Trust**。

第一次進入需要選擇團隊名稱（之後可以改），Free 方案 50 人以下免費。

進去後選：**安全地存取私人 Web 應用程式** → **連結私人 Web 應用程式**

填入應用程式資訊：

| 欄位 | 填什麼 |
|---|---|
| 應用程式名稱 | `n8n` |
| 內部主機名稱或 IP | `127.0.0.1` |
| 通訊協定 | HTTP |
| 連接埠 | `5678` |
| 子網域 | `butler` |
| 網域 | `marsen.me` |

**為什麼 IP 填 `127.0.0.1` 而不是 GCP 外部 IP？**

因為 cloudflared 已經在 VM 裡跑了，它負責把流量從 Cloudflare 帶進來。
Access 只需要知道「cloudflared 要連哪裡」，也就是 VM 內部的 `127.0.0.1:5678`。
GCP 不用開防火牆，外部 IP 不需要暴露。

```text
使用者瀏覽器
    ↓ HTTPS
Cloudflare Access（驗證身份）
    ↓ 通過後
cloudflared tunnel（在 VM 裡跑）
    ↓ 本機連線
127.0.0.1:5678（n8n）
```

---

## 小結

- cloudflared tunnel 讓 n8n 對外，不用開 GCP 防火牆、不暴露任何 port
- Cloudflare Access 擋在前面，只有通過驗證的 email 才能進到 n8n
- cloudflared 設成 systemd service，VM 重開機自動恢復，不需要手動啟動
- 整條鏈路：瀏覽器 → Cloudflare Access（驗證）→ cloudflared tunnel → n8n

(fin)
