---
title: " [實作筆記] 踩雷 GCP Instance Schedules 排程開關機"
date: 2025/02/24 12:01:41
tags:
  - 學習筆記
---

## 前情提要

因為周末機器閒置很浪費，想透過排程來開關機達到節省成本的效果。  
以為設定排程開關機這麼簡單，結果在操作過程中，遇到了一些讓我差點摔機的問題。  
這篇就來記錄一下，從設定到踩雷，再到如何解決的過程。

### 幾個作法

上網找了一些作法，得到以下的建議  
最簡單的方法：使用 Instance Schedules  
較靈活的方法：使用 Cloud Scheduler + Cloud Functions  
基礎架構管理：使用 Terraform  

我沒有複雜的情境，當然選最簡單的
只要在 GCP Console 上面操作即可。

### 遇到問題

> Compute Engine System service account <service-YOUR-PROJECT-ID@compute-system.iam.gserviceaccount.com>
> needs to have [compute.instances.start,compute.instances.stop] permissions applied in order to perform this operation.  

而且 Console IAM 裡面看不到這個帳號 `service-YOUR-PROJECT-ID@compute-system.iam.gserviceaccount.com`
類似的帳號有`YOUR-PROJECT-ID-compute@developer.gserviceaccount.com`但兩者並不相同

實際上是 GCP 會有一群隱藏帳號  
清單如下：
📌 Compute Engine → <service-YOUR_PROJECT_NUMBER@compute-system.iam.gserviceaccount.com>
📌 App Engine → <YOUR_PROJECT_ID@appspot.gserviceaccount.com>
📌 Cloud Run / Functions → <YOUR_PROJECT_ID-compute@developer.gserviceaccount.com>
📌 GKE → <service-YOUR_PROJECT_NUMBER@container-engine-robot.iam.gserviceaccount.com>
📌 Cloud Build → <service-YOUR_PROJECT_NUMBER@gcp-sa-cloudbuild.iam.gserviceaccount.com>
📌 BigQuery → <service-YOUR_PROJECT_NUMBER@gcp-sa-bigquerydatatransfer.iam.gserviceaccount.com>
📌 Firestore → <service-YOUR_PROJECT_NUMBER@gcp-sa-firestore.iam.gserviceaccount.com>

🔍 查詢方式：

```sh
gcloud projects get-iam-policy YOUR_PROJECT_ID --format=json | jq -r '.bindings[].members[]' | grep 'serviceAccount:'
```

解法，加權限就好，在 Console 直接加  
但是在 Console 還是看不到帳號, IAM/Service Account/Roles 都看不到

不過我們可以用 CLI 查詢方式

#### 先確認目前權限

```sh
gcloud projects get-iam-policy YOUR_PROJECT_ID --format=json | jq '.bindings[] | select(.members[] | contains("serviceAccount:ACCOUNT_TO_REMOVE"))'
```

#### 移除指定角色

```sh
gcloud projects remove-iam-policy-binding YOUR_PROJECT_ID \
  --member="serviceAccount:ACCOUNT_TO_REMOVE" \
  --role="roles/要移除的角色"
```

⚠️ 需要確保該角色正確，否則可能會影響服務運行。

## 小結

原本只是想透過排程來節省成本，結果遇到 GCP 隱藏帳號的問題，導致權限無法正常配置。  
最後透過 CLI 查詢，手動補上必要權限，才讓 Instance Schedules 順利運作。

### 這次知道的事

Console 不一定能看到所有帳號，CLI 查詢更可靠。  
GCP 預設有許多隱藏帳號，遇到權限問題時，先確認 IAM policy。  
權限管理應該精確設定，避免不必要的風險。  
遇到類似問題時，先從 gcloud projects get-iam-policy 下手，能少走不少彎路。

(fin)
