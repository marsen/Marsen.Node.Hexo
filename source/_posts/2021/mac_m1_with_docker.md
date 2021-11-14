---
title: "[踩雷筆記] MacBook M1 silicon X Docker X MySQL Image"
date: 2021/11/05 11:31:07
---

## 前情提要

新工作配發了新的 MacBook 搭載最新的 M1 晶片,  
沒想到反而遇到了許多問題, 不會有太多的文字說明,  
僅用來記錄我實作有效的解決方法

### 情境:在 Docker 跑 MySQL

當執行以下語法時，

```shell
docker pull mysql 
```

會得到錯誤訊息如下

no matching manifest for linux/arm64/v8 in the manifest list entries

### 解決方案

```shell
docker pull --platform linux/amd64 mysql
```

## 參考

- <https://github.com/docker-library/mysql/issues/778>

(fin)
