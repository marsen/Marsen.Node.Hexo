---
title: "[踩雷筆記] Gitlab 整合 Azure DevOps Pipeline 以 python on Azure Functions 為例"
date: 2024/08/13 11:17:42
---

## 前情提要

我目前主要使用 GitLab 進行版本控制和持續集成(CI/CD)，  
主要整合 Google Cloud Platform (GCP) ，許多專案都運行在 GCP 上。  
因商務需求最近開始探索第二朵雲 Azure。  
雖然 Azure DevOps 也有提供 CI／CD 與 Repo 的解決方案;  
但為了減少邏輯與認知負擔，我希望能將 GitLab 與 Azure DevOps Pipeline 進行整合。  
具體來說，如果我要在 Azure Functions 上部署 Python 應用程式，  
我應該一樣將我的程式往 Gitlab 推送就好，而不用特別區分這個服務在哪朵雲上，需要不同的部署方式。  
面對這樣的需求，我想找到最簡單且高效的解決方案。  

## 實作

以這次的例來說，我需要控管 Azure 的 Serverless 解決方案「Azure Functions」的程式。  
但是 Azure DevOps Pipeline 有相當高度的整合 Azure Cloud，只要能將程式推送到 Azure DevOps Repo，  
部署就會相當簡單。
CI／CD 流程大致如下

- 建立相對應的權限與憑証並提供給 Gitlab-Runner
- RD 推送新版本程式給 Gitlab，觸發 Gitlab-Runner
- Gitlab-Runner 執行測試、建置等相關作業後，部署到 Azure Functions

1. **設置 Azure DevOps Pipeline**：
   1. 選擇 New pipeline
   2. Other Git
   3. 設定連線方式 & 選擇分支
      1. Connection name (任意命名)
      2. Repo URL 輸入 Gitlab Repo URL
      3. User Name (任意命名)
      4. Password / Token Key (Gitlab PAT，需注意效期)
   4. 使用「Azure Functions for Python」Template
      - Build extensions
      - Use Python 3.10(可以更換合適的版本)
      - Install Application Dependencies
      - Archive files
      - Publish Artifact: drop
   5. 追加 Agent job Task
      - 搜尋「Azure Functions Deploy」
      - 填寫
        - Azure Resource Manager connection
          - Manage > Service Connection > New Service Connection > Azure Resource Manager > Service principal(automatic)
            - Service connection name
            - Description (optional)
          - 也可以選擇 > Service principal (manual)，需要先加上 App registrations 具體流程如下：
            - 在 Azure Portal 上建立一個新的 Azure Registration。
            - 選擇 Certificates & secrets，建立一組 Certificates & secrets。
            - 回到 Service principal (manual)
              - Subscription Id
              - Subscription Name
              - Service Principal Id (App registrations Client secrets 的 Secret ID)
              - Service principal key (App registrations Client secrets 的　Value)
              - Tenant ID
        - App type (我的情況是選 Function App on Linux)
        - Azure Functions App name

2. **配置 GitLab CI/CD**：
   - 在 GitLab 中，建立 `.gitlab-ci.yml` 如下：

    ```yml
    image: debian:stable-slim
    variables:
      AZURE_PIPELINE_NAME: "dispatch-worker-deploy-pipeline"

    before_script:
      - apt-get update && apt-get install -y curl jq

    stages:
      - deploy

    trigger_pipeline:
      stage: deploy
      script:
        - |
          json=$(curl -u $AZURE_DEVOPS_USER:$AZURE_DEVOPS_PAT \
            -H "Content-Type: application/json" \
            "https://dev.azure.com/Aiplux/Inpas/_apis/build/definitions?api-version=6.0")
          id=$(echo $json | jq -r --arg pipeline_name "$AZURE_PIPELINE_NAME" '.value[] | select(.name==$pipeline_name) | .id')
          echo -e "\033[1;33mPipeline: $AZURE_PIPELINE_NAME ID is $id\033[0m"
          RESPONSE_CODE=$(curl -X POST "https://dev.azure.com/My_Organization/My_Project/_apis/build/builds?api-version=6.0" \
            --data '{"definition": {"id": '$id'}}' \
            -u ${AZURE_DEVOPS_USER}:${AZURE_DEVOPS_PAT} \
            -H 'Content-Type: application/json' \
            -w "%{http_code}" -o /dev/null -s)
          if [ "$RESPONSE_CODE" -ne 200 ]; then
            echo -e "\033[1;31mRequest failed with status code $RESPONSE_CODE\033[0m"
            exit 1
          fi

    rules:
      - if: '$CI_COMMIT_BRANCH == "main"'
    ```

    可以看到 AZURE_DEVOPS_USER 與 AZURE_DEVOPS_PAT 兩個參數,  
    可以在登入 Azure DevOps 後，在 User Settings >> Personal Access Tokens 取得,  
    實務上由管理者提供，但是有時效性，仍然需要定期更新（１年），應該有更簡便的方法才對。

完成以上的設定後，只要推送到 Gitlab Repo 的 main 分支，就會觸發 Azure DevOps Pipeline 部署。

## 踩雷

在整合過程中遇到了一個問題。  
儘管使用了 Azure Pipeline 提供的官方模板，部署過程依然出現了錯誤。
具體而言並沒有明顯的錯誤，但是 Log 會記錄

> **1 function found**  
> **0 function loaded**

導致 Azure Functions Apps 無法正常工作。

## 解法

[參考](https://github.com/Azure/azure-functions-python-worker/issues/708#issuecomment-765528255)

在 Azure Functions for Python 其中一步驟　Install Application Dependencies  
Template 如下：

```shell
python3.6 -m venv worker_venv
source worker_venv/bin/activate
pip3.6 install setuptools
pip3.6 install -r requirements.txt
```

需要修改成才能作用，不確定 Azure 會提出修正或新的 template

```shell
python3.10 -m venv .python_packages
source .python_packages/bin/activate
pip3.10 install setuptools
pip3.10 install --target="./.python_packages/lib/site-packages" -r ./requirements.txt
```

查閱了 Azure 和 GitLab 的官方文檔以及技術社群中的討論，找到了有效的解決方案。
通過修改 template，成功解決了部署問題，  
GitLab 和 Azure DevOps Pipeline 的整合成功。

## 參考

- [ModuleNotFoundError when using DevOps Pipeline Task AzureFunctionApp@1](https://github.com/Azure/azure-functions-python-worker/issues/708#issuecomment-765528255)

(fin)
