---
title: " [實作筆記] Gitlab CI/CD 與 GCP - Workload Identity Federation"
date: 2024/03/13 15:25:55
tags:
  - 實作筆記
  - CI/CD
---
## 前情提介

現況用 Service Account 會有什麼問題？  
當我們希望與第三方的服務與 GCP 作整合時，  
傳統的（我之前的）作法是透過建立 Service Account Key 去提供給第三方資源存取的權限。  
這會產生資安隱患，主要是這把 Key 的權粒度大難以稽核，有效期長風險，  
而要定期更換 Key 會變成一個麻煩的管理問題。

### 需求介紹

我目前透過 GCS 並掛載 Load Balancing 部署靜態網站，  
而 CI/CD 是透過 Service Account 的 Key 去執行工作，  
這是一種有資安隱憂的作法，所以我試著使用 Workload Identity Federation 取代

## 概念

TLDR;

而 Workload Identity Federation 是基於 IAM 機制，允許第三方服務整合 GCP 資源，  
背後的技術原理是基於 OIDC， 在這裡我們不過度展開，簡單描述如下:

1. Gitlab CI／CD 首先取 Gitlab OIDC Token，取得 Token 的作法可以參考[官方文件](https://docs.gitlab.com/ee/ci/secrets/id_token_authentication.html)，下面是個簡單的範例:

  ```yaml

  job_with_id_tokens:
    id_tokens:
      FIRST_ID_TOKEN:
        aud: https://first.service.com
      SECOND_ID_TOKEN:
        aud: https://second.service.com
    script:
      - first-service-authentication-script.sh $FIRST_ID_TOKEN
      - second-service-authentication-script.sh $SECOND_ID_TOKEN
  ```

  ｜OIDC 是基於 Oauth2 的標準，簡單可以想成 Oauth2 再加上身份驗証。  
2. 有了  Gitlab OIDC Token，我們可以透過 GOOGLE STS(Security Token Service) API 取得　Federated Token  
  在這裡我們需要先建立好 Workload Identity Provider(IdP)，而可以設定 Attribute Conditions 來作限制  
3. 這個時候可以用 Federated Token 與 GCP IAM API 交換來一個短周期的 Access Token  
4. 本質上還是用 Service Account 在作事，但是用短周期的 Access Token 取代 Key, 從而簡化了 Key 的管理工作  

## 實作步驟

1. 建立 Workload Identity Pool

    ```shell
    #Update $GCP_PROJECT_ID value
    gcloud iam workload-identity-pools create gitlab-test-wip \
      --location="global" \
      --description="Gitlab demo workload Identity pool" \
      --display-name="gitlab-test-wip" \
      --project=$GCP_PROJECT_ID
    ```

2. 設定 workload identity pool provider 並建立 Attribute conditions,  
這步的關鍵是讓只符合你條件設定的 User 才能取得 Token

    ```shell
    #Update GITLAB_NAMESPACE_PATH value
    gcloud iam workload-identity-pools providers create-oidc gitlab-identity-provider --location="global" \
    --workload-identity-pool="gitlab-test-wip" \
    --issuer-uri="https://gitlab.com" \
    --allowed-audiences=https://gitlab.com \
    --attribute-mapping="google.subject=assertion.sub,attribute.aud=assertion.aud,attribute.project_path=assertion.project_path,attribute.project_id=assertion.project_id,attribute.namespace_id=assertion.namespace_id,attribute.namespace_path=assertion.namespace_path,attribute.user_email=assertion.user_email,attribute.ref=assertion.ref,attribute.ref_type=assertion.ref_type" \
    #--attribute-condition="assertion.namespace_path.startsWith(\"$GITLAB_NAMESPACE_PATH\")" \
    --attribute-condition="assertion.namespace_path.startsWith(\"marsen\")" \
    --project=$GCP_PROJECT_ID
    ```

3. 建立 GCP Service Account
在我的例子中

    ```shell
    #Create a service account
    gcloud iam service-accounts create gitlab-runner-sa --project=$GCP_PROJECT_ID

    #Add sample permissions to the Service account
    gcloud projects add-iam-policy-binding $GCP_PROJECT_ID \
      --member=serviceAccount:gitlab-wif-demo@${GCP_PROJECT_ID}.iam.gserviceaccount.com \
      --role=roles/storage.admin
    ```

4. 建立 Service Account 與 WIP 的角色關係綁定

    可以先取得專案的 GCP Project Id

    ```shell
    PROJECT_NUMBER=$(gcloud projects describe $(gcloud config get-value core/project) --format=value\(projectNumber\) --project $GCP_PROJECT_ID)
    ```

    設定 Service Account 的角色為 workloadIdentityUser，並將其設定為 workloadIdentityPools 的服務帳戶

    ```shell
    gcloud iam service-accounts add-iam-policy-binding gitlab-runner-sa@${GCP_PROJECT_ID}.iam.gserviceaccount.com \
        --role=roles/iam.workloadIdentityUser \
        --member="principalSet://iam.googleapis.com/projects/$PROJECT_NUMBER/locations/global/workloadIdentityPools/gitlab-test-wip/*"
    ```

5. 建立 Gitlab CI/CD 進行測試

```yaml
image: node:20.9.0-alpine

.gcp_wif_auth: &gcp_wif_auth
  #id_tokens to create JSON web tokens (JWT) to authenticate with third party services.This replaces the CI_JOB_JWT_V2
  id_tokens:
    GITLAB_OIDC_TOKEN:
      aud: https://gitlab.com
  before_script:
    - apt-get update && apt-get install -yq jq
    #Get temporary credentials using the ID token
    - |
      PAYLOAD=$(cat <<EOF
      {
      "audience": "//iam.googleapis.com/${GCP_WORKLOAD_IDENTITY_PROVIDER}",
      "grantType": "urn:ietf:params:oauth:grant-type:token-exchange",
      "requestedTokenType": "urn:ietf:params:oauth:token-type:access_token",
      "scope": "https://www.googleapis.com/auth/cloud-platform",
      "subjectTokenType": "urn:ietf:params:oauth:token-type:jwt",
      "subjectToken": "${GITLAB_OIDC_TOKEN}"
      }
      EOF
      )
    - |
      echo "Payload: ${PAYLOAD}"
    - |
      FEDERATED_TOKEN=$(curl -s -X POST "https://sts.googleapis.com/v1/token" \
      --header "Accept: application/json" \
      --header "Content-Type: application/json" \
      --data "${PAYLOAD}" \
      | jq -r '.access_token'
      )
    #- | 
    #  echo "Federated Token: ${FEDERATED_TOKEN}"
    #Use the federated token to impersonate the service account linked to workload identity pool
    #The resulting access token is stored in CLOUDSDK_AUTH_ACCESS_TOKEN environment variable and this will be passed to the gcloud CLI
    - |
      WHAT_IT_IS=$(curl -s -X POST "https://iamcredentials.googleapis.com/v1/projects/-/serviceAccounts/${SERVICE_ACCOUNT_EMAIL}:generateAccessToken" \
      --header "Accept: application/json" \
      --header "Content-Type: application/json" \
      --header "Authorization: Bearer ${FEDERATED_TOKEN}" \
      --data '{"scope": ["https://www.googleapis.com/auth/cloud-platform"]}' \
      | jq -r '.'
      )
    - |
      echo "WHAT_IT_IS: ${WHAT_IT_IS}"
    - |
      export CLOUDSDK_AUTH_ACCESS_TOKEN=$(curl -s -X POST "https://iamcredentials.googleapis.com/v1/projects/-/serviceAccounts/${SERVICE_ACCOUNT_EMAIL}:generateAccessToken" \
      --header "Accept: application/json" \
      --header "Content-Type: application/json" \
      --header "Authorization: Bearer ${FEDERATED_TOKEN}" \
      --data '{"scope": ["https://www.googleapis.com/auth/cloud-platform"]}' \
      | jq -r '.accessToken'
      )

stages:
  - deploy-prod

deploy-prod:
  variables:
    GCP_PROJECT_NAME: my-project-9527
    GCP_WORKLOAD_IDENTITY_PROVIDER: "projects/000009527/locations/global/workloadIdentityPools/gitlab-test-wip/providers/gitlab-identity-provider"
    SERVICE_ACCOUNT_EMAIL: "gitlab-runner@my-project9527.iam.gserviceaccount.com"
  <<: *gcp_wif_auth
  stage: deploy-prod
  image: google/cloud-sdk:latest
  script:
    - echo "Deploying artifacts to PROD GCS🚀🚀🚀"
    - echo $CLOUDSDK_AUTH_ACCESS_TOKEN
    - gcloud config set project ${GCP_PROJECT_NAME}
    - gcloud storage cp -r $CI_PROJECT_DIR/dist/* gs://my-static-website/

```

## 參考

- <https://cloud.google.com/iam/docs/workload-identity-federation>
- <https://www.youtube.com/watch?v=4vajaXzHN08>
- [GCP IAM generateAccessToken](https://cloud.google.com/iam/docs/reference/credentials/rest/v1/projects.serviceAccounts/generateAccessToken)
- [Security Token Service API](https://cloud.google.com/iam/docs/reference/sts/rest)
- [Workload 可能的資安風險:How Attackers Can Exploit GCP’s Multicloud Workload Solution](https://ermetic.com/blog/gcp/how-attackers-can-exploit-gcps-multicloud-workload-solution/)
- OIDC 相關
  - [OIDC(OpenID Connection)](https://hackmd.io/@Burgess/rkjLdxbmU#)
  - [Ory - OAuth2 and OpenID Connect](https://www.ory.sh/docs/oauth2-oidc)
  - [Gitlab OIDC](https://docs.gitlab.com/ee/ci/cloud_services/google_cloud/)
  - [一文带你搞懂OAuth2.0](https://juejin.cn/post/7175385017479069754?from=search-suggest)
  - [彻底搞懂OAuth2.0第三方授权免登原理](https://juejin.cn/post/7340481613144293395)
  - [認識 OAuth 2.0：一次了解各角色、各類型流程的差異](https://www.technice.com.tw/experience/12520/)
  - [Google Oauth2 API Explained](https://medium.com/@pumudu88/google-oauth2-api-explained-dbb84ff97079)
- [實作筆記] Gitlab CI/CD 與 GCP 相關文章
  - [架構全貌](https://blog.marsen.me/2023/04/13/2023/gitlab_ci_and_gcp_vm/)
  - [建立 Web Server VM](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_create_server/)
  - [建立 Gitlab Runner VM](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_cretae_runner/)
  - [防火牆設定](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_firewall/)
  - [Linux User 與資料夾權限](https://blog.marsen.me/2023/04/24/2023/gitlab_ci_and_gcp_vm_account/)
  - [機敏資料的處理](https://blog.marsen.me/2023/05/29/2023/gitlab_ci_and_gcp_vm_secret_config/)
  - [錯誤處理](https://blog.marsen.me/2023/11/16/2023/gitlab_ci_error_handle/)

(fin)
