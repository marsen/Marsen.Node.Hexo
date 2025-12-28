---
title: " [實作筆記] GCP Cloud Run 私有化部署：透過 Load Balancer 實現內網存取與 IP 限制"
date: 2025/06/04 11:42:01
tags:
  - 學習筆記
---
## 前情提要

我用 cloud run 建立一個 api 並且有 webhook 的功能
並且希望提供一個對外網址給客戶，而不是 `*.run.app` 結尾的網址。  

在某些場景下，我們不希望 GCP Cloud Run 對外暴露，

例如內部 API Gateway 呼叫、CI/CD 系統內部流程、或僅提供給 VPC 內特定服務存取的微服務。

這次的例子是希望加上 IP 的防護，只允許特定的 IP 呼叫，這時候就會希望：

- Cloud Run 不對外公開（Private）

- 僅允許 內網 IP 或 Internal Load Balancer 存取

看似簡單，但 GCP Cloud Run 原生是無伺服器架構，預設就是「公開網址」，要讓它只對內部可見，需要搭配一些 GCP 網路元件操作。這篇筆記會帶你逐步設定，並避開一些常見坑洞。

## 實作記錄

1. 將 Cloud Run 設為「不公開」
    預設 Cloud Run 是公開的，要改為「只有授權的主體可以存取」：

    ```bash
    gcloud run services update YOUR_SERVICE \
      --ingress internal-and-cloud-load-balancing \
      --allow-unauthenticated \
      --region YOUR_REGION \
      --project YOUR_PROJECT

    ```

    這樣只有內部的網路才能呼叫 Cloud Run。

    也可以透過 GUI 設定，選擇你的 Cloud Run >  

    Networking > Ingress 選 internal 勾選 Allow Traffic from external Application Load Balancers

    可以[參考](https://cloud.google.com/run/docs/securing/ingress#gcloud)

2. 建立 Serverless NEG（Network Endpoint Group）
   可以用 GUI 建立 Loading Blancer >  

   Edit > Backend configuration >  

   下拉選單選 Create backend service >  

   Backend Type 選 Serverless network endpoint group  

   *因為是 Cloud Run*

3. 設定 Loading Blancer 的 Backend Service"
   LB > Edit backend service

   Regions 選擇 Cloud Run 所在的 Region

   預設會有一組 Security Policy

   Cloud Armor policies > 找到這組 Policy 可以作更細緻的設定，Ex:指定 Ip 呼叫

4. Loading Blancer URL Map
   這是一個額外的設定，情境是原本我已有一組 Domain 與 Backend Service  

   並且已經設定在 LB 上面，在想要共用 Domain 的情況下，我需要設定 URL Map

   保留一組 Path 讓流量打向 Cloud Run，但仍保留其他 API 可以用

   LB > Edit > Routing rules > Mode 選擇 Advanced host and path rule  

   找到指定的 Domain，設定參考如下

   ```yml
   defaultService: projects/my-gcp-proj/global/backendServices/my-api
   name: path-matcher-7
   pathRules:
   - paths:
     - /api/v1/*
       service: projects/my-gcp-proj/global/backendServices/my-api
   - paths:
     - /cloud-run-app/*
       service: projects/my-gcp-proj/global/backendServices/my-cloud-run-app
       routeAction:
         urlRewrite:
           pathPrefixRewrite: /
   ```

    以上設定簡單說明如下，

    - /api/v1/*：導向 my-api，保留原始路徑。
    - /cloud-run-app/*：導向 my-cloud-run-app，並將路徑重寫為去除 /cloud-run-app 前綴。
    - 其他路徑：預設導向 my-api。

    這樣設定可讓多個服務共用同一個負載平衡器並支援乾淨的 URL 管理。

## 參考

- [好用測試 Callback 的工具](https://webhook.site)
- [Restrict network ingress for Cloud Run](https://cloud.google.com/run/docs/securing/ingress#gcloud)

(fin)
