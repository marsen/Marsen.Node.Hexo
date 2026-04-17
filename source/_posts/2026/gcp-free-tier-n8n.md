---
title: "[實作筆記] 用 GCP 免費 VM 跑 n8n，打造個人自動化平台"
date: 2026/04/17 11:37:12
tags:
  - 實作筆記
---

## 前情提要

我想建一個可以自由擴充的個人內容自動化平台，
核心概念是把「抓資料」、「處理內容」、「發送到哪裡」拆成三種元件，
隨時可以組合或替換。

選 n8n 當 runtime，因為它本來就是這個思路設計的，
而且可以自己架，不用把資料交給第三方。

跑在 GCP e2-micro，always free，完全不用花錢。

---

## 架構概念

```text
Sources（抓資料）  →  Processors（處理）  →  Sinks（發送）
─────────────────     ─────────────────      ─────────────
AI 新聞               摘要整理                LINE
RSS 訂閱              翻譯                    Email
YouTube               事實查核                Instagram
Gmail                 格式化                  GitHub
...                   ...                     ...
```

一條 pipeline = 選幾個 Source + Processor + Sink 組合起來。
加新管道只需要加一個 Sink 節點，不動其他邏輯。

---

## 環境規格

GCP Always Free 的條件：

- 機器：`e2-micro`（2 vCPU 共享、1 GB RAM）
- 區域：美國區（us-east1 / us-west1 / us-central1）
- 磁碟：30 GB 標準永久磁碟
- 網路：每月 1 GB 對外流量

個人用的 n8n 排程任務，這個規格完全夠。

---

## 步驟一：建立 VM

先裝 gcloud CLI：

```bash
brew install google-cloud-sdk
```

登入並設定 project：

```bash
gcloud auth login
gcloud config set project YOUR_PROJECT_ID
```

建立 VM：

```bash
gcloud compute instances create n8n-server \
  --zone=us-east1-b \
  --machine-type=e2-micro \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --boot-disk-size=30GB \
  --boot-disk-type=pd-standard \
  --tags=http-server,https-server
```

---

## 步驟二：設定 SSH

gcloud 預設會把 SSH key 存在 `~/.ssh/`，但如果你的 home 在 Dropbox 這類同步資料夾裡，
key 可能會壞掉。解法是開 OS Login，用 Google 帳號認證，不依賴 key：

```bash
gcloud compute instances add-metadata n8n-server \
  --metadata enable-oslogin=TRUE \
  --zone=us-east1-b
```

之後連線就直接：

```bash
gcloud compute ssh n8n-server --zone=us-east1-b
```

---

## 步驟三：設定 Swap

e2-micro 只有 1 GB RAM，n8n 本身就吃掉一半左右。
加 swap 避免 OOM crash：

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

最後一行讓 swap 重開機後也自動生效。

---

## 步驟四：安裝 Docker

```bash
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
```

`usermod` 讓你的帳號不用 `sudo` 就能跑 docker，要重新登入才生效。

---

## 步驟五：跑 n8n

```bash
mkdir -p ~/.n8n
docker run -d \
  --name n8n \
  --restart unless-stopped \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  -e N8N_SECURE_COOKIE=false \
  docker.n8n.io/n8nio/n8n
```

幾個參數說明：

| 參數 | 說明 |
|---|---|
| `--restart unless-stopped` | VM 重開機自動重啟 n8n |
| `-v ~/.n8n:/home/node/.n8n` | 資料持久化，不會因為容器重建而消失 |
| `N8N_SECURE_COOKIE=false` | 先用 HTTP 測試，等 HTTPS 設好再拿掉 |

確認有跑起來：

```bash
docker ps
docker logs n8n --tail 20
```

---

## 步驟六：開 Firewall（待補）

目前 n8n 跑在 port 5678，還需要：

- GCP 防火牆規則開 5678
- 或用 cloudflared tunnel 對外，不直接暴露 port

cloudflared 的好處是不用改防火牆，直接用 Cloudflare 的 CDN 代理，
還能順便拿到 HTTPS。這部分後續文章再補。

---

## 小結

- GCP e2-micro 免費，加 swap 跑 n8n 沒問題
- OS Login 比 SSH key 好管，不依賴本機檔案
- n8n 的 Source/Processor/Sink 模型天然適合個人自動化 pipeline
- cloudflared tunnel 是下一步，解決對外存取問題

(fin)
