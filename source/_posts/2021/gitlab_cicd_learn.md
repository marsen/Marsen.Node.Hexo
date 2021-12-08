---
title: "[實作筆記] "
date: 2021/12/03 10:42:10
tag:
  - 實作筆記
---

## 前情提要

我想完成什麼 ?  
如何用最佳解完成全端應用的開發 ?

1. 前端應包含 Web、App(iOS、Android 或簡稱三螢) 與其它
2. 後端不限語言、框架(.Net Core、nodejs)
3. 測試部署應該自動化(CI/CD)
4. 成本應優化
5. 應該使用雲原生的技術與 know how

這次我們專注在第 3 點的 CI/CD 上，為了第 1 點，我建置的專案為 flutter  
選用的 CI/CD Server 為 Gitlab , 我將會在這裡作較多的著墨，  
最終的產出物是 flutter app image, 並推送到 image registry 上 。

![概觀，每個多邊型都應該可以被置換](https://i.imgur.com/rdrDJ9g.png)

如上圖，每個多邊型都應該可以被置換，  

- Application 可以被換成 Nodejs/.Net Core/ ...
- Gitlab Runner 有三種，本文中會使用 specific runners ，另外有 2 種
  - Shared runners(需要信用卡認証身分，每月限制 400 分鐘)
  - Group runners (在 Gitlab 上可以)
  我將 Gitlab 設定在本機 ( MacBook Pro ) 環境上，下文會講解細節，  
  也可以置換到雲端的伺服器上，搭配 K8S
  ex:
    - GKE: Google Cloud Platform Kubernetes Engine
    - EKS: Amazon Web Services’ Elastic Kubernetes Services
    - AKS: Microsoft Azure Kubernetes Service
- Container Registry 這裡我使用最主流的 Docker Hub Registry
   以雲原生的三大平台都有對應的功能，如果有機會應該優先選用。

## 工具準備

首先需要 [Gitlab](https://gitlab.com/) Account ，我們選用 Gitlab 作為 CI/CD Server,  
這個階段我們只會作到持續整合，而未部署，相當於只有 CI 的部份。
理論上也可以選用其它的 CI/CD 服務，像是 Azure 或是 GitHub , 這裡就不作過多的展開。

Docker , 簡單說我們的工作只有兩個步驟

1. 在 Gitlab 你的專案 > Settings > CI/CD > Runners 註冊 [Runner Executors](https://docs.gitlab.com/runner/executors/)
   - 一般來說，我們會選用 docker 或 Kubernetes，本章我會用 docker 為例
2. 撰寫 Pipeline 與 Job 腳本，
   ![Pipeline](https://i.imgur.com/mFACovE.png)
   我們預計執行以下的工作(Pipeline)
   - Test
     - Run Test
     - Run Lint
   - Build
     - Build Application
     - Build Image

## 開始

### [註冊 Gitlab Runner](https://docs.gitlab.com/runner/register/)

我們要在 docker 建立來執行 gitlab Runner

1. Create the Docker volume:

   ```bash
   docker volume create gitlab-runner-config
   ```

2. Start the GitLab Runner container using the volume we just created:

   ```bash
   docker run -d --name gitlab-runner --restart always \
       -v /var/run/docker.sock:/var/run/docker.sock \
       -v gitlab-runner-config:/etc/gitlab-runner \
       gitlab/gitlab-runner:latest
   ```

3. Register Docker Runner

   依序輸入 Gitlab URL、gitlab-ci 的 token、runner 說明/名稱、Runner 的 tag、executor、docker image

   ```bash
   docker run --rm -it -v gitlab-runner-config:/etc/gitlab-runner \
   gitlab/gitlab-runner:latest register
   ```

   - Enter your GitLab instance URL (also known as the gitlab-ci coordinator URL).
   - Enter the token you obtained to register the runner.
   - Enter a description for the runner. You can change this value later in the GitLab user interface.
   - Enter the tags associated with the runner, separated by commas. You can change this value later in the GitLab user interface.
   - Provide the runner executor. For most use cases, enter docker.
   - If you entered docker as your executor, you’ll be asked for the default image to be used for projects that do not define one in .gitlab-ci.yml.
GitLab instance URL 是 <https://gitlab.com/>
你可以在專案中的 Settings > CI/CD  找到 token，  
description 會顯示在 Runner List 中，可以用易懂的描述，  
tags 可以更多的參考這[本篇文章設定](https://docs.gitlab.com/ee/ci/runners/configure_runners.html#use-tags-to-control-which-jobs-a-runner-can-run) 
executor 選用 docker 記得需要安裝 docker daemon  
executor 為 docker 時，需要註明預設的 image，我是選用 `docker:stable`

### 執行 RUNNER

> gitlab-runner run

### 移除 RUNNER(為了重新安裝)

1. 驗証 runner 狀態是否 alive

   > gitlab-runner verify --delete

2. 移除 gitlab-runner

   > gitlab-runner unregister --all-runners

[參考](https://blog.csdn.net/edgar_t/article/details/113980215)

### 錯誤: 當使用 MacBook M1 當作 RUNNER

在 RUNNER JOB 中執行 flutter pub get 時
`$ flutter pub get`
發生錯誤訊息如下，當我換了一個 intel 晶片的 MacBook 當作 RUNNER 就不會有問題了。

```shell
Running "flutter pub get" in rettulf...
   ===== CRASH =====
   si_signo=Trace/breakpoint trap(5), si_code=128, si_addr=(nil)
   version=2.14.4 (stable) (Wed Oct 13 11:11:32 2021 +0200) on "linux_x64"
   pid=57, thread=116, isolate_group=main(0x4002e9fc00), isolate=main(0x4002e7d000)
   isolate_instructions=4001cf2ec0, vm_instructions=4001cf2ec0
   pc 0x0000ffffab7e28ec fp 0x000000400a1af838 Unknown symbol
   pc 0x0000ffffab49ed0f fp 0x000000400a1af898 Unknown symbol
   pc 0x0000ffffab78114d fp 0x000000400a1af8d8 Unknown symbol
   pc 0x0000ffffacaa294c fp 0x000000400a1af940 Unknown symbol
   pc 0x0000ffffacaa220b fp 0x000000400a1af980 Unknown symbol
   pc 0x0000ffffacd028ff fp 0x000000400a1af9f8 Unknown symbol
   pc 0x0000004001e6a123 fp 0x000000400a1afaa0 dart::DartEntry::InvokeCode(dart::Code const&, unsigned long, dart::Array const&, dart::Array const&, dart::Thread*)+0x153
   pc 0x0000004001e69f75 fp 0x000000400a1afb00 dart::DartEntry::InvokeFunction(dart::Function const&, dart::Array const&, dart::Array const&, unsigned long)+0x165
   pc 0x0000004001e6c55d fp 0x000000400a1afb50 dart::DartLibraryCalls::HandleMessage(dart::Object const&, dart::Instance const&)+0x15d
   pc 0x0000004001e93e46 fp 0x000000400a1afc30 dart::IsolateMessageHandler::HandleMessage(std::__2::unique_ptr<dart::Message, std::__2::default_delete<dart::Message> >)+0x596
   pc 0x0000004001ebe7ac fp 0x000000400a1afca0 dart::MessageHandler::HandleMessages(dart::MonitorLocker*, bool, bool)+0x14c
   pc 0x0000004001ebeecf fp 0x000000400a1afd00 dart::MessageHandler::TaskCallback()+0x1df
   pc 0x0000004001fdde48 fp 0x000000400a1afd80 dart::ThreadPool::WorkerLoop(dart::ThreadPool::Worker*)+0x148
   pc 0x0000004001fde27c fp 0x000000400a1afdb0 dart::ThreadPool::Worker::Main(unsigned long)+0x5c
   pc 0x0000004001f57a48 fp 0x000000400a1afe70 /sdks/flutter/bin/cache/dart-sdk/bin/dart+0x1f57a48
   -- End of DumpStackTrace
   qemu: uncaught target signal 6 (Aborted) - core dumped
   /sdks/flutter/bin/internal/shared.sh: line 225:    57 Aborted                 "$DART" --disable-dart-dev --packages="$FLUTTER_TOOLS_DIR/.packages" $FLUTTER_TOOL_ARGS "$SNAPSHOT_PATH" "$@"
   Cleaning up project directory and file based variables
```

## DIND(Docker In Docker)

我們的 Gitlab-Runner 是執行在 Docker 上，
而當我需要 build Docker Images 時，我會在 Container 中建立 docker  

### 如何在 DIND 中 Docker Login

參考以下的 `.gitlab-ci.yml`

```shell
   before_script:
      - echo "$DOCKER_REGISTRY_PASS" | docker login -u $DOCKER_REGISTRY_USER --password-stdin
```

### 無法對 docker hub registry  

在 RUNNER JOB 中執行 `docker push` 時  
錯誤訊息如下:  

`dial tcp: lookup docker on x.x.x.x:53: no such host error runner inside docker on armhf`  

這裡要修改 RUNNER 中的設定檔 `config.toml` , 路徑參考如下:  

>You can find the config.toml file in:
>
> - /etc/gitlab-runner/ on *nix systems when GitLab Runner is executed as root (this is also the path for service configuration)
> - ~/.gitlab-runner/ on*nix systems when GitLab Runner is executed as non-root
> - ./ on other systems

在  [[runners]] > [runner.docker] 加入 `image = docker:stable` 或是 `privileged = true`  
![just add image = docker:stable  and privileged = true](https://i.imgur.com/BBl9oxo.png)  
這可能不是一個正確的 solution , 可以更多的參考這篇[討論](https://gitlab.com/gitlab-org/gitlab-runner/-/issues/4794)  

### 在 DIND 編輯文件

如上題，我們需要在 container 之中編輯文件，  
這裡我會使用 vim，  
不過安裝前記得先更新 apt-get  
不然會出現以下錯誤  

> E: Unable to locate package vim on Debian jessie simplified Docker container

```shell
apt-get update
apt-get install apt-file
apt-file update
apt-get install vim     # now finally this will work !!!
```

### 問題

1. gitlab-ci-multi-runner 與 gitlab-runner 的差異為何 ?
2. 當 docker gitlab-runner image 的 instance 執行 gitlab-runner run 會產生以下訊息
Configuration loaded                                builds=0
listen_address not defined, metrics & debug endpoints disabled  builds=0
[session_server].listen_address not defined, session endpoints disabled  builds=0

(fin)
