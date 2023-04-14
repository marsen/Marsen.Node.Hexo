---
title: "[實作筆記] Gitlab CI/CD 與 CGP - 架構全貌"
date: 2023/04/13 18:09:50
tag:
  - CI/CD
---

## 前言

在一個團隊合作開發的場景中，多個開發人員可能同時對同一個程式碼庫進行更改，  
而 CI/CD 可以自動進行建置、測試和部署，以確保程式碼的品質和穩定性。  
此外，CI/CD 還可以幫助提高交付速度，減少錯誤，並支持快速反饋和迭代開發。  

在這個背景下，選擇 GitLab CI/CD 與 Google Cloud Platform（GCP）作為 CI/CD 解決方案有幾個優點。  
首先，GitLab CI/CD 提供了完整的 CI/CD 功能，包括強大的持續整合和持續部署工具，能夠無縫集成到 GitLab 的版本控制平台中，  
方便團隊進行版本控制、自動建置和測試，並實現自動化的程式碼部署。  
其次，GCP 是一個廣泛使用的雲端平台，提供了豐富的計算、儲存、網路和資料庫等服務，  
方便團隊在雲端環境中進行應用程式部署和運行，並且與 GitLab CI/CD 有良好的整合性，可以輕鬆設定和管理 CI/CD 流程。  

替代方案：若不選擇 GitLab CI/CD 與 GCP，團隊可能需要考慮其他 CI/CD 工具和雲端平台的組合。  
例如，替代 GitLab 的版本控制工具可以是 GitHub、Bitbucket 等；替代 GCP 的雲端平台可以是 AWS、Azure 等。  

## 架構

在我們的架構之中，我們使用 Docker 作為 GitLab Runner 的執行環境，  
並在 GCP 上使用虛擬機器（VM）作為運行應用程式(web server)部署環境。  
透過 GitLab Runner 和 Docker，我們能夠自動執行 CI/CD 任務，  
並根據需要動態調整運行環境，確保我們的應用程式在開發和部署過程中保持高品質和穩定性。  
如下圖。

![GCP 與 Gitlab](/images/2023/gitlab-gcp.jpg)

## 步驟

接下來我將進行以下動作：

- 在 GCP 上建立兩個虛擬機器（VM），一個作為我們的 Web Server，另一個作為 GitLab Runner。
- 將 GitLab Runner VM 註冊為 GitLab Group runner，以便進行自動化的建置和部署。
- 設定適當的防火牆規則，讓 Gitlab Runner 可以存取 Web Server
- 讓 GitLab Runner VM 的 Docker 容器可以存取我們的 Web Server，我會設定相應的公鑰和私鑰，以確保安全的連線。
- 我會撰寫一個適合的 gitlab-ci.yml 檔案，來定義建置和部署的流程，並將其配置在 GitLab CI/CD 中，以實現自動化的流程。
- 最後，我會在 Web Server 上設定好使用者、群組與 Nginx 的 Hot reload。

以上的步驟將協助我們建立基於 Gitlab 與 GCP 的 CI/CD 流程，並以此提供程式開發的品質和穩定性。

## 參考

- [實作筆記] Gitlab CI/CD 與 CGP 相關文章
  - [架構全貌](https://blog.marsen.me/2023/04/13/2023/gitlab_ci_and_gcp_vm/)
  - [建立 Web Server VM](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_create_server/)
  - [建立 Gitlab Runner VM](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_cretae_runner/)
(fin)
