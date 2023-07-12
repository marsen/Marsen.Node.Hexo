---
title: " [實作筆記] 靜態網站部署整合 GCP (二) --- Load Balancing & Cloud Storage"
date: 2022/03/09 10:53:48
tags:
  - 實作筆記
---

## 前請提要

去年有記錄一篇有關 GCP 部署靜態網站時遇到情境與問題，  
不過比較偏像流水帳，不同的部署策略都在同一篇文章，另外有一些問題也得到解決，  
除了更新原有文章外，這篇的目的是為了將 Load Balancing 與 Cloud Storage 抽出，  
並加以潤飾。

## 概要

- 在 Cloud Storage 建立一個 Bucket
- 在 Network Service 建立一個 Load Balancing
- 設定 Load Balancing 的 Backend 綁定到 Bucket
- 作為網站，需要允許所有人讀取 Bucket 的內容

## 必要條件

GCP 平台的帳戶與足夠的權限，並且已建立專案

## 在 Cloud Storage 建立一個 Bucket

第一步前往[Cloud Storage](https://console.cloud.google.com/storage/browser)，  
點擊 +CREATE BUCKET，  
在 Choose where to store your data 的區塊，  
Location type 有三種

- multi-region
- dual-region
- region

這是地理位置與高可用性的相關設定，越後面的設定成本越便宜，但是可用性也越低。  
即使如此 google 仍保証了 SLA: 99.95% 的高可用性。

接下來是 Choose a default storage class for your data 的區塊，有

- Standard
- Nearline
- Coldline
- Archive

等四種不同的設定，  
與檔案的使用頻率有關，對於網站來說建議使用 Standard 。
收費可以參考下表

|                                  | Standard | Nearline | Coldline | Archive |
| -------------------------------- | -------- | -------- | -------- | ------- |
| Storage(per GB-Month)            | $0.026   | $0.01    | $0.007   | $0.004  |
| retrieval(per GB-Month)          | Free     | $0.01    | $0.02    | $0.05   |
| Class A Operations(per 1000ops)  | $0.005   | $0.01    | $0.01    | $0.05   |
| Class B Operations(per 1000 ops) | $0.0004  | $0.001   | $0.005   | $0.05   |
| SLA                              | 99.95%   | 99.9%    | 99.9%    | 99.9%   |

接下來是 Choose how to control access to objects 的設定，  
不要勾選 Enforce public access prevention on this bucket，  
Access control 選擇 Uniform ， 這裡的設定是為了避免從 internet 存取 bucket 的資料，  
但是我們的目的是放置靜態網站的資料，所以不需設定。

再來是 Choose how to protect object data  
這是保護資料的策略，有版本(versioning)與備份(retention)兩種策略，  
我們不需要所以選擇 None

按下 Create 以建立 Bucket，  
接下來為了讓 web 存取我們選擇 more action(3 個點的 Icon ) > Edit Access  
New Principals > 選擇 allUsers > Storage Object Viewer.  
接下來可以上傳你的靜態網站的資源了，這裡我們多作一個設定，通常我們要指定網站的首頁為何，  
約定成俗是 index.html，一樣 more action(3 個點的 Icon ) > edit website configuration  
將 Index (main) page suffix 設定為 index.html(記得 bucket 裡要有這個檔)

## Load Balancing 相關設定

接下來設定 Load Balancing，  
這只需要設定一次，在目前的實作未涉及 VPC、CDN、Path Rules 等相關議題，  
實務上需要的話，請再加上去考量。

1. GCP > Network Service > Load Balancing
2. Create Load Balancer
3. Backend configuration > Create A Backend Bucket
   - 自已取一個 Backend Bucket name
   - Cloud Storage Bucket > Browse > 選取之前所建立的 Bucket
   - 不勾選 Enable Cloud CDN， 相關的設定與應用程式的應用有關， 較為複雜之後再進行處理
4. Host and path rules > Simple host and path rule
5. Frontend IP and PORT > Add Frontend IP And Port
   - Protocol > HTTP (正常應使用 HTTPS， 這次為了求快而未作相關設定)
   - Network Service Tier > Premium

可以參考 GCP 的教學與我們實作上的細微差異

- Create a bucket. → 手動建立只需要處理一次
- Upload and share your site's files. → CI 執行
- Set up a load balancer and SSL certificate. → 手動建立只需要處理一次
- Connect your load balancer to your bucket. → 手動建立只需要處理一次
- Point your domain to your load balancer using an A record. → 未處理
- Test the website.

## CI/CD 的相關設定

參考以下部份的 .gitlab-ci.yml 檔  
非常簡單，只需要 `gsutil rsync -R` 將前一個 job 建置的檔案推到 Cloud Storage 即可

```yml
deploy-job: # This job runs in the deploy stage.
  stage: deploy # It only runs when *both* jobs in the test stage complete successfully.
  image: google/cloud-sdk
  needs:
    - job: build-job
      artifacts: true
  script:
    # - gcloud auth list # Show the ACTIVE  ACCOUNT *
    - gsutil rsync -R build gs://your_bucket_name
    - echo "Application successfully deployed."
```

## 其它

這裡仍有一個未知的設定，gsutil 命令會需要權限才能對 Cloud Storage 寫入，  
如何讓 Gitlab-Runner 底下的 Container 擁有指定的 GCP 權限呢 ?  
試著下 `gcloud auth list` 會發現確實有一個可工作的 Account 所以一定有相關的設定要處理，  
因為沒有實作到，故不作記錄，但未來再有機會不要忘了這一段的工程。

大部份的工作只需要設定一次，未來只需要 CI 將新版的靜態網站上傳到 Cloud Storage 網站就會更新。

## 參考

- <https://cloud.google.com/storage/docs/hosting-static-website>
- <https://cloud.google.com/load-balancing/docs/https/ext-load-balancer-backend-buckets>

(fin)
