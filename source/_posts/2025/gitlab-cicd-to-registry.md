---
title: " [實作筆記] 用 GitLab CI/CD 實現自動打 Tag、Build 與 Push Docker Image 到 Registry"
date: 2025/06/05 19:11:43
tags:
  - 實作筆記
---

## 前情提要

在專案自動化部署流程中，讓 CI/CD pipeline 自動產生遞增 tag、build Docker image 並推送到 GitLab Container Registry，是現代 DevOps 的常見需求。

這篇文章記錄我在 GitLab CI/CD 上實作這一流程的經驗。

### 目標

- **自動產生遞增 tag**（如 ver.1.0.1）
- **自動 build 並 push Docker image**，image tag 與 git tag 同步
- **支援多專案共用同一份 CI 設定**
- **僅 main 分支觸發**

## 實作

1. 變數抽象化，支援多專案共用

    首先，將專案名稱、群組路徑等資訊抽成變數，方便不同專案複用：

    ```yaml
    variables:
      PROJECT_NAME: my-proj
      GROUP_PATH: my-group
      REGISTRY_PATH: registry.gitlab.com/$GROUP_PATH/$PROJECT_NAME
    ```

2. 多階段 Dockerfile 精簡 image

    使用 multi-stage build，確保 production image 只包含必要檔案與 production dependencies：

    ```dockerfile
    FROM node:23-alpine AS builder
    WORKDIR /app
    COPY package*.json ./
    RUN npm install
    COPY . .
    RUN npm run build

    FROM node:23-alpine AS production
    WORKDIR /app
    COPY package*.json ./
    RUN npm install --production
    COPY --from=builder /app/dist ./dist
    ENV NODE_ENV=production
    EXPOSE 3000
    CMD ["npm", "start"]
    ```

    建議搭配 .dockerignore 避免多餘檔案進入 image。

3. GitLab CI/CD Pipeline 設定只在 main 分支執行

    ```yaml
    rules:
      - if: '$CI_COMMIT_BRANCH == "main"'
        when: always
      - when: never
    ```

4. 自動產生遞增 tag

   1. 採用 major.minor.patch 的版本規則，啟始版號為 ver.1.0.0
   2. 先同步 remote tag，避免本地殘留影響
   3. 取得最大 patch 號，自動 +1
   4. 檢查 tag 是否已存在，確保唯一

    ```yaml
    script:
      - git fetch --prune --tags
      - git tag -l | xargs -n 1 git tag -d
      - git fetch --tags
      - |
        prefix="ver."
        max_patch=$(git tag -l "${prefix}[0-9]*.[0-9]*.[0-9]*" | grep -oE '[0-9]+\.[0-9]+\.[0-9]+' | sort -t. -k1,1nr -k2,2nr -k3,3nr | head -n1 | awk -F. '{print $3}')
        if [ -z "$max_patch" ]; then
          new_tag="${prefix}1.0.0"
        else
          major=$(git tag -l "${prefix}[0-9]*.[0-9]*.[0-9]*" | grep -oE '[0-9]+\.[0-9]+\.[0-9]+' | sort -t. -k1,1nr -k2,2nr -k3,3nr | head -n1 | cut -d. -f1)
          minor=$(git tag -l "${prefix}[0-9]*.[0-9]*.[0-9]*" | grep -oE '[0-9]+\.[0-9]+\.[0-9]+' | sort -t. -k1,1nr -k2,2nr -k3,3nr | head -n1 | cut -d. -f2)
          patch=$((max_patch + 1))
          new_tag="${prefix}${major}.${minor}.${patch}"
        fi
        # 確保 tag 唯一
        while git rev-parse "$new_tag" >/dev/null 2>&1; do
          patch=$((patch + 1))
          new_tag="${prefix}${major}.${minor}.${patch}"
        done
        echo "new tag is $new_tag"
        echo "NEW_TAG=$new_tag" >> build.env
      - source build.env
    ```

5. 自動打 tag 並 push

    ```yaml
      - git config --global user.email "rd@example.com"
      - git config --global user.name "gitlab-runner"
      - git remote set-url origin https://gitlab-runner:$RAG_CICD_TOKEN@gitlab.com/$GITLAB_REPO.git
      - git tag "$NEW_TAG"
      - git push origin "$NEW_TAG"
    ```

