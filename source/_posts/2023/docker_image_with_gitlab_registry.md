---
title: "[實作筆記] 基於 Docker 的 CI Docker 映像檔加速構建流程"
date: 2023/06/01 17:05:57
tags:
  - 實作筆記
  - CI/CD
---

## 前情提要

在現代軟體開發的流程中，持續整合（Continuous Integration，簡稱 CI）是一個重要的概念，  
它可以幫助開發團隊在頻繁的程式碼變更中保持項目的穩定性和品質。  
而 Docker 作為一個輕量級的容器技術，在 CI 中也扮演著重要的角色。  
本篇文章將介紹一個基於 Docker 的 CI Docker 映像檔（Dockerfile_project）的實作記錄(手動，未來將調整為自動化)。

## 實作

首先，這個 CI Docker 映像檔是基於 php:8.x 映像檔建立的，並且預先安裝了 composer 和 ext-mongodb 擴展。  
這樣開發團隊在進行 PHP 項目的 CI 時，就可以直接使用這個映像檔，而不需要額外安裝這些依賴。

在開始使用之前，我們需要訪問 GitLab 的 Docker Registry，所以我們首先需要創建一個 Token。  
在 GitLab 的項目設置中，可以找到「Settings > Repository > Deploy tokens」，  
在這裡我們需要勾選「read_registry」和「write_registry」權限，以便於讀取和寫入映像檔。

接下來，我們需要使用以下指令來登錄到 GitLab 的 Docker Registry：

```shell
docker login $GITLAB_REGISTRY_URL -u $GITLAB_REGISTRY_USERNAME -p $GITLAB_REGISTRY_TOKEN
```

登錄成功後，我們就可以開始建立映像檔了。使用以下指令可以建立映像檔：

```shell
docker build -t registry.gitlab.com/my_group/subgroup/project .
```

如果你的電腦跟我一樣是 Mac M2 晶片，可以使用以下指令進行建立：

```shell
docker buildx build --platform linux/amd64 -t registry.gitlab.com/my_group/subgroup/project --load .
```

最後，我們需要將建立的映像檔推送到 GitLab 的 Docker Registry 中：

```shell
docker push registry.gitlab.com/my_group/subgroup/project
```

至此，我們已經完成了 CI Docker 映像檔的建立和推送的過程。  
現在開發團隊可以在持續整合的過程中使用這個映像檔，從而確保項目的穩定性和品質。

## 小結

在本文中，我們介紹了一個基於 Docker 的 CI Docker 映像檔（Dockerfile_project）的實作步驟。  
透過這個映像檔的建立和使用，開發團隊在持續整合（CI）過程中取得了顯著的效率提升。

使用這個新的 Docker 映像檔後，我們見證了從原本的 4 分鐘大幅改進至僅需 30 秒的驚人效果。  
這意味著開發團隊現在能夠更快速地完成整個 CI 流程，並且在更短的時間內獲得即時的反饋。

這種效率的提升主要歸功於 Docker 的輕量級容器技術，以及映像檔中預先安裝的依賴（如 composer 和 ext-mongodb）。  
透過這些優勢，開發團隊可以在相同的硬體資源下更有效地執行構建和測試操作，從而節省了寶貴的時間。

因此，使用這個新的 Docker 映像檔對於任何需要進行持續整合的專案來說都是一個明智的選擇。  
它不僅能提升開發團隊的效率，同時也能確保項目的穩定性和品質。

(fin)
