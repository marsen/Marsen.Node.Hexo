---
title: " [實作筆記] Gitlab CI/CD 與 GCP - 錯誤處理"
date: 2023/11/16 16:32:09
tags:
  - 實作筆記
  - CI/CD
---

## 前情提要

在持續整合/持續交付 (CI/CD) 的過程中，我們常常會遇到一個問題：  
當發生錯誤時，系統並不總是會拋出 exit 1 的錯誤碼。  
這種情況下，即使發生錯誤，CI/CD 仍然會繼續執行，這可能導致部署了錯誤的版本上線，增加系統的不穩定性。

## 解決方法

為了解決這個問題，我們可以採取一些措施，確保在發生錯誤時 CI/CD 立即停止，並拋出 exit 1 錯誤碼。

### 1. 記錄錯誤信息到 output.log

  首先，我們可以將所有的執行日誌輸出到一個文件，例如 output.log。這樣一來，  
  不管是哪個階段出現了問題，我們都能夠查閱這個文件，以便更好地理解錯誤的發生原因。  

  ```bash
  Copy code
  # 在腳本中添加以下命令，將所有輸出寫入 output.log 文件
  ./your_script.sh > output.log 2>&1
  ```

### 2. 使用 grep 檢查錯誤

  其次，我們可以使用 grep 命令來檢查 output.log 文件，查找關鍵字或錯誤模式。  
  如果發現了任何錯誤，我們可以採取相應的措施。

  ```bash
  Copy code
  # 使用 grep 查找關鍵字，並在發現錯誤時執行相應的操作
  if grep -q "error" output.log; then
      echo "Error found in output.log"
      exit 1
  fi
  ```

### 3. 發現錯誤時丟出 exit 1

  最後，在檢查完錯誤後，如果發現了問題，我們應該明確地拋出 exit 1，這會通知 CI/CD 停止進一步的運行，  
  確保不會將有問題的版本部署上線。  

  ```bash
  Copy code
  # 在發現錯誤時，明確地拋出 exit 1
  if [ $? -ne 0 ]; then
      echo "Error occurred. Exiting with code 1."
      exit 1
  fi
  ```

透過這些步驟，我們可以更好地管理 CI/CD 中的錯誤，確保系統的穩定性和可靠性。  
同時，我們能夠更快速地響應和解決問題，減少部署錯誤版本的風險。  

```yaml
generate-qa:
  stage: generate-qa
  script:
    - npm install
    - npm run generate:qa | tee output.log  # 可能出錯的命令，將輸出寫到文件
    - grep -q "Error.*\[500\]" output.log && echo -e "\e[31m生成異常！ Prerendering 發生 500 Error\e[0m" && exit 1
    - echo "generated successfully"
  artifacts:
    paths:
      - ./.output/public
  rules:
    - if: '$CI_COMMIT_BRANCH == "qa"'
```

## 小結

這些方法不僅有助於提高系統的穩定性和可靠性，還能夠加速問題的識別和解決過程，減少了部署錯誤版本的風險。  
透過這些實踐，我們可以更加信心滿滿地運用 Gitlab CI/CD 與 GCP，確保順暢的開發和部署流程。  

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

(fin)
