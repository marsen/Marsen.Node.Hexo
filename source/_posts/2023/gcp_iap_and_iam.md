---
title: " [實作筆記] GCP 使用 IAP 連線 VM"
date: 2023/07/10 16:23:19
tags:
  - 實作筆記
---

## 簡介 IAP（Identity-Aware Proxy

GCP 的 IAP（Identity-Aware Proxy）是一個身份驗證和授權服務，  
用於保護和管理對 Google Cloud 資源的訪問。  
它提供了對虛擬機器（VM）實例的安全連接和控制，並可用於 SSH、RDP 等流量的安全轉發(Forwarding)。

通過 IAP，我們可以實現零信任訪問模型，僅允許經過身份驗證和授權的用戶訪問資源，  
並且可以根據用戶的身份和上下文進行細粒度的訪問控制。  
這提高了資源的安全性並減少了潛在的安全風險。

### 零信任訪問模型

零信任訪問模型（Zero Trust Access Model）是一種安全設計方法，該方法不假設內部網絡是可信的，  
並對每個訪問請求進行驗證和授權，無論該請求來自內部還是外部網絡。  
在零信任模型中，所有訪問都需要通過驗證和授權，並且需要進一步的身份驗證和授權步驟，  
以確定用戶是否具有訪問資源的權限，更多可以查看參考資料。

### 比較 VPN 與 RDP

其他常見的連線方式包括傳統的虛擬專用網絡（VPN）和遠程桌面協議（RDP），  
它們通常用於在企業內部建立安全連接，但對於雲環境和遠程用戶來說，這些方法可能不夠靈活和安全。

IAP 提供了一個更強大和安全的連接方式。  
它在零信任訪問模型下工作，通過驗證和授權機制確保只有經過驗證的用戶可以訪問資源。  
以下是 IAP 的優勢：

- 精確的身份驗證和授權：IAP 可以根據用戶的身份和上下文進行細粒度的訪問控制，僅允許授權的用戶訪問資源。
- 無需公開 IP 地址：IAP 通過提供 IAP 隧道和代理服務，不需要公開資源的實際 IP 地址，增強了安全性。
- 雲原生和易於使用：IAP 是 GCP 的原生服務，與其他 GCP 服務整合，易於設置和管理。

## 前情提要

客戶在 GCP 上部署了多台 VM，每個 VM 都有公開的外部 IP 地址，並允許開發者使用 SSH 進行連線。  
這樣的設置存在著安全風險，也需要為這些公開的 IP 付出額外的成本。

為了提高安全性並解決這個問題，我們使用 GCP 的 IAP（Identity-Aware Proxy）。  
透過 IAP，我可以實現更安全的連線方式，不需要使用 Public IP 。  
IAP 提供了精確的身份驗證和授權，只有經過驗證的使用者才能訪問 VM。  
這種設置符合零信任訪問模型，提高了資源的安全性，同時減少了潛在的安全風險。

### 實務操作

我有一台測試用的 VM `beta` 已經拔除了 public IP,

1. 首先啟用(enable) Cloud Identity-Aware Proxy API,「Security」->「Identity-Aware Proxy」
2. 在 IAM > Permissions 找到需要登入 VM 主機的帳戶加上「IAP-secured Tunnel User」這個角色
3. 防火牆規則，IP Range: `35.235.240.0/20`, 開通指定的 Protocols and ports，比如我們要 SSH 連線，就開通 `tcp:22`，Targets 設為 `Specified targets tags` Target tags 為 `ingress-from-iap`,
4. 需要登入 VM 加上 tag `ingress-from-iap`
5. 可以用以下語法測試連線

   > ```shell
   > gcloud compute ssh beta --project=my-project --zone=asia-east1-c --troubleshoot --tunnel-through-iap
   > ```

6. 排除所有問題後嚐試連線

   > ```shell
   > gcloud compute ssh beta --project=my-project --zone=asia-east1-c --tunnel-through-iap
   > ```

### 補充說明

- 開發環境在連線時已經透過 `gcloud auth login` 取得權限了。
- 連線的目標主機仍然需要設定 SSH 金鑰
- `35.235.240.0/20` 是 GCP 中 IAP 使用的特定 IP 範圍，不可修改

#### 20230711 補充

透過 IAP 連線會出現　"Increasing the IAP TCP upload bandwidth"　的警告
可以參考[官方文件](https://cloud.google.com/iap/docs/using-tcp-forwarding#increasing_the_tcp_upload_bandwidth)

下載 Numpy

```shell
$(gcloud info --format="value(basic.python_location)") -m pip install numpy
```

設定環境變數

```shell
export CLOUDSDK_PYTHON_SITEPACKAGES=1
```

## 參考

- [IAP Concepts Overview](https://cloud.google.com/iap/docs/concepts-overview) '
- [Preparing your project for IAP TCP forwarding](https://cloud.google.com/iap/docs/using-tcp-forwarding#preparing_your_project_for_tcp_forwarding)
- [新世代資安概念：零信任安全模型介紹(Zero Trust)](https://www.webcomm.com.tw/blog/zero-trust-security-model/)
- [Zero Trust 安全性 | 什麼是 Zero Trust 網路？](https://www.cloudflare.com/zh-tw/learning/security/glossary/what-is-zero-trust/)
- [[GCP] 你還在用 VPN 連線嗎？ 快點試試 Cloud IAP，資安手法大開公 | Giving it a Try to let Cloud IAP protect your system (上)](https://joehuang-pop.github.io/2020/10/25/GCP-%E4%BD%A0%E9%82%84%E5%9C%A8%E7%94%A8VPN%E9%80%A3%E7%B7%9A%E5%97%8E%EF%BC%9F-%E5%BF%AB%E9%BB%9E%E8%A9%A6%E8%A9%A6Cloud-IAP%EF%BC%8C%E8%B3%87%E5%AE%89%E6%89%8B%E6%B3%95%E5%A4%A7%E9%96%8B%E5%85%AC-Giving-it-a-Try-to-let-Cloud-IAP-protect-your-system-%E4%B8%8A/)
- [[GCP] 網頁透過 Cloud IAP 保護 | Giving it a Try to let Cloud IAP protect your system (下)](https://joehuang-pop.github.io/2020/10/25/GCP-%E7%B6%B2%E9%A0%81%E9%80%8F%E9%81%8ECloud-IAP%E4%BF%9D%E8%AD%B7-Giving-it-a-Try-to-let-Cloud-IAP-protect-your-system-%E4%B8%8B/)

(fin)
