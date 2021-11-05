---
title: "[踩雷筆記] 針對遺留代碼加入單元測試的藝術 "
date: 2021/11/05 11:31:07
---

## 問題

當執行以下語法時，

```shell
docker pull mysql 
```

會得到錯誤訊息如下

no matching manifest for linux/arm64/v8 in the manifest list entries

## 解決方案

```shell
docker pull --platform linux/amd64 mysql
```

## 參考

- <https://github.com/docker-library/mysql/issues/778>

(fin)
