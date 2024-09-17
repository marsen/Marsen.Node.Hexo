---
title: " [踩雷筆記] 移除 VM 外部 IP 後 GCS 檢查異常的問題"
date: 2023/07/12 17:57:23
tags:
  - 踩雷筆記
---

## 前情提要

我們使用 GCP Compute Engine 建立 VM，並且配置了一個對外的 IP(External IP)，  
同時設定了一個排程，定期將 VM 上產生的資料轉存到 GCS(Google Cloud Storage)，  
我們主要使用的工具是 `gcloud` 並且在 VM 上設定了某個 Service Account，  
並且一切運作順利。

某日因為資安考量我們移除了 VM 的 External IP，而搬檔到 GSC 的排程就異常了。  
我們簡單使用以下指令作檢查

```shell
gcloud storage ls
```

這個指令會列出我們 GCS 上的資源，  
當 VM 有 External IP 時一切運作正常，而移除時會無回應。

## 解決方法

這裡只需要修改 GCP 的 VPC 設定，  
選擇 subnet (ex:asia-east1，看你的 VM 在哪個 Zone)  
啟用 Private Google access 為 On 然後存檔

## 參考

- [How to read from a Storage bucket from a GCE VM with no External IP?](https://stackoverflow.com/questions/54984713/how-to-read-from-a-storage-bucket-from-a-gce-vm-with-no-external-ip)

(fin)
