---
title: HTTP POST 的 Body 有四種
tag:
  - nodejs
  - expressjs
  - post
  - http
---
## 前情提要

學習用 expressjs 開發網站的過程中取得為了 post 參數資料 ,  
必需使用下列語法
```javascript
var express = require('express');
var router = express.Router();
//// 中略
router.post('/', function(req, res, next) {
  
  res.send("post data is : " + req.body.param_name);
});
```
### 我的環境
- Express 4.x 
- 測試工具 Postman


## 使用Postman測試

### `application/x-www-form-urlencoded`
常用的POST方法,簡單說就是KEY VALUE
### `multipart/form-data`
上傳檔案會用這個
### `raw` 
- application/json
目前主流的API會用輕巧的JSON作為傳遞資訊的媒介
- text/xml / application/xml
古老但標準的Web服務通常會透過xml作為交換資訊的媒介

### `binary`



## 參考
- https://imququ.com/post/four-ways-to-post-data-in-http.html
- https://codeforgeek.com/2014/09/handle-get-post-request-express-4/
- https://scotch.io/tutorials/use-expressjs-to-get-url-and-post-parameters
- http://stackoverflow.com/questions/4832357/whats-the-difference-between-text-xml-vs-application-xml-for-webservice-respons?ref=mythemeco&t=pack
(fin)