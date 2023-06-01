---
title: "[實作筆記] Mac M2 建立 Docker Image"
date: 2023/06/01 16:33:41
tags:
  - CI/CD
---

## 前情提要

在我的 Gitlab-CI 當中有個步驟 --- 建立 Php Image --- 十分的冗贅，  
需要花３分鐘左右安裝 ext-mongo 與 composer，  
如果我可以找到已經安裝好的 image 版本就可以省下這３分鐘。
但是我找不到，那不如自已建一個 Image 吧

## 遇到的問題

在構建 GitLab CI 流程時，我遇到了一個問題。  
我在開發機上建立了 PHP 的 Docker Image，但在 Gitlab-CI Runner 運行 Image 時，  
出現了 _"standard_init_linux.go:219: exec user process caused: exec format error"_ 的錯誤。  
這個錯誤表示容器內部執行的可執行文件格式不正確或無效。

進一步分析後，我意識到問題可能是由於容器映像與主機操作系統的處理器架構不兼容引起的。  
由於我的筆電是基於 Apple M2 晶片的 Mac，它使用的是 ARM 架構，而我的 CI 使用的容器映像可能是針對 x86 架構設計的。

## 解決方法

Docker for Mac 提供了一個功能稱為「多架構建置（Buildx）」，它允許您在具有不同處理器架構的環境中建立 Docker image。
以下是在 Mac 上建立 x86 架構的 Docker image 的步驟：
確保您已安裝 Docker for Mac，並確保 Docker 客戶端已啟用 Buildx 功能。您可以使用以下命令來檢查是否啟用了 Buildx：

```shell
docker buildx version
```

如果 Buildx 功能已啟用，您應該能夠看到有關 Buildx 的相關訊息。

創建一個新的 Buildx builder，指定 x86 架構。您可以使用以下命令創建一個新的 builder：

```shell
docker buildx create --name mybuilder --use
docker buildx inspect --bootstrap
```

這將創建一個名為 "mybuilder" 的 Buildx builder，並將其設置為當前使用的 builder。

使用新建立的 builder 建立 Docker image。在 Dockerfile 所在的目錄中執行以下命令：

```shell
docker buildx build --platform linux/amd64 -t image-name:tag .
```

在這個命令中，--platform linux/amd64 指定要建立的架構為 x86 架構。

為了將結果映像推送到容器倉庫，您可以加上 --push 選項，例如：

```shell
docker buildx build --platform linux/amd64 -t image-name:tag --push .
```

這將在建置完成後將映像推送到指定的容器倉庫。

如果您只是想將映像載入到本地的 Docker 中，您可以使用 --load 選項，例如：

```shell
docker buildx build --platform linux/amd64 -t image-name:tag --load .
```

實際上使用 `--load` 在本地端建立的 image 也無法在本地端執行，因架構不同。

## 小結

使用了 Docker for Mac 的「多架構建置（Buildx）」功能，  
讓我在 M2 Mac 上也可以成功建立 x86 架構的 Docker image。
使用這個 image 讓我們可以省下 89% 的時間(3 分鐘 →20 秒)

(fin)
