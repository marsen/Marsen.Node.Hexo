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

## 步驟二：設定 SSH（OS Login）

預設的 SSH key 方式要管理本機的 key 檔案，換機器或多人共用都麻煩。
OS Login 改用 Google 帳號做身份驗證，key 綁在帳號上，不依賴本機檔案。

開啟 OS Login：

```bash
gcloud compute instances add-metadata n8n-server \
  --metadata enable-oslogin=TRUE \
  --zone=us-east1-b \
  --project=YOUR_PROJECT_ID
```

之後連線就直接，不需要帶任何 key 參數：

```bash
gcloud compute ssh n8n-server --zone=us-east1-b --project=YOUR_PROJECT_ID
```

第一次連會問你要不要建立 SSH key，輸入 `y` 就好，之後不用再動。

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
sudo docker run -d \
  --name n8n \
  --restart unless-stopped \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  --user $(id -u):$(id -g) \
  -e N8N_SECURE_COOKIE=false \
  docker.n8n.io/n8nio/n8n
```

幾個參數說明：

| 參數 | 說明 |
|---|---|
| `--restart unless-stopped` | VM 重開機自動重啟 n8n |
| `-v ~/.n8n:/home/node/.n8n` | 資料持久化，不會因為容器重建而消失 |
| `--user $(id -u):$(id -g)` | 讓容器以當前用戶身份執行，避免 `~/.n8n` 出現 permission denied |
| `N8N_SECURE_COOKIE=false` | 先用 HTTP 測試，等 HTTPS 設好再拿掉 |

確認有跑起來：

```bash
sudo docker ps
sudo docker logs n8n --tail 20
```

正常的話會看到 port `0.0.0.0:5678->5678/tcp`，STATUS 是 `Up`。

log 裡可能有 community packages 的 error，是 n8n 找不到額外套件，不影響運作，忽略即可。

---

## 小結

到這裡，n8n 已經在 GCP 免費 VM 上跑起來了。

- GCP e2-micro 免費，加 swap 跑 n8n 沒問題
- OS Login 用 Google 帳號認證，不依賴本機 SSH key 檔案
- n8n image 約 2.3GB，第一次 pull 要等幾分鐘
- `--restart unless-stopped` 確保 VM 重開機後 n8n 自動恢復

目前 n8n 只能從 VM 內部存取，下一篇會用 cloudflared tunnel 讓它可以從外部用 HTTPS 開啟，不用動防火牆。

(fin)
