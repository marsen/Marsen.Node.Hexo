---
title: " [實作筆記] 錯誤處理 the gcp auth plugin is deprecated "
date: 2022/06/08 15:53:51
tags:
  - 實作筆記
---

## 前情提要

環境 Mac OS，  
安裝了
Docker daemon 版本 4.8.2(79419)
gcloud
版本資訊如下

```shell
gcloud --version
Google Cloud SDK 389.0.0
beta 2022.06.03
bq 2.0.74
core 2022.06.03
```

執行以下指令時

> ❯ kubectl -n qa get secret deposit-secret -o json

會出現以下的警告訊息

```shell
W0608 15:50:50.291340   13396 gcp.go:120] WARNING: the gcp auth plugin is deprecated in v1.22+, unavailable in v1.25+; use gcloud instead.
To learn more, consult https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke
```

## 原因

在安裝 Docker 時，會同時安裝上 `kubectl`，
我們可以用以下的指令作查詢，

```shell
❯ where kubectl
/usr/local/bin/kubectl
```

這個時候我們可以看看[這篇](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke),  
簡而言之，就是在 kubectl v1.25 版本後，  
對授權的處理作了一點變化，改用 plugin 處理。
所以當使用舊版本( < v1.25 版)時會出示警告

## 解法

兩個步驟
移除原有的 kubectl 並透過 gcloud 安裝新版本的 kubectl

### 移除 kubectl 的 link

```shell
sudo rm kubectl
```

這是在安裝 Docker 時產生的。
如果你想查看 link 的資訊，可以在移除前用以下指令切換到系統目錄

> ❯ cd /usr/local/bin

再透過 `ll` 檢視細節，大概的回應如下

```shell
❯ ll
total 412936
...略
lrwxr-xr-x  1 root  wheel    55B  6  8 15:45 kubectl -> /Applications/Docker.app/Contents/
...略
```

## gcloud 安裝 kubectl

```shell
gcloud components install kubectl
```

出現以下訊息再按 y

```shell
Your current Google Cloud CLI version is: 389.0.0
Installing components from version: 389.0.0

┌──────────────────────────────────────────────────────────────────┐
│               These components will be installed.                │
├─────────────────────┬─────────────────────┬──────────────────────┤
│         Name        │       Version       │         Size         │
├─────────────────────┼─────────────────────┼──────────────────────┤
│ kubectl             │              1.22.9 │             65.2 MiB │
│ kubectl             │              1.22.9 │              < 1 MiB │
└─────────────────────┴─────────────────────┴──────────────────────┘

For the latest full release notes, please visit:
  https://cloud.google.com/sdk/release_notes

Do you want to continue (Y/n)?  y
```

再次執行 kubectl 相關的指令就不會噴錯了

## 參考

- [There are older versions of Google Cloud Platform tools: Docker](https://stackoverflow.com/questions/53597361/there-are-older-versions-of-google-cloud-platform-tools-docker)
- [Here's what to know about changes to kubectl authentication coming in GKE v1.25](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)

(fin)
