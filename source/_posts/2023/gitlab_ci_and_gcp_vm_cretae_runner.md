---
title: "[實作筆記] Gitlab CI/CD 與 GCP - 建立 Gitlab Runner VM"
date: 2023/04/14 15:55:41
tag:
  - CI/CD
---

## 前言

請參考[前篇](https://blog.marsen.me/2023/04/13/2023/gitlab_ci_and_gcp_vm/)，我們將建立兩台 VM，  
一台作為 CI/CD 用的 Gitlab Runner，另一台作為 Web Server，  
本篇將介紹 Gitlab Runner 的相關設定，  
架構如下
![GCP 與 Gitlab](/images/2023/gitlab-gcp.jpg)

## 建立 VM

建立　GCP VM 步驟如下：

1. 登入 GCP 控制台並選擇適合的專案。
2. 在主選單中選擇「Compute Engine」，進入虛擬機器的管理頁面。
3. 點選「建立」(Create) 按鈕，開始建立新的虛擬機器。
4. 選擇適合的地區 (Region) 與區域 (Zone)，這會決定虛擬機器的物理位置與可用性。
5. 選擇適合的機器類型 (Machine type) 與規模 (Instance size)，這會決定虛擬機器的運算能力與資源配置。
6. 選擇適合的作業系統映像 (Operating System Image)，這會作為虛擬機器的基本作業系統。基本上我都選 Ubantu
7. 選擇適合的開機磁碟 (Boot disk) 配置與容量，這會作為虛擬機器的主要儲存空間。
8. 選擇適合的防火牆設定 (Firewall) 與網路 (Network) 設定，這會決定虛擬機器的網路連線與安全性設定。
9. 完成其他選項設定，例如使用者資料 (User data)、預設密鑰 (SSH Keys)、啟用擴展 (Enable extensibility) 等。
10. 檢閱與確認虛擬機器的設定，並按下「建立」(Create) 按鈕，開始建立虛擬機器。
11. 等待 GCP 完成虛擬機器的建立與啟動。
12. 成功建立虛擬機器後，可以透過 SSH 連線或其他遠端連線方式進入虛擬機器，並進行相關的設定與應用程式部署。

## 註冊 Group Runner

1. 安裝 GitLab Runner， 參考 [GitLab Runner 官方文件](https://docs.gitlab.com/runner/install/) 上的指南進行安裝。
2. 將該 Runner 註冊到您的 GitLab 項目中。注意 Runner 有 Group/Project/Shared 之分。
3. 我們選擇 Group Runner， 請安裝 CLI  

    ```terminal
    # 添加 Gitlab Runner 存儲庫的 GPG 金鑰
    $ curl -L https://packages.gitlab.com/gpgkey/gitlab-runner/gpgkey | apt-key add -

    # 添加 Gitlab Runner 存儲庫
    $ curl -L https://packages.gitlab.com/runner/gitlab-runner/ubuntu/$(lsb_release -cs)/

    # 更新 apt 軟體包索引
    $ apt-get update

    # 安裝 Gitlab Runner
    $ apt-get install gitlab-runner
    ```

4. 執行註冊

```terminal
# 註冊 Gitlab Runner，請按照提示輸入 Gitlab 的 URL 和 Runner 的註冊 token
$ gitlab-runner register
```

您可以在 GitLab 網站的項目設定中找到 Runner 註冊的相關資訊，並按照指南進行註冊。  
下面是 2023 年的實作記錄，如果你有遇到任何狀況，再查閱 Gitlab 相關文件.
> 註: 需注意 Gitlab 的官方文章指出，未來將計劃棄用 Gitlab Register Token 的方式
> > The new registration process is expected to become available in %16.0，  
> > and the legacy registration process will be available side-by-side  
> > for a few milestones before the being sunset through a feature flag.  
> > Removal is planned for %17.0.
>  
> 請[參考](https://gitlab.com/gitlab-org/gitlab/-/issues/380872)
  
首先，你必須是 Group Owner。  
接下來在 Group > CI/CD > Register a group runner > Registration token > 👁️ 取得 Token
執行 `gitlab-runner register`， 依提示輸入

- url 這是指 gitlab 的 服務網址，如果你不是自已架設的 gitlab server 請輸入 "https://gitlab.com/"  
- registration-token 這是在前面步驟取得的 token ，但這個作法預計在 Gitlab 17 版被棄用需注意
- executor 請輸入 "docker" ， 更多資訊請[參考](https://docs.gitlab.com/runner/executors/)

其它參數請[參考](https://docs.gitlab.com/runner/register/)
執行命令後，應該可以在　Group > CI/CD >　Runner 看到它，記得要啟動才會開始作業

## 建立 Group Variable

我們現在完成了 Gitlab-Runner，如果你也完成了你的服務伺服器設定，回頭看一下架構圖，　　
可以注意到，我們會透過 docker container 執行我們的 CI/CD 工作，
並且存取 GCP 的資源，在我們的例子中是使用 VM ，讓我們先忽略防火牆與使用者帳號的相關設定，　　
單純的連線機器實體，我會採用 SSH 的連線方式。　　

在一般的情況，我們會在　Client 端機器上生成 Private Key 與 Public Key，並且將　Public Key 放到 Server 端(服務伺服器)上，　　
不過在使用 docker container 的情況下，我們的 Client 端可以想像成是一個拋棄式的機器，  
每次都建立新的 Private/Public Key 會讓 Server 端記錄著一大堆沒有用的 Public Key，  
所以我們先建立好一組 Key，然後透過 [Gitlab Variable](https://docs.gitlab.com/ee/ci/variables/#for-an-instance) 的機制傳遞到 Docker Container。

### 設定

- 在群組中，前往「Settings > CI/CD」。
- 選擇「Add variable」並填入相關資訊：
  - Key：必須是單行文字，不能包含空格，只能使用字母、數字或底線。
  - Value：無限制。
  - Type：變數類型，預設為「Variable」，也可以選擇「File」。
  - Environment scope：可選，可以選擇套用到全部環境 (All) 或特定環境。
  - Protect variable：可選，如果選中，該變數只會在運行於受保護的分支或標籤的流程中使用。
  - Mask variable：可選，如果選中，該變數的值將在作業日誌中被遮蔽。如果變數的值不符合遮蔽要求，則無法保存。

在此例中我，我用「File」儲存 Private Key。  
未來可能會再評估一下資安風險。

## 參考

- Gitlab 相關
  - [Gitlab Group Runner](https://docs.gitlab.com/ee/ci/runners/runners_scope.html#group-runners)
  - [Registering runners (deprecated)](https://docs.gitlab.com/runner/register/#linux)
  - [Gitlab Changelog](https://gitlab.com/gitlab-org/gitlab-foss/blob/master/CHANGELOG.md)
  - [Deprecation - Support for registration tokens and server-side runner configuration parameters in `gitlab-runner register` command](https://gitlab.com/gitlab-org/gitlab/-/issues/380872)
  - [Gitlab Runner Executor](https://docs.gitlab.com/runner/executors/)
  - [Gitlab Variable](https://docs.gitlab.com/ee/ci/variables/#for-an-instance)
- [實作筆記] Gitlab CI/CD 與 GCP 相關文章
  - [架構全貌](https://blog.marsen.me/2023/04/13/2023/gitlab_ci_and_gcp_vm/)
  - [建立 Web Server VM](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_create_server/)
  - [建立 Gitlab Runner VM](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_cretae_runner/)
  - [Firewall](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_firewall/)

(fin)