6. Loging、Build & Push Docker image

    ```yaml
      - docker login -u "$CI_REGISTRY_USER" -p "$CI_REGISTRY_PASSWORD" $CI_REGISTRY
      - docker build -t $REGISTRY_PATH:$NEW_TAG .
      - docker push $REGISTRY_PATH:$NEW_TAG
    ```

7. Docker-in-Docker 設定

    `DOCKER_TLS_CERTDIR=''` 是為了讓 dind 關閉 TLS，讓 CI job 可以直接用明文 TCP 連線 docker daemon，避免 TLS 憑證錯誤。

    ```yaml
    services:
      - docker:dind
    variables:
      DOCKER_HOST: tcp://docker:2375/
      DOCKER_TLS_CERTDIR: ''
    ```

    [可以參考本文](https://about.gitlab.com/blog/docker-in-docker-with-docker-19-dot-03/)

8. 一些要注意的小問題

- **DOCKER_TLS_CERTDIR 不能不設定。**
- **tag 跳號或重複？**  
  請務必先刪除本地 tag 再 fetch remote tag。
- **無法 push tag？**  
  請確認 Deploy Token/PAT 權限，且 remote url 正確。
- **docker build 失敗，daemon 連不到？**  
  請檢查　gitlab runner 是否有 privileged mode，且 dind 有啟動。
  在`/etc/gitlab-runner/config.toml`中

  ```toml
  [[runners]]
    name = "docker-runner"
    executor = "docker"
    [runners.docker]
      privileged = true
      ...
  ```

## 小結

完整 .gitlab-ci.yml 範例如下，

```yaml
stages:
  - build
variables:
  PROJECT_NAME: my-proj
  GROUP_PATH: my-group
  REGISTRY_PATH: registry.gitlab.com/$GROUP_PATH/$PROJECT_NAME

build-image:
  image: docker:latest
  services:
    - docker:dind
  variables:
    DOCKER_HOST: tcp://docker:2375/
    DOCKER_TLS_CERTDIR: ''
  stage: build
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: always
    - when: never
  script:
    - git fetch --prune --tags
    - git tag -l | xargs -n 1 git tag -d
    - git fetch --tags
    - |
      prefix="ver."
      max_patch=$(git tag -l "${prefix}[0-9]*.[0-9]*.[0-9]*" | grep -oE '[0-9]+\.[0-9]+\.[0-9]+' | sort -t. -k1,1nr -k2,2nr -k3,3nr | head -n1 | awk -F. '{print $3}')
      if [ -z "$max_patch" ]; then
        new_tag="${prefix}1.0.0"
      else
        major=$(git tag -l "${prefix}[0-9]*.[0-9]*.[0-9]*" | grep -oE '[0-9]+\.[0-9]+\.[0-9]+' | sort -t. -k1,1nr -k2,2nr -k3,3nr | head -n1 | cut -d. -f1)
        minor=$(git tag -l "${prefix}[0-9]*.[0-9]*.[0-9]*" | grep -oE '[0-9]+\.[0-9]+\.[0-9]+' | sort -t. -k1,1nr -k2,2nr -k3,3nr | head -n1 | cut -d. -f2)
        patch=$((max_patch + 1))
        new_tag="${prefix}${major}.${minor}.${patch}"
      fi
      while git rev-parse "$new_tag" >/dev/null 2>&1; do
        patch=$((patch + 1))
        new_tag="${prefix}${major}.${minor}.${patch}"
      done
      echo "new tag is $new_tag"
      echo "NEW_TAG=$new_tag" >> build.env
    - source build.env
    - git config --global user.email "rd@example.com"
    - git config --global user.name "rag-cicd"
    - git remote set-url origin https://rag-cicd:$RAG_CICD_TOKEN@gitlab.com/$GITLAB_REPO.git
    - git tag "$NEW_TAG"
    - git push origin "$NEW_TAG"
    - docker login -u "$CI_REGISTRY_USER" -p "$CI_REGISTRY_PASSWORD" $CI_REGISTRY
    - docker build -t $REGISTRY_PATH:$NEW_TAG .
    - docker push $REGISTRY_PATH:$NEW_TAG
```

這樣設定後，每次 main 分支有 commit，CI/CD 就會自動產生新 tag、build 並推送對應版本的 Docker image 讓大家共用。

(fin)
