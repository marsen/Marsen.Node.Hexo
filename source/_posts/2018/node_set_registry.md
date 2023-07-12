---
title: " [實作筆記] 設定 NPM Registry"
date: 2018/07/29 16:21:22
tags:
  - Nodejs
  - GulpJs
  - 實作筆記
---

## 原因

最近換了新電腦，在家工作時發生了 npm install Error;  
錯誤訊息如下，明顯看到 npm 嘗試去連線 `http://npm.mycompany.io`  
<http://npm.mycompany.io> 是公司內部的私有網路;  
印象中沒有特別設定，可能是公司內的預設安裝設定的。

```shell=
PS D:\Repo\Marsen\Marsen.Node.Express> npm i -g gulp
npm ERR! code ETIMEDOUT
npm ERR! errno ETIMEDOUT
npm ERR! network request to http://npm.mycompany.io/gulp failed, reason: connect ETIMEDOUT ***.***.***.***:80
npm ERR! network This is a problem related to network connectivity.
npm ERR! network In most cases you are behind a proxy or have bad network settings.
npm ERR! network
npm ERR! network If you are behind a proxy, please make sure that the
npm ERR! network 'proxy' config is set properly.  See: 'npm help config'

npm ERR! A complete log of this run can be found in:
npm ERR!     C:\Users\Mark Lin\AppData\Roaming\npm-cache\_logs\2018-07-29T07_17_25_263Z-debug.log
```

## 解法

一個簡單的方法是 VPN 回公司內部網路;  
另一個方法是重新設定，

```shell=
> npm get registry
http://npm.mycompany.io
> npm set registry https://registry.npmjs.org
> npm get registry
https://registry.npmjs.org
```

(fin)
