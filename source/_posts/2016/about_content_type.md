---
title: 有關 HTTP Header Content-Type
date: 2016/10/28 12:54:55
tags:
  - post
  - HTTP
  - html
---

## 引言

當我們對 Server 發出 request 的時候  
需要註明你 request 的 Content-Type  
以下簡單介紹一下這些格式

### 測試工具

[PostMan](https://www.getpostman.com/)這套工具
可以模擬不同的格式資料發動 request 到你的 Server

---

## Content-Type

### `application/x-www-form-urlencoded`

常用的 Content-Type，簡單說就是 KEY-VALUE 的方式  
如下， KEY firstname 的值是 marsen  
lastname 是由使用者輸入

```html
<form>
  First name:<br />
  <input type="text" name="firstname" value="marsen" /><br />
  Last name:<br />
  <input type="text" name="lastname" />
</form>
```

同時資料會作一次 url encoded，  
產生類似下列的資料
firstname=marsen&lastname=lin&key%5b1%5d=value%5b1%5d

---

### `multipart/form-data`

PostMan 中的選項 `binary` 其實就是包成這種格式  
上傳檔案會使用這種 Content-Type，  
這通常表示你的 html element 包含有 `<input type="file">`

---

### 其它

PostMan 中的選項 `raw`，可以用字串組合成任意 Content-Type，  
參考[Content-Type Table](http://www.freeformatter.com/mime-types-list.html)

- `application/json`
  目前主流的 API 會用輕巧的 JSON 作為傳遞資訊的媒介
- `text/xml`、`application/xml`  
  早期標準的 Web 服務通常會透過 xml 作為交換資訊的媒介
- `text/plain`
  有些 email 或 debug 的情況會使用 text/plain 作為 Content-Type,但是一般的 Request 情況不建議使用
- 更多請參考[Spec](https://www.w3.org/TR/html5/forms.html#text/plain-encoding-algorithm)

---

## 參考

- <https://imququ.com/post/four-ways-to-post-data-in-http.html>
- <https://codeforgeek.com/2014/09/handle-get-post-request-express-4/>
- <https://scotch.io/tutorials/use-expressjs-to-get-url-and-post-parameters>
- <https://stackoverflow.com/questions/4832357/whats-the-difference-between-text-xml-vs-application-xml-for-webservice-respons?ref=mythemeco&t=pack>
- <https://stackoverflow.com/questions/4526273/what-does-enctype-multipart-form-data-mean>

(fin)
