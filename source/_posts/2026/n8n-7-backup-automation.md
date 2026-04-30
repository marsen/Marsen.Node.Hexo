---
title: "[實作筆記] 個人自動化平台(七) n8n 備份自動化：workflow + credentials → GitHub"
date: 2026/04/30 03:08:11
tags:
  - 實作筆記
---

## 前情提要

n8n 跑起來、workflow 也在跑了，下一個問題：掛掉怎麼辦？

VM 重開機沒問題，但容器砍掉重建、或換 VM，workflow 和 credentials 就都要重設。
這篇記錄怎麼用 CLI export + git 做自動備份。

---

## 備份方案決策

考慮過幾個方向：

| 方案 | 問題 |
|---|---|
| 備份整個 `.n8n` 目錄 | SQLite 每次執行都寫入，幾乎天天變動，太肥 |
| 只備份 workflow JSON | 沒有 credentials 無法完整還原 |
| n8n Source Control（GitOps） | Enterprise/Business 付費功能，Community 版不支援 |
| CLI export workflow + credentials | 可行，輕量，Community 版都有 |

最終選 **CLI export + GitHub private repo**，搭配 cron job 每天自動跑。

---

## 架構

```text
GCP VM（cron job 每天凌晨 3 點）
  └─ docker exec n8n → export workflow + credentials JSON
       └─ bind mount（/home/user/.n8n）
            └─ 複製到 repo backup/YYYYMMDD-NNN/ → git commit + push → GitHub
```

每次跑都建立新的日期資料夾（`20260430-001`、`20260430-002`），保留最近 30 份，舊的自動刪除。就算刪了，git log 還是查得到歷史。

---

## 實作

### Repo 結構

```text
Marsen.Butler.n8n.Backup/
├── README.md
├── install.sh    ← 一次性安裝，設定 cron job
├── backup.sh     ← 每次執行的備份邏輯
└── backup/
    └── 20260430-001/
        ├── workflows/
        └── credentials/
```

### backup.sh 核心邏輯

```bash
# container name 預設 n8n，可用環境變數覆蓋
N8N_CONTAINER="${N8N_CONTAINER:-n8n}"
KEEP_COUNT="${KEEP_COUNT:-30}"

# 產生日期流水號資料夾
DATE_TAG="$(date +%Y%m%d)"
SERIAL=$(printf "%03d" $(( $(ls -d backup/${DATE_TAG}-* 2>/dev/null | wc -l) + 1 )))
BACKUP_DIR="backup/${DATE_TAG}-${SERIAL}"

# 確認容器在跑，export，複製到 repo...

# 刪除舊備份（保留最近 KEEP_COUNT 份）
TOTAL=$(ls -d backup/[0-9]* 2>/dev/null | wc -l)
if [ "$TOTAL" -gt "$KEEP_COUNT" ]; then
  ls -d backup/[0-9]* | sort | head -n $(( TOTAL - KEEP_COUNT )) | xargs rm -rf
fi

# 每次都 commit
git add backup/
git commit -m "backup ${DATE_TAG}-${SERIAL}"
git push
```

`docker inspect` 自動找 bind mount 路徑，不寫死，換 VM 也通用。

### install.sh

```bash
# 設定每天凌晨 3 點執行
CRON_JOB="0 3 * * * ${BACKUP_SCRIPT} >> /var/log/n8n-backup.log 2>&1"

# 避免重複寫入
if crontab -l 2>/dev/null | grep -q "$BACKUP_SCRIPT"; then
  echo "Cron job already exists, skipping."
else
  (crontab -l 2>/dev/null; echo "$CRON_JOB") | crontab -
fi
```

---

## 踩坑記錄

### git clone 下來的腳本沒有執行權限

`git pull` 之後直接跑 `./backup.sh`，結果：

```text
-bash: ./backup.sh: Permission denied
```

原因：git 預設不保留 `chmod +x`，clone 下來的檔案是 644。

解法：用 `git update-index` 把執行權限記進 git：

```bash
git update-index --chmod=+x backup.sh install.sh
git commit -m "fix: preserve execute permission for scripts in git"
git push
```

之後 `git pull` 拿到的檔案就自帶執行權限，不需要手動 `chmod`。

### Container name 不應該硬綁定

一開始寫死 `N8N_CONTAINER="n8n"`，但我的容器叫 `n8n-01`，直接報錯。

