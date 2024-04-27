---
title: " [實作筆記] Gitlab CI/CD 與 GCP - Cloud Run 方案選擇過程記錄"
date: 2024/04/17 10:18:25
tags:
  - 實作筆記
  - CI/CD
---

## 前情提介

在我的實務經驗上，很常與新創團隊合作，而架構選擇常常是第一個問題，  
最常見的解法是，由技術負責人選擇最熟悉的技術…  
這樣真的好嗎?  
比如說，T 社早期由某群工程人員開發，  
選擇了 Go、Nodejs、C# 等多語言與雲端 GKE(K8S) 作為開發架構，  
事後不負責任離職後，對持續的運作與維護都造成了困難  
這件事對我的警鐘是 --- **作為管理者，如果不具備識別專業的能力，就只能透過信賴某人或團隊;這是非常脆弱且危險的。**

### 執行環境的選擇

作為 Web 開發者，最常用的執行環境由簡入繁如下:

#### 地端到雲端到

- 開發機就是正式機: 通常只是用來展示，開發者最常這樣作
- 地端的主機: 沒有技術能力公司，很有可能都是這樣的架構，直接向開發商買主機放機房或是由開發商部署
- 地端的虛擬機(VM): 會用較高規格的機器部署多台 VM，可以更有效使用資源
- 雲端的虛擬機(VM): 使用上與上面差不多，但是由大廠來維持高可用性，學習門檻低
- 雲端的 Server Less: 需要熟悉不同雲的相關產品，算是有點門檻，完成後可以精準控制預算、減少人力成本、透過雲維持高可用性
- 雲端的 K8s: 門檻較上面更高，也有相同的優點，但是會有更高的可控性，適合有一定使用規模後，需要細部彈性調整資源的公司。
- 再回地端: 如果規模再繼續成長，雲端的成本變得高不可攀時，或控制力下降，在有一定的技術水準情況下，有的公司會選擇回到地端。[參考 37signals 的例子](https://37signals.com/podcast/leaving-the-cloud/)  

## GCP Cloud Run

回歸本文，我選擇了 GCP Cloud Run，這是一種雲端的 Server Less 的解決方案。  
產品的規模沒有大到需要 K8s，團隊成員的具備足夠的能力，  
類似的方案還有 Cloud Function，
作為一個 Web API Base 的輕量專案，我評估 Cloud Run 更適合
Cloud Function 較適合 Event Driven 的片段行為
Cloud Run 較適合有點複雜度，但是可以容器化的應用。

### 相關作業

首先準備好我的程式，這是一個透過 Azure 寄信程式，需要在 `Azure Registered APP` 設定，  
基本的讀取組態、寫 Log 到雲端與錯誤處理(Error Handle)、相依注入與測試等…  
其中一個功能需要將 token 存儲在 `GCS(gcloud storage)` 裡，  
並透過 `Cloud Scheduler` 定期更新，所以需要為其提供必要的權限與帳號(GCP Service Account).  

使用 `Cloud Run` 前，`Docker` 也是必要的前置知識，  
你會需要撰寫 Dockerfile ，並且需要在 GCP 上建立一個 `Artifacts Registry`  
如此一來，就可以透過 `Cloud Build` 建立並部署 `Cloud Run` 了  
用　`GCP Workload Identity Federation` 管理 `Service Account`  
管理的對像有 2 個，CI 的 Service Account 與執行程式的 Cloud Run 的 Service Account.  

後續有考慮透過 Cloud Run 提供 `Swagger` 之類的 API 文檔，但現階段先共享 `Postman` 資訊處理。

#### 20240427 更新

我的 CI 腳本如下  

```yaml
export CLOUDSDK_AUTH_ACCESS_TOKEN={provider by workload identify federation}
gcloud builds submit --tag your-registry/your-app:latest .
gcloud run services update your-app --image=your-registry/your-app:latest 
```

我的 Gitlab Runner 是透過 GCP 上 A Project 的 VM 建立並註冊的，  
而我的 Cloud Run 需要部署並運作在 GCP B Project 上，  
預設執行的身份會是 AProj 的 Compute Engine default service account，以下簡稱 A_CE_Account，  
而 `CLOUDSDK_AUTH_ACCESS_TOKEN` 代表的是 B Project 的 Gitlab Runner Service Account，以下簡稱 B_GR_Service_Account，  
另外在運作 B Project 的 Cloud Run 需要的是 B Project 的 Cloud Run Service Account，以下簡稱　B_CR_Service_Account，  

Gitlab Runner 在執行時，執行 `gcloud auth list` 會顯示 `A_CE_Account`，  
但是由於有設定 `CLOUDSDK_AUTH_ACCESS_TOKEN`，所以相關命令的執行身份是　`B_GR_Service_Account`，  
這會有足夠的權限執行　`gcloud builds submit --tag your-registry/your-app:latest .`  
但是沒有權限執行　`gcloud run services update your-app --image=your-registry/your-app:latest`  
原因是 Cloud Run 的執行身份是 `B_GR_Service_Account`，  
為此，我們需要賦予 `B_GR_Service_Account` 角色 `Service Account User`

## 參考關鍵字

- [Cloud Run](https://cloud.google.com/run?hl=en)
- [Cloud Function](https://cloud.google.com/functions)
- [Cloud Build](https://cloud.google.com/build)
- [Cloud Storage](https://cloud.google.com/storage)
- [Cloud Scheduler](https://cloud.google.com/scheduler)
- [Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation)
- [Docker](https://www.docker.com/)
- [Azure Registered APP](https://learn.microsoft.com/en-us/security/zero-trust/develop/app-registration)

## 參考

- [實作筆記] Gitlab CI/CD 與 GCP 相關文章
  - [架構全貌](https://blog.marsen.me/2023/04/13/2023/gitlab_ci_and_gcp_vm/)
  - [建立 Web Server VM](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_create_server/)
  - [建立 Gitlab Runner VM](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_cretae_runner/)
  - [防火牆設定](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_firewall/)
  - [Linux User 與資料夾權限](https://blog.marsen.me/2023/04/24/2023/gitlab_ci_and_gcp_vm_account/)
  - [機敏資料的處理](https://blog.marsen.me/2023/05/29/2023/gitlab_ci_and_gcp_vm_secret_config/)
  - [錯誤處理](https://blog.marsen.me/2023/11/16/2023/gitlab_ci_error_handle/)
  - [Workload Identity Federation](https://blog.marsen.me/2024/03/13/2024/gitlab_ci_and_gcp_workload_federation/)
  - [Cloud Run](https://blog.marsen.me/2024/04/17/2024/gitlab_ci_and_gcp_cloud_run/)

(fin)
