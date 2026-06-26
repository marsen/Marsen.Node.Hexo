---
title: "[實作筆記] 個人自動化平台(一) n8n & GCP VM"
date: 2026/04/17 11:37:12
tags:
  - 實作筆記
---

## 前情提要

最近想整合 AI 與自已的職能，建立一些小工具。

第一個想法是資料搜集器，可以自由擴充的個人內容自動化平台。

## 核心概念

「捕捉資料」、「整理資料」、「發送報告」拆成三種抽象概念元件，

隨時可以組合或替換。

```text
Sources（捕捉資料）  →  Processors（整理）  →  Sinks（發送）
─────────────────     ─────────────────      ─────────────
AI 新聞               摘要整理                LINE
RSS 訂閱              翻譯                    Email
YouTube               事實查核                Instagram
Gmail                 格式化                  GitHub
...                   ...                     ...
```

一條 pipeline = 選幾個 Source + Processor + Sink 組合起來。
加新管道只需要加一個 Sink 節點，不動其他邏輯。

## 技術選擇與環境規格

選 n8n 當 runtime，因為它本來就是這個思路設計的，

而且可以自己架，不用把資料交給第三方。

也有考慮過其他方案：

| 方案 | 優點 | 缺點 |
| --- | --- | --- |
| CCR（Claude Code Remote，Anthropic 雲端排程 agent） | 免維護 | 無法自訂節點、靈活度低 |
| VM + cron | 自由度高 | 要自己維護腳本 |
| n8n | 視覺化、可擴充、節點生態豐富、私心想學 | 需要維護 VM |

考慮控制力與學習新技能，最後選 n8n。

一、跑在 GCP e2-micro，always free，完全不用花錢。

GCP Always Free 的條件：

- 機器：`e2-micro`（2 vCPU 共享、1 GB RAM）
- 區域：美國區（us-east1 / us-west1 / us-central1）
- 磁碟：30 GB 標準永久磁碟
- 網路：每月 1 GB 對外流量

### 關於 IP 費用

GCP 的外部 IP 分兩種：

| 類型 | 說明 | 費用 |
| --- | --- | --- |
| 臨時外部 IP | VM 運行時自動分配，刪除後釋放 | 免費 |
| 靜態外部 IP（附在 VM 上） | 固定 IP，VM 刪了還保留 | 免費 |
| 靜態外部 IP（未附在 VM） | 保留但沒有用 | 收費 |

我們用臨時 IP，免費。

### 關於流量費用

GCP 計算的是 VM 的對外流量（VM → 外部）。
後面會用 cloudflared tunnel，流量路徑是 VM → Cloudflare → 使用者，
GCP 端的出站流量極小，個人 n8n 使用幾乎不會超過每月 1GB 的免費額度。

個人用的 n8n 排程任務，這個規格完全夠。

二、在 GCP 的 VM 上安裝 n8n(文章待補)

三、在 n8n 上設定 pipeline(文章待補)

---

## 步驟一：建立 VM

以下我直接讓 AI 執行，你也可以手動或是使用瀏覽器的圖形介面操作

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

我個人比較喜歡　OS Login 改用 Google 帳號做身份驗證，key 綁在帳號上，不依賴本機檔案。

實際上還是有 ssh key 只是不用特別在意它。

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

n8n 容器裡的 `node` 用戶 UID 是 1000。

先把 volume 目錄的 owner 設成 1000，避免容器啟動後出現 permission denied：

port 使用預設的 `5678`。後面會用 cloudflared tunnel 轉發，這個 port 不會對外暴露，改不改都一樣。

```bash
mkdir -p ~/.n8n
sudo chown 1000:1000 ~/.n8n
sudo docker run -d \
  --name n8n \
  --restart unless-stopped \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  -e N8N_SECURE_COOKIE=false \
  docker.n8n.io/n8nio/n8n
```

幾個參數說明：

| 參數 | 說明 |
| --- | --- |
| `--restart unless-stopped` | VM 重開機自動重啟 n8n |
| `-v ~/.n8n:/home/node/.n8n` | 資料持久化，不會因為容器重建而消失 |
| `N8N_SECURE_COOKIE=false` | n8n 預設要求 HTTPS 才設定 cookie。對外雖然是 HTTPS，但 cloudflared 到 n8n 的內部連線是 HTTP，所以這個設定要保留 |

n8n 映像檔本來就用非 root 的 `node` 用戶執行，不需要加 `--user`。

確認有跑起來：

```bash
sudo docker ps
sudo docker logs n8n --tail 20
```

正常的話會看到 port `0.0.0.0:5678->5678/tcp`，STATUS 是 `Up`。

log 裡可能有 community packages 的 error，是 n8n 找不到額外套件，不影響運作，忽略即可。

---

## 參考

- [個人自動化平台(二) Cloudflare Access & Cloudflare Tunnel](/2026/04/17/2026/n8n-2-cloudflared-and-tunnel/)
- [個人自動化平台(三) 收集、處理、輸出：三層可插拔管道設計](/2026/04/20/2026/n8n-3-pipeline-overview/)
- [個人自動化平台(番外) 拆掉重建 GCP VM & Cloudflared Tunnel & Cloudflare Access](/2026/04/18/2026/n8n-3.1-rebuild-infromation/)
- [個人自動化平台(四) n8n 實作：收集層，RSS 資料來源](/2026/04/20/2026/n8n-4-pipeline-fetch/)
- [個人自動化平台(五) n8n 實作：處理層，Aggregate + Gemini 週報整理](/2026/04/27/2026/n8n-5-pipeline-process/)
- [個人自動化平台(六) n8n 實作：輸出層，Instagram API 取得 Token 全記錄](/2026/04/22/2026/n8n-6-pipeline-output/)
- [個人自動化平台(七) n8n 備份自動化：workflow + credentials → GitHub](/2026/04/29/2026/n8n-7-backup-automation/)

## 小結

到這裡，n8n 已經在 GCP 免費 VM 上跑起來了。

- GCP e2-micro 免費，加 swap 跑 n8n 沒問題
- OS Login 用 Google 帳號認證，不依賴本機 SSH key 檔案
- n8n image 約 2.3GB，第一次 pull 要等幾分鐘
- `--restart unless-stopped` 確保 VM 重開機後 n8n 自動恢復

目前 n8n 只能從 VM 內部存取，下一篇會用 cloudflared tunnel 讓它可以從外部用 HTTPS 開啟，不用動防火牆。

(fin)
