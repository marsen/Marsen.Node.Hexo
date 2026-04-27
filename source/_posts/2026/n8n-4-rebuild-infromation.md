---
title: "[實作筆記] 個人自動化平台(番外) 拆掉重建 GCP VM & Cloudflared Tunnel & Cloudflare Access "
date: 2026/04/18 11:29:53
tags:
  - 實作筆記
---

## 前情提要

前兩篇把 GCP 免費 VM + n8n + cloudflared tunnel + Cloudflare Access 都設好了。

但整個過程是邊做邊踩坑，步驟不一定是最乾淨的順序。
這篇把環境完整清理掉，再照前兩篇的步驟重建一次，驗證文章可以重現。

---

## 清理步驟

清理順序：從外到內，先 Cloudflare 再 GCP。

### 步驟一：刪除 Cloudflare Access Application

進 Cloudflare 主控台 → **Zero Trust** → **Access → Applications**

找到 `butler.marsen.me` 的 application，刪除。

**驗證**：開瀏覽器試 `https://butler.marsen.me`，如果 Access 已清除，不會再看到驗證頁面。

### 步驟二：刪除 Cloudflare Tunnel

SSH 進 VM，先停 cloudflared service，再刪 tunnel：

```bash
sudo systemctl stop cloudflared
cloudflared tunnel delete butler
```

**驗證**：

```bash
cloudflared tunnel list
```

`butler` 不在列表裡就是刪成功了。

> 也可以在 UI 刪：**Zero Trust → Networks → Tunnels** → 找 `butler` → 刪除。
> 但要先確認 tunnel 已停止（cloudflared service 停掉或 VM 已關機）。

### 步驟三：刪除 DNS 記錄

進 Cloudflare 主控台 → **marsen.me → DNS**

找到 `butler.marsen.me` 的 CNAME 記錄（`4ba20fc4-...cfargotunnel.com`），刪除。

**驗證**：開瀏覽器，出現 `DNS_PROBE_FINISHED_NXDOMAIN` 就是清除成功。

> DNS 記錄目前只能透過 UI 刪除，或使用 Cloudflare API（需要 API Token）。

### 步驟四：刪除 GCP VM

```bash
gcloud compute instances delete ai-butler-vm00 \
  --zone=us-east1-b \
  --project=YOUR_PROJECT_ID
```

確認後輸入 `Y`，VM 和磁碟都會一起移除。

## 參考

清理完成後，照以下兩篇文章重建

- [個人自動化平台(一) n8n & GCP VM](/2026/n8n-1-in-gcp-free-tier-vm/)
- [個人自動化平台(二) Cloudflare Access & Cloudflare Tunnel](/2026/n8n-2-cloudflared-and-tunnel/)
- [個人自動化平台(三) 收集、處理、輸出：三層可插拔管道設計](/2026/n8n-3-pipeline-overview/)
- [個人自動化平台(四) n8n 實作：收集層，RSS 資料來源](/2026/n8n-5-pipeline-fetch/)
- [個人自動化平台(五) n8n 實作：處理層，Aggregate + Gemini 週報整理](/2026/n8n-6-pipeline-process/)
- [個人自動化平台(六) n8n 實作：輸出層，Instagram API 取得 Token 全記錄](/2026/n8n-7-pipeline-output/)

## 小結

- 清理順序：從外到內，先 Cloudflare 再 GCP，避免 tunnel 刪除時出現狀態不一致
- 刪 VM 時磁碟會一起移除，n8n 資料不保留

(fin)
