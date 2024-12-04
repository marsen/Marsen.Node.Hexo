---
title: " [實作筆記] GCP 防火牆設定與 Health Check"
date: 2024/12/04 16:16:13
---

## 前言

GCP 防火牆（Firewall）用來控制進出 Google Cloud 資源的網路流量，確保服務的安全性。  
而 Health Check 則是用來監控後端服務的健康狀態，透過 GCP 負載平衡器（Load Balancer）進行流量分配。  
我將介紹如何在 GCP 設定防火牆規則以支持 Health Check 的運作。

## 情境

instance 有一個 Allow Load Balancer Health checks 選項，  
被勾起來後　Network tags 會加上`lb-health-check`
但是不知為什麼狀態仍然會顯示為 Off　？

所以我們必須多作一些步驟

我有許多服務都掛在負載平衡器之後，並且使用 Backend Service 來管理這些流量。  
一個 LB 會有多個 Backend Service (本文先不涉及 Backend Bucket)  
由於成本考量，為了提高資源利用率，一台機器可能會承載多個 Web 服務。  
這樣做可以節省硬體和虛擬機的成本，但同時也需要確保每個 Web 服務能夠獨立且穩定地運行。

在這樣的架構中，LB 會依 Path Rules 將流量分配到多個實例組（Instance Group）中，  
每個實例中可能運行著多個 Web 服務。  
透過這些實例和 Backend Service 的組合，負載平衡器能夠有效地將流量分配到不同的服務上。  
然而，為了確保服務的穩定性，我們必須配置健康檢查（Health Check）來監控每個 Web 服務的運行狀態。  

在 GCP 中，當我們使用負載平衡器（`Load Balancer`）時，會涉及以下資源：

- `Backend Service`：負責接收來自負載平衡器的流量，並將流量分發給後端的實例。
- `Health Check`：用來檢查後端服務是否健康，負載平衡器會依據這些檢查結果決定是否將流量導向特定實例。
- `Instance Group`：一組相同配置的虛擬機（VM），負責運行服務，並自動進行縮放。
- `Instance`：每個實際的虛擬機，執行後端服務。

這些資源之間的關係是，Backend Service 根據 Health Check 監控的結果選擇健康的 Instance，  
並將流量分發給 Instance Group 中的實例。  
當負載平衡器根據路徑規則（LB Path Rule）將流量導向某個服務時，這些流量會被引導到 Instance 中運行的 Web 服務。

我們會在一台 Instance 上使用多個 web 服務，使用不同的 Port 來區分，範圍是 6000~6100。  
而我們需要對這些所有的服務設定 Health Check

## 實作

每個實例有一個 **"Allow Load Balancer Health checks"** 的選項，  
勾選後會自動加上 `lb-health-check` 的網路標籤(Network Tag)。
不過不知道為什麼狀態仍會顯示為 Off。  
也就是說單純打勾是不夠的我們仍需要額外的設定，我們可以利用這個 `lb-health-check` 來作一些事

我們可以利用 Network Tag:`lb-health-check` 來建立相應的防火牆規則，  
確保來自負載平衡器的流量能夠順利進行健康檢查。  

接下來，我們可以創建一個防火牆規則，限制僅允許 6000~6100 端口的 TCP 流量。  
這樣做的原因是，健康檢查會直接使用 TCP 進行連接檢查，而不是 HTTP 協定，  
這樣可以避免 HTTP 協定中請求和響應的延遲，並且減少負載平衡器在檢查健康狀態時的處理開銷。

在 GCP Console 中創建防火牆規則，指定源 IP 或範圍（例如內部網路的 IP 範圍）。  
Google 的 Health Check 源自 `35.191.0.0/16` 與 `130.211.0.0/22`，  
設定在 Source filters > IP ranges 上  
Targets ＞ Target tags 請設定為 `lb-health-check`  
設定 Protocols and ports > Specified protocols and ports 選擇 TCP 協定 Ports 限制在 6000-6100。  
在 Health checks 中，Protocol 也設定 TCP，Port 要設定為真實的 Web 服務使用的 Port  
在 VM 的 instance 上，本來需要設定 Network Tag，但是我們可以勾選 "Allow Load Balancer Health checks" 的選項,  
就會自動加上 Tag 囉。

## 參考

- [GCP HEALTH CHECK CONCEPT](https://cloud.google.com/load-balancing/docs/health-check-concepts)
- [Use health checks](https://cloud.google.com/load-balancing/docs/health-checks)
- [Use VPC firewall rules](https://cloud.google.com/vpc/docs/using-firewalls)

(fin)