改為環境變數覆蓋，預設值 `n8n`：

```bash
N8N_CONTAINER="${N8N_CONTAINER:-n8n}"
```

臨時要換：`N8N_CONTAINER=n8n-01 ./backup.sh`，不需要改腳本。

---

## VM 上安裝步驟

```bash
# 1. clone repo
git clone git@github.com:marsen/Marsen.Butler.n8n.Backup.git ~/n8n-backup

# 2. 安裝 cron job
cd ~/n8n-backup
./install.sh

# 3. 手動跑一次確認
./backup.sh
```

---

## 還原

### 情境一：同 VM，換新容器

```bash
docker run -d \
  --name n8n \
  -e N8N_SECURE_COOKIE=false \
  -v /home/user/.n8n:/home/node/.n8n \
  -p 5678:5678 \
  --restart unless-stopped \
  docker.n8n.io/n8nio/n8n

# 取最新備份
LATEST=$(ls -d ~/n8n-backup/backup/[0-9]* | sort | tail -1)
cp -r "${LATEST}/workflows/." /home/user/.n8n/exports/workflows/
cp -r "${LATEST}/credentials/." /home/user/.n8n/exports/credentials/
docker exec n8n n8n import:workflow --separate --input=/home/node/.n8n/exports/workflows/
docker exec n8n n8n import:credentials --separate --input=/home/node/.n8n/exports/credentials/
```

### 情境二：新 VM

從密碼管理器取出 `N8N_ENCRYPTION_KEY`，啟動時帶入：

```bash
docker run -d \
  --name n8n \
  -e N8N_ENCRYPTION_KEY=你的key \
  -e N8N_SECURE_COOKIE=false \
  -v /home/user/.n8n:/home/node/.n8n \
  -p 5678:5678 \
  --restart unless-stopped \
  docker.n8n.io/n8nio/n8n
```

clone repo 並還原最新備份：

```bash
git clone git@github.com:marsen/Marsen.Butler.n8n.Backup.git ~/n8n-backup
LATEST=$(ls -d ~/n8n-backup/backup/[0-9]* | sort | tail -1)
cp -r "${LATEST}/workflows/." /home/user/.n8n/exports/workflows/
cp -r "${LATEST}/credentials/." /home/user/.n8n/exports/credentials/
docker exec n8n n8n import:workflow --separate --input=/home/node/.n8n/exports/workflows/
docker exec n8n n8n import:credentials --separate --input=/home/node/.n8n/exports/credentials/
```

> Credentials 是用 `N8N_ENCRYPTION_KEY` 加密儲存，還原時必須用相同的 Key，否則全部無法解密。Key 存密碼管理器，不存 repo。

---

## 參考

- [個人自動化平台(一) n8n & GCP VM](/2026/n8n-1-in-gcp-free-tier-vm/)
- [個人自動化平台(二) Cloudflare Access & Cloudflare Tunnel](/2026/n8n-2-cloudflared-and-tunnel/)
- [個人自動化平台(三) 收集、處理、輸出：三層可插拔管道設計](/2026/n8n-3-pipeline-overview/)
- [個人自動化平台(番外) 拆掉重建 GCP VM & Cloudflared Tunnel & Cloudflare Access](/2026/n8n-3.1-rebuild-infromation/)
- [個人自動化平台(四) n8n 實作：收集層，RSS 資料來源](/2026/n8n-4-pipeline-fetch/)
- [個人自動化平台(五) n8n 實作：處理層，Aggregate + Gemini 週報整理](/2026/n8n-5-pipeline-process/)
- [個人自動化平台(六) n8n 實作：輸出層，Instagram API 取得 Token 全記錄](/2026/n8n-6-pipeline-output/)
- [個人自動化平台(番外) Google OAuth redirect_uri_mismatch 除錯記錄](/2026/n8n-6.1-google-oauth-redirect-uri-debug/)

## 小結

- CLI export 是 Community 版可用的最輕量備份方案
- 每次建日期資料夾（`YYYYMMDD-NNN`），保留最近 30 份，超過自動 prune
- 就算資料夾被刪，git log 還是查得到歷史
- `git update-index --chmod=+x` 讓腳本執行權限跟著 repo 走
- `N8N_CONTAINER` 環境變數讓腳本不寫死 container name
- 還原關鍵：`N8N_ENCRYPTION_KEY` 要另外存好

(fin)
