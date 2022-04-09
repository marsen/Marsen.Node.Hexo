---
title: "[實作筆記] 升級現有基於 TypeScript 的 React 17 專案到 React 18"
date: 2022/04/08 02:20:08
---

## 前情提要

升級 React 在官方的文件之中，似乎是一件極為簡單的事，  
但是如果你的專案是基於 TypesScript 開發，  
在前幾周應該無法順利升級，主要的原因是相對應的 @types 套件並未同時更新，  
幸運的是，現在看到文章的你，已經可以順利更新了，請參考以下的步驟。

## 記錄

安裝 react、react-dom、@types/react、＠types/react-dom

```shell
npm i react@18 react-dom@18 @types/react@18 @types/react-dom@18
```

更改 `ReactDOM.render` 的寫法

原本的寫法

```tsx
import React from "react";
import ReactDOM from "react-dom";

ReactDOM.render(<App />, document.getElementById("root"));
```

後來的寫法

```tsx
import React from "react";
import ReactDOM from "react-dom/client";

const ele = document.getElementById("root") as Element;
const root = ReactDOM.createRoot(ele);
root.render(<App />);
```

特別注意要處理 `Element` 的型別

## 參考

- [https://www.youtube.com/watch?v=N0DhCV\_-Qbg](https://www.youtube.com/watch?v=N0DhCV_-Qbg)
- [How to Upgrade to React 18](https://reactjs.org/blog/2022/03/08/react-18-upgrade-guide.html)

(fin)
