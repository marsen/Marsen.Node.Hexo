---
title: " [實作筆記] Gitlab CI/CD 與 GCP - 機敏資料的處理"
date: 2023/05/29 16:41:02
tags:
  - 實作筆記
  - CI/CD
---

## 前言

請參考[前篇](https://blog.marsen.me/2023/04/13/2023/gitlab_ci_and_gcp_vm/) 架構,  
我們已經可以部署程式了.
更進一步討論一些實務上的狀況,這篇我們來討論如何處理一些特別的環境設定。

## 功能、組態與祕密

我們的程式有時會有一些設定,依照不同情境我的分類如下：  
功能,是指提供給使用者的設定,簡單的有訂閱通知的開關,或是回呼的 callback url 等.  
組態,不牽扯商業邏輯,純屬系統的設定,例如：站台第三方交互的網址,Stage 的名稱等.  
祕密,相當於組態,但是不可以外留,否則有資安風險,例如：ClientId、Token、Secret 等.

今天討論的會是組態與祕密,然後我想參考一些資料,  
一個是分散系統的建議作法 [12 Factors - Config](https://12factor.net/config)

> The twelve-factor app stores config in environment variables (often shortened to env vars or env).  
> Env vars are easy to change between deploys without changing any code; unlike config files,  
> there is little chance of them being checked into the code repo accidentally; and unlike custom config files,  
> or other config mechanisms such as Java System Properties, they are a language- and OS-agnostic standard.

另一個是 Php Laravel 的官方作法

> Laravel utilizes the DotEnv PHP library. In a fresh Laravel installation,  
> the root directory of your application will contain a .env.example file that defines many common environment variables.  
> During the Laravel installation process, this file will automatically be copied to .env.

## 想法

理想上、長遠上來我想要走到符合 12 Factory 的建議,  
實務上考慮到現實情況和時間限制,加上我對 PHP 和 Laravel 不太熟悉：  
現有系統的架構又不是分散式,我決定優先處理減少人為部署的工,而選擇以下的解決方案,

1. 使用 Gitlab CI/CD 變數來保護敏感資料：  
   Gitlab 提供了 CI/CD Variables 功能,可以將敏感資料儲存在 Group Variables 變數中
2. 在 CI 過程中置換這些變數。  
   這樣可以避免將敏感資料直接寫入到 .env 檔案中,提高了安全性(至少機敏資料不會在 Teams 或 Slack 中丟來丟去,或是寫入版控)。  
   使用讀取 .env.example 檔案的方式,然後根據需要替換其中的參數值。  
   這樣一來,我們可以根據不同的環境需求,動態生成 .env 檔案,而不需要手動修改原始 .env 檔案。

這樣的解決方案結合了 Gitlab CI/CD 的變數功能和動態生成 .env 檔案的方式,使得我們能夠更安全和靈活地管理參數。  
雖然這不是 Laravel 官方的最佳實踐,但在缺乏 Laravel 經驗的情況下,這是一個可行且有效的解決方案。

重要的是,這個解決方案能夠讓我們在開發過程中保護敏感資料,同時又不需要深入理解 Laravel 的配置機制。  
希望這個解決方案能對其他面臨相同問題的開發者有所幫助。  
如果你對於 Laravel 的配置機制比較熟悉,當然也可以探索更適合的方法來保護參數。

## 實作

![Gitlab Group](/images/2023/gitlab_group.png)
如圖,GitLab Group 是 GitLab 平台上的一個功能,它允許用戶在同一組織或專案的上下文中管理多個項目,方便協作、權限管理和組織層次的控制。

在 GitLab 中,Group 除了提供組織和專案的管理功能外,還可以使用 CI/CD Variables（持續整合/持續部署變數）來保護敏感資料。  
CI/CD Variables 是在 GitLab CI/CD 過程中使用的環境變數,可以在項目層級或 Group 層級定義。

通過 Group 層級的 CI/CD Variables,你可以在整個 Group 內的多個專案中共享和管理變數。  
這樣可以方便地保護和管理敏感資訊,例如 API 金鑰、密碼、配置設定等。  
透過使用 Group 層級的 CI/CD Variables,你可以在所有專案中統一管理這些敏感資料。

而下面是一段 gitlab-ci 的範例,CI 過程中讀取 Gitlab 的 Group Variables 的並生成 .env 檔

```yaml
- sed -e "s|#{APP_KEY}|$APP_KEY|"
  -e "s|#{GOOGLE_CLIENT_ID}|$GOOGLE_CLIENT_ID|"
  -e "s|#{GOOGLE_CLIENT_SECRET}|$GOOGLE_CLIENT_SECRET|"
  .env.qa > .env
```

然後在 GCP 的 VM 加上 IAM 設定,Gitlab 對每個帳號設定適合的角色,  
初步達成自動化與安全限制。

## 可能的(?)更好的作法

- 環境變數與分散式系統 ???
- 使用雲服務的 Secret Management ???
- 實作 Laravel Best Practice ???

如果你更好的作法請推薦給我

## 20230607 補充

使用 CI 建置不同環境的 `.env` 檔是一件痛苦的事，再更好的方法出現前，我只能儘量減少重複，  
下面是一個例子，我需要建立 QA 與 Production 的 `.env` 檔，  
而我目前的作法會在 Gitlab 的 CI/CD Variables 建立不同的 Key,  
像是 `MY_QA_APP_KEY` 與 `MY_PROD_APP_KEY`，  
而基礎的設定值又來自不同的檔案 `.env.qa`　與 `.env.production` apl
這真是一個糟糕的實作，不過為了動態組出不同的 key，  
可以參考以下的寫法，[**注意單雙引號有不同行為**](https://phoenixnap.com/kb/bash-single-vs-double-quotes)

```yaml
.config_temp: &config_script
  script:
    # Create Env
    - echo "KEY:$(eval echo '${'${PREFIX}'APP_KEY}')"
    - sed -e "s|#{APP_KEY}|$(eval echo '${'${PREFIX}'APP_KEY}')|"
      -e "s|#{GOOGLE_CLIENT_ID}|$(eval echo '${'${PREFIX}'GOOGLE_CLIENT_ID}')|"
      -e "s|#{GOOGLE_CLIENT_SECRET}|$(eval echo '${'${PREFIX}'GOOGLE_CLIENT_SECRET}')|"
      .env.${suffix} > .env
# 中略
config-qa:
  stage: build
  variables:
    PREFIX: "MY_QA_"
    suffix: "qa"
  needs: [build]
  <<: *config_script

config-production:
  stage: build
  variables:
    PREFIX: "MY_PROD_"
    suffix: "production"
  needs: [build]
  <<: *config_script
```

## 參考

- [Consul](https://www.consul.io/)
- [12 Factors - Config](https://12factor.net/config)
- [Laravel Configuration](https://laravel.com/docs/10.x/configuration)
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
