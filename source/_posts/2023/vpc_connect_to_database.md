---
title: " [實作筆記] QA DB 的連線方案決擇"
date: 2023/04/06 17:29:40
tags:
  - 實作筆記
  - GCP
---

## 前情提要

QA 環境的 GCP Cloud SQL 沒有開 public IP。  
不開 public IP 有兩個好處：資安性提高、省錢（GCP 的 public IP 要額外收費）。  
但這樣一來要怎麼從本機連到資料庫？研究了一下，有三個方向。

## 三個方案比較

### 1. Cloud SQL Auth Proxy

GCP 官方工具，在本機起一個 proxy process，讓應用程式連 `127.0.0.1` 即可。

優點：不需要 VPN 或 SSH，安全性高，官方維護。  
缺點：需要安裝設定，而且有個陷阱——

**只有 private IP 的 instance，啟動時必須加 `--private-ip` flag：**

```bash
./cloud-sql-proxy --private-ip INSTANCE_CONNECTION_NAME
```

沒加這個 flag 的話，預設會嘗試用 public IP 連，然後噴這個錯誤：

```text
Failed to connect to instance: Config error: instance does not have IP of type "PUBLIC"
```

另外，Auth Proxy 的機器本身必須在同一個 VPC 內才能透過 private IP 連到 Cloud SQL。

### 2. VPN 連線 VPC

在 GCP 上建立 Cloud VPN，讓本機透過 VPN 加入 VPC 網路，直接連 private IP。

優點：連線後就像在同一個內網，所有服務都能存取。  
缺點：設定成本高，需要維護 VPN gateway，適合團隊長期使用，對單一 QA 環境來說有點殺雞用牛刀。

### 3. SSH Tunnel

透過一台在 VPC 內的 VM（跳板機）建立 SSH tunnel，把本機的 port 轉到 Cloud SQL 的 private IP。

優點：不需要在 GCP 上另外設定網路，只要有一台跳板機就好。  
缺點：需要管理 SSH key，每個開發者都要自己設定，高流量下有效能瓶頸。

## 實作：DataGrip + SSH Tunnel

我選了 SSH Tunnel，用 DataGrip 設定最方便。

**步驟一**：開 DataGrip → 左上角「+」→「Data Source」→ 選 MySQL（或你的 DB 類型）。

**步驟二**：填寫 DB 連線資訊。Host 填 Cloud SQL 的 **private IP**，Port、User、Password 照填。

**步驟三**：切到「SSH/SSL」頁籤 → 勾選「Use SSH tunnel」→ 點「Add SSH configuration」：

- Host：跳板機的 IP 或 hostname
- Port：22
- User name：SSH 登入帳號
- Auth type：選 Key pair
- Private key file：本機的 SSH private key 路徑

注意：這裡填的 Host 是**跳板機**，不是 Cloud SQL。

**步驟四**：按「Test Connection」確認連線成功，儲存設定。

## 小結

三個方案各有適用場景：

- **Auth Proxy**：適合 CI/CD 或應用程式連線，記得加 `--private-ip`
- **VPN**：適合整個團隊需要長期存取多個 GCP 資源
- **SSH Tunnel**：適合個人開發者臨時連 QA DB，DataGrip 設定簡單直覺

## 參考

- [Cloud SQL Auth Proxy 官方文件](https://cloud.google.com/sql/docs/mysql/connect-auth-proxy)
- [DataGrip SSH Tunnel 設定文件](https://www.jetbrains.com/help/datagrip/connect-to-a-database-with-ssh.html)

(fin)
