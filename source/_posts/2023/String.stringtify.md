---
title: " [技術筆記] JSON.stringify 的參數"
date: 2023/01/29 21:30:12
---

## 簡介 JSON.stringify()

JSON.stringify() 是 Javascript 提供的一個方法，  
可以將 JavaScript 的值轉換成 JSON 字串(String)。

這方法需要 3 個參數

1. value
   這個參數是必填的，會被轉成 JSON 字串

   ```javascript
   var obj = { name: "John", age: 30, city: "New York" };
   var myJSON = JSON.stringify(obj);
   //myJSON is 「{ "name":"John", "age":30, "city":"New York"}」
   ```

2. replacer
   這個選擇性參數，會是一個陣列或是一個 function ，主要的目的可以過濾 value 的值。
   我們可以用 w3school 的[範例說明一下](https://www.w3schools.com/jsref/tryit.asp?filename=tryjson_stringify)
   如果我們傳入的是一個陣列，那它會像一個過濾器一樣，只篩選出陣列中的字串

   ```javascript
   var obj = { name: "John", age: 30, city: "New York" };
   var myJSON = JSON.stringify(obj, ["name", "city"]);
   //myJSON is 「{"name":"John","city":"New York"}」
   ```

   如果我們傳入的是一個 function，會將每一個參數丟入 function 之中(可以進行加工或裝飾)

   ```javascript
   var obj = { name: "John", age: 30, city: "New York" };
   var myJSON = JSON.stringify(obj, (key, value) => {
     if (key.match(/(age|city)/)) return "*";
     return value;
   });
   //myJSON is 「{"name":"John","age":"*","city":"*"}」
   ```

3. space 選擇性
   這個參數是為了可讀性

   ```javascript
   var obj = { name: "John", age: 30, city: "New York" };
   var myJSON = JSON.stringify(obj, null, 2);
   //myJSON is 「{ "name": "John", "age": 30, "city": "New York" }」
   ```

`JSON.stringify(obj, null, 2);`或`JSON.stringify(obj, null, 4);`便是開發者常用的呼叫方式

## Ref

- [What is null in JSON.stringify(obj,null,2)](https://akshaymattoo.medium.com/what-is-null-in-json-stringify-obj-null-2-8282b2e4eee1)
- [JSON.stringify()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)
- [JavaScript JSON stringify() Method](https://www.w3schools.com/jsref/jsref_stringify.asp)
  (fin)
