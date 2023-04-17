---
title: " [實作筆記] 靜態網站部署整合 GCP (一) --- Artifacts Registry & Cloud Run"
date: 2021/12/22 17:48:42
tag:
  - 實作筆記
---

## 目的

有機會在 GCP 上實作了兩種不同方式的靜態網站部署，
一種是透過 Artifacts Registry 與 Cloud Run，  
另一種是透過 Google Cloud Storage 與 Load Balancing
特別記錄一下。

## Build Web

build 建置網站， EX: `flutter build web`、`npm rub build` ...

- flutter 建置的 web 靜態檔位置在 `build/web`
- `react-scripts build` 靜態檔位置在 `build` 底下

### 問題與解決方式

#### react build 不同環境的靜態檔

開發上會有不同環境需
預設執行 react-scripts build 會建置出 production 的靜態檔，  
故需要透過套件 [env-cmd](https://www.npmjs.com/package/env-cmd) 來幫我們建立不同的靜態檔，  
請參考以下的 package.json > scripts

```json
    "build": "react-scripts build"，
    "build:dev": "env-cmd -f .env.development react-scripts build"，
```

#### ERR_OSSL_EVP_UNSUPPORTED

截至 2021 年 12，nodejs LTS 版本為 `v16.13.1`  
Latest Features 為 `v17.3.0`  
如果 nodejs 的版本在 `v17.*.*` 以上  
執行 react-scripts build 會發生以下錯誤

```shell
Error: error:0308010C:digital envelope routines::unsupported
… …
…
opensslErrorStack: [ 'error:03000086:digital envelope routines::initialization error' ]，
library: 'digital envelope routines'，
reason: 'unsupported'，
code: 'ERR_OSSL_EVP_UNSUPPORTED'
```

解決方法，執行以下語法

> export NODE_OPTIONS=--openssl-legacy-provider

或是將 node version 換回 `v16.*.*` 的版本

> nvm use v16.13.1

## Build Image

如果選擇 Artifacts Registry 與 Cloud Run 的 solution  
必須撰寫 Dockerfile 來建立 Docker Images

### 問題 [Dart] no active package dhttpd

[dhttpd](https://pub.dev/packages/dhttpd) 是 Dart 的靜態網站解決方案，  
不過在部署到 Cloud Run 時會發生錯誤，  
更多資訊請參考 [stackoverflow](https://stackoverflow.com/questions/70388111/can-not-run-flutter-web-app-on-google-cloud-runner) 的提問與 [issueTracker](https://issuetracker.google.com/issues/211083676) 的後續

#### 20220308 已有解決方案，更新

加上 dart global activate 指令，這會在執行前啟用 dhttpd 的服務，
RUN 是建立鏡像檔的語法， CMD 才是 docker image 啟動時會執行的語法，  
所以應該將 activate 指令放在啟動

```yml
FROM docker.io/dart

WORKDIR /flutter

ENV PATH $PATH:$HOME/.pub-cache/bin

COPY build/web ./
COPY start.sh ./
EXPOSE 8082

CMD dart pub global activate dhttpd && dart pub global run dhttpd --port=8082 --host=0.0.0.0
```

參考聯結

- [StackOverflow](https://stackoverflow.com/questions/70388111/can-not-run-flutter-web-app-on-google-cloud-runner)
- [Google Issue Report](https://issuetracker.google.com/issues/211083676)

~~暫解是使用其它的靜態網站服務或套件 ，~~  
~~EX: [NGINX](https://www.nginx.com/) 、 npm 的 [serve](https://www.npmjs.com/package/serve) 套件，~~  
~~或是其它你熟悉的靜態網站服務。~~  
~~請參考以下 Dockerfile~~

```yml
# pull official base image
FROM node

# set working directory
WORKDIR /flutter

# install app dependencies
RUN npm i -g serve

# add `/app/node_modules/.bin` to $PATH
ENV PATH $PATH:$HOME/.pub-cache/bin

# add app
COPY build/web ./

EXPOSE 8080

# start app and run at 0.0.0.0
CMD serve -p 8080
```

## Artifacts Registry & Cloud Run

1. 登入私有的 Container Registry
2. 透過 CI 建置 image
3. 加上版本的 Tag
4. 推上 Container Registry

參考部份的 .gitlab-ci.yml

```yml
script:
  - echo "Compiling the image..."
  - echo -n $GCR_KEY | docker login -u _json_key_base64 --password-stdin https://asia-east1-docker.pkg.dev
  - docker build -t $CI_PROJECT_NAME . --no-cache
  - docker tag "$CI_PROJECT_NAME" "asia-east1-docker.pkg.dev/company_registry/docker/$CI_PROJECT_NAME:$ver"
  - docker push "asia-east1-docker.pkg.dev/company_registry/docker/$CI_PROJECT_NAME:$ver"
  - echo "Compile image complete."
```

接下手動設定 Cloud Run(這部份應該也可以自動化，未實作故無記錄)

1. Create Service
2. Container > General > Container Image URL > SELECT > ARTIFACT REGISTRY > 選取上面部署上去的 images
3. Container Port 8080
4. DEPLOY

缺點是每次 CI 跑完 Image 推上 Artifact 後，仍要手動到 Cloud Run 執行 Deploy。  
這個步驟應該放在 CI Server 上（即使是需要手動觸發也是），才不會有斷點。

(fin)
