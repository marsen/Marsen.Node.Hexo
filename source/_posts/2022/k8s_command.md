---
title: " [實作筆記] 最近操作 k8s 的常用指令集錦 (透過 GKE 實作)"
date: 2022/08/17 12:00:23
tag:
  - 實作筆記
---

## Docker 相關

在建立映像檔的時候

`--build-arg` 可以提供參數給 Dockerfile

e.g

```shell
docker build \
--build-arg PackageServerUrl="{Url}" \
-t deposit -f ./Dockerfile .
```

## ~~在 container 裡~~ 一些 Linux 常用指令

有時候使用太輕量的 image 會缺少一些工具，  
我們可以直接換成有工具的 image。  
如果真的需要即時的在 container 裡面操作，  
可以參考以下語法
更新 apt-get

> apt-get update

透過 apt-get 安裝 curl (其它套件也適用)

> apt install curl

印出環境變數
方法一、 後面可以加上指定的變數名

> printenv {environment value}

方法二、 效果跟 printenv 一樣

> env | grep {environment value}

## GKE(k8s)相關指令

準備好 manifests.yml,

### GCP 授權

> gcloud container clusters get-credentials {kubernetes cluster name} --region {region} --project {project name}

e.g

- gcloud container clusters get-credentials gke-cluster-qa --region asia-east1 --project my-first-project

### 取得專案列表

kubectl config get-contexts

### 切換專案(prod/qa)

> kubectl config use-context {namespace}

e.g

> kubectl config use-context {prod_namespace}
> kubectl config use-context {qa_namespace}

### 套用 manifests(記得切換環境)

kubectl apply (-f FILENAME | -k DIRECTORY)

e.g

> kubectl apply -f ./manifests.yml
> kubectl apply -f ./k8s/manifests.yml --namespace=prod

### 查詢 configMap

- kubectl -n=qa get configmap {config_name} -o yaml

### 查詢 secret

- kubectl -n prod get secret {secret_name} -o json
- kubectl -n qa get secret {secret_name} -o json

### pods port 轉入開發者環境

> kubectl -n={namespace} port-forward {service} 80:80

e.g

> kubectl -n=qa port-forward service/deposit 80:80

### 取得資訊

- kubectl -n=qa get po
- kubectl -n=qa get deploy
- kubectl -n=qa get svc

### 進入 pods terminal 環境

> kubectl -n=qa exec -it {pods_name} -- /bin/bash

e.g

> kubectl -n=qa exec --stdin --tty my-service-5b777f56b8-q7lf7 -- /bin/sh

### 重啟服務

> kubectl rollout restart {service_name} -n qa

e.g

> kubectl rollout restart my-service-n qa

### 其它 CI 中用到的指令

- [envsubst](https://myapollo.com.tw/zh-tw/linux-command-envsubst/)
  - <https://medium.com/@SiegeSailor/%E4%BD%BF%E7%94%A8-envsubst-%E6%9B%BF%E6%8F%9B%E7%92%B0%E5%A2%83%E8%AE%8A%E6%95%B8-8c2826996fd5>
- [cut](https://www.javatpoint.com/linux-cut)
- [export](https://www.javatpoint.com/linux-export-command)

### 綜合應用

設定環境變數，並且置換 yml 檔中的參數，最後 apply 到線上環境

e.g

> export K8S_NAMESPACE=qa && envsubst < k8s/manifests.yml | kubectl -n=$K8S_NAMESPACE apply -f -
>
> export K8S_NAMESPACE=qa && envsubst < k8s/configmap.qa.yml | kubectl -n=$K8S_NAMESPACE apply -f -

### manifests.yml 的範例

#### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: $SERVICE_NAME
  namespace: $K8S_NAMESPACE
spec:
  replicas: 1
  selector:
    matchLabels:
      app: $SERVICE_NAME
  template:
    metadata:
      labels:
        app: $SERVICE_NAME
    spec:
      containers:
        - name: $SERVICE_NAME
          image: "asia-east1-docker.pkg.dev/$PROJECT_NAME/docker/$SERVICE_NAME:latest"
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
          envFrom:
            - configMapRef:
                name: my-service-config
            - secretRef:
                name: my-service-secret
```

#### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: $SERVICE_NAME
  namespace: $K8S_NAMESPACE
spec:
  type: ClusterIP
  selector:
    app: $SERVICE_NAME
  ports:
    - port: 8080
      targetPort: 8080
```

#### ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-service-config
  namespace: $K8S_NAMESPACE
data:
  THIRD_PARTY_URL: "https://third_party_url/"
  THIRD_PARTY_TOKEN: TBD
  THIRD_PARTY_ID: TBD
```

#### Secret

!!要記得作 [base64 Encode](https://www.base64encode.org/)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-service-secret
  namespace: $K8S_NAMESPACE
data: secret.Key:<<must encoded base64>>
```

#### manifest 實作上的 tip

更多參考官方文件與範例，設定太多了，不懂就去查  
可以用 `----` 串連多份文件，但我實務上不會串 `Secret`
e.g

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: $SERVICE_NAME
  namespace: $K8S_NAMESPACE
spec:
  replicas: 1
  selector:
    matchLabels:
      app: $SERVICE_NAME
  template:
  ## 中間省略
----
apiVersion: v1
kind: Service
metadata:
  name: $SERVICE_NAME
  ## 中間省略
----
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-service-config
  ## 以下省略
```

## 參考

- [使用-envsubst-替換環境變數](https://medium.com/@SiegeSailor/使用-envsubst-替換環境變數-8c2826996fd5)
- [kubernetes](https://kubernetes.io/)

(fin)
