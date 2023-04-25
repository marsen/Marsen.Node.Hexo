---
title: "[實作筆記] QA DB 的連線方案決擇"
date: 2023/04/06 17:29:40
---

## 前情提要

GCP SQL是Google Cloud Platform上的一個全管理式的關聯式資料庫服務，  
可支援MySQL、PostgreSQL、SQL Server和Oracle資料庫，  
讓使用者在不需要擔心基礎架構運維的情況下，快速部署、維護、擴展和監控資料庫。  

現在我建立一個 QA 環境的 GCP SQL 沒有 public IP，  
不設定 public IP 連線 GCP SQL，可提高資安性並降低成本，  
因為可以避免公開暴露資料庫（比如 xRent），也不需要額外支付 Public IP 的費用。
那要如何安全的連線資料庫？這是一個常見的問題，因為沒有開啟 Public IP，必須要透過其他方式來連接。  

## 可能的方案

1. Cloud SQL Proxy
2. 建立 VPN 連線 VPC
3. 以及使用 SSH Tunnel

### Cloud SQL Proxy

優點：使用 Cloud SQL Proxy 可以簡單地建立一個加密和安全的通道，使您的應用程序可以安全地連接到 Cloud SQL。
這種方法不需要建立 VPN 或 SSH 通道，並且具有與使用公共 IP 類似的方便性，但同時也提供了更高的安全性。
缺點：需要安裝和配置 Cloud SQL Proxy，整體而言比較麻煩。

1. 下載 Cloud SQL Proxy ,設定執行權限
2. 建立 CSP 的 IAM ,並下載設定金鑰
3. 連線的 SQL 沒有 public IP 會有以下錯誤  
   > Failed to connect to instance: Config error: instance does not have IP of type "PUBLIC"
4. 連線的 SQL 沒有 private IP 時需要使用 VPN 連線 VPC  

### 建立 VPN 連線 VPC

優點：使用 VPN 可以建立一個私人和安全的通道，使您的應用程序可以與 Cloud SQL 通信，並且可以通過相同的 VPC 網絡進行通信，可以輕鬆地將多個客戶端連接到 Cloud SQL。
缺點：需要在 GCP 上設置和配置 VPN 網絡，並且需要安裝和配置 VPN 客戶端，設置和維護成本較高。

### 使用 SSH Tunnel

優點：使用 SSH 隧道可以通過一個安全的加密通道來連接 Cloud SQL，而不需要在 GCP 上設置和配置 VPN 網絡，可以輕鬆地將多個客戶端連接到 Cloud SQL。
缺點：需要在每個客戶端上安裝和配置 SSH 客戶端，且在高流量情況下可能會有性能問題。同時，由於 SSH 通道需要打開 SSH 端口，因此也存在一定的安全風險。

## 實作

在這篇文章中，我們將介紹如何使用 DataGrip SSH 來建立 SSH 隧道，以連接到 GCP SQL。

步驟一：打開 DataGrip，進入 Database 視窗。在左上角的方框中，點選「＋」號，選擇「Data Source」。

步驟二：在彈出的視窗中，選擇「MySQL」。在右側的表格中，填寫資料庫的連接資訊，包括 Host、Port、Database Name、User Name、Password 等等。

步驟三：在左側的列表中，選擇「SSH/SSL」。勾選「Use SSH tunnel」，填寫 SSH tunnel 的資訊，包括 Host、Port、User name、Auth type、Private key file 等等。注意：Host 和 Port 應該是 SSH 服務器的地址和端口，而不是 GCP SQL 的地址和端口。

步驟四：在左側的列表中，選擇「SSL」。勾選「Use SSL」，填寫 SSL 的資訊，包括 CA file、Client certificate file、Client key file 等等。這些資訊應該已經在上面的對話中解釋過了。

步驟五：在完成上述步驟之後，按下「Test Connection」，測試是否可以成功連接到 GCP SQL。如果成功，就可以儲存這個資料庫的連接資訊，並且開始使用它了。

## 小結

透過 DataGrip SSH 建立 SSH 隧道來連接到 GCP SQL，是一種簡單而有效的方法。  
使用這個方法，我們可以不必為 GCP SQL 啟用 public IP，同時又能夠安全地連接到資料庫。  
當然，這個方法也有一些限制和缺點，需要根據實際情況進行選擇和應用。

(fin)
