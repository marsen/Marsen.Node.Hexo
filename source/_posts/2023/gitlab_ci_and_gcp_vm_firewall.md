---
title: "[實作筆記] Gitlab CI/CD 與 CGP - 防火牆"
date: 2023/04/14 18:13:09
tag:
  - CI/CD
---

## 前言

請參考[前篇](https://blog.marsen.me/2023/04/13/2023/gitlab_ci_and_gcp_vm/)，  
我們已經建立兩台 VM，  
本篇將介紹 Firewall 的相關設定，  
架構如下
![GCP 與 Gitlab](/images/2023/gitlab-gcp.jpg)

## 情境

為了建立我們的 QA 測試環境，我們需要在 Google Cloud Platform (GCP) 上進行一些設定。  
由於 QA 環境只能允許公司內部網路的存取，但我們又需要使用 Google 登入功能，這就需要綁定真實的 Domain 與對外 IP。  
為了確保安全性，我們還需要設定防火牆，只允許經過公司路由的流量進入 QA 環境，以限制訪問權限。  
這樣可以確保只有公司內部的流量可以訪問 QA 環境，保證資料的安全。  
這個設定可以讓我們擁有一個安全、受控的 QA 環境，並且能夠順利使用 Google 登入功能，提供更好的使用體驗。

## 實作

登入至 GCP > VPC 網路防火牆設定頁面。在此頁面中，您可以新增防火牆規則。

接著，建立名稱為 "block-all" 的防火牆規則，並將優先級設定為 1000，以阻擋所有流量。  
設定來源 IP 範圍為 0.0.0.0/0，並將允許的協定設定為 "deny"，以阻擋所有網路流量。  
同時，建立名稱為 "allow-my-company" 的防火牆規則，並將優先級設定為 999，以允許公司內部網路的流量通過。  
設定來源 IP 範圍為公司內部網路的 IP 範圍，並設定允許的協定包括 HTTP（80）、HTTPS（443）、SSH（22）和 ICMP，以便允許相應的流量通過。  

最後，在創建防火牆規則時，務必勾選 "套用於標籤" 選項，並選擇要套用防火牆規則的 VM 的標籤。  
確認防火牆規則的設定與標籤套用正確無誤後，點選 "建立" 來創建防火牆規則。  
確保您的 VM 上都已經套用了 "block-all" 和 "allow-my-company" 這兩個防火牆規則的 tag，以確保該 VM 可以通過防火牆規則的設定進行存取。  
最後，確認這兩個防火牆規則都已保存並生效，以確保 QA 環境只能通過公司內部網路存取，並且仍然可以使用 Google 登入功能。  

建立 GCP 防火牆規則並套用 VM 標籤，確保 QA 環境的存取限制符合公司的需求。  

## 參考

- [實作筆記] Gitlab CI/CD 與 CGP 相關文章
  - [架構全貌](https://blog.marsen.me/2023/04/13/2023/gitlab_ci_and_gcp_vm/)
  - [建立 Web Server VM](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_create_server/)
  - [建立 Gitlab Runner VM](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_cretae_runner/)
  - [Firewall](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_firewall/)
(fin)
