---
title: "[實作筆記] GCP Load Balancer"
date: 2023/04/17 14:47:41
---

## 前言

Load Balancing 的主要目的是確保系統的可靠性、高可用性和性能。  
透過平均分發網絡流量到多個伺服器，我們可以防止單一伺服器的過度負載，從而減少系統故障的風險。  
此外，使用負載平衡器還可以提供可擴展性，使我們能夠根據流量需求自動調整伺服器的數量，確保系統在高負載情況下仍能保持良好的性能。  

但是，我希望使用負載平衡器來達到降低對外 IP 地址和憑證費用的目的，以降低維運成本。  
透過只需為負載平衡器配置一個對外 IP 地址和憑證，而不需要為背後的多個伺服器分別配置對外 IP 地址和憑證。  

現在的公司，在舊架構上開了多台的 VM，並且都提供了對外的 IP address 與憑証，  
對我來說，不僅實際上較為省錢，管理上也會更好理解。

## 架構

![GCP LB](/images/2023/gcp_lb.jpg)

如圖所示，我有兩台 VM ， 一台用來放置 api 相關的服務，比如說，member 會員系統，  
另外還有一些整合性的服務，分別使用 Php 與 Nodejs 開發，簡單想成 api Server 就好，  
實際上在 VM 當中有掛載 Nginx 系統來管理其相對應的路由設定。  
另一台 VM 也不單純，儘管他只提供靜態的 Landing Page 網頁與 Vue 建置 SPA 網站，  
但在可預知的未來他會 Host 更多的靜態網站。  

也許可以考慮其它的 GCP 解決方案，ex:App Engine、Cloud Storage;  
但總之目前就是這樣的架構，而我主要目的是不希望花太多時間處理憑証與對外的固定 IP 的問題，  
所以本篇將會以這樣的架構進行建置 Load Balancer

## 實作

1. 建立後端服務：在這個案例我們建立兩台 VM，實際上你也可使用 Cloud Storage 或 Container。
2. 配置後端服務的群組：在這個案例我們建立兩個 Group，可以參加架構圖，真正的實務上，你的 Group 應該會有多個(至少要兩個) VM。
3. 建立 Load Balancer：在本案我們使用 HTTP(S) Load Balancing
4. 設定 Frontend configuration，在這裡我們給它一個對外的固定 IP 並建立憑証(certificate)
5. 設定 Backend configuration，在這裡我們會建立一個名叫 Backend Service 的抽像層，選擇 Instance group(參考第2點，也可以在這階段再進行建立)
6. 設定 Routing rules，選擇 Simple host and path rule:  
    > 本案中，我們會有兩個 domain 分代表前後端 `api.example` 與 `app.example`，  
    host 設定為 domain，而路徑設定為`/*`，後端指向對應的 group ，如下  
    `api.example`|`/*`|`prod-api-group`  
    `app.example`|`/*`|`prod-f2e-group`

### GCP 與 Load Balancer 相關的服務  

整個建置流程與 SSL 憑証、IP Address、backend-services、instance-groups 等服務相關  
雖然在建立 LB 的過程中就會建立憑証(certificate)，  
但有些情況可以使用 gcloud 指令作細節的控管  

比如說，列出 SSL 憑證：  

```terminal
gcloud compute ssl-certificates list
```

更多資訊請執行以下語法，  

```terminal
gcloud compute addresses --help
```

同理，可以用類似的語法查詢其它的資源  

```terminal
gcloud compute instance-groups --help
```

```terminal
gcloud compute addresses --help
```

```terminal
gcloud compute backend-services --help
```

ChatGpt 有時會亂掰一些指令出來，我沒有一一實際執行過的這裡就不提供了。  
測試負載平衡器：在將負載平衡器放入實際生產環境之前，您應該進行測試，確保負載平衡器正確運作並且可以處理預期的流量。  
監控和調整：一旦負載平衡器正式運行，您應該定期監控其性能和健康狀況，並根據需求進行必要的調整和優化。  

## 參考

- [Cloud Load Balancing overview](https://cloud.google.com/load-balancing/docs/load-balancing-overview)

(fin)
