---
title: "[踩雷筆記] 誰改了我的 package-lock.json ?"
date: 2018/07/30 15:53:21
tags:
  - Nodejs
  - 踩雷筆記
---

## 環境

OS Windows 10
npm v6.2.0
node v8.11.1

## 問題

承上篇[[實作筆記] 設定 NPM Registry](https://blog.marsen.me/2018/07/29/2018/node_set_registry/)，  
後續發生的一些問題，  
每當我執行 `npm install` 的時候都會異動到 `package-lock.json` 檔;  
異動這個檔案並不意外，可以參考[官方文件](https://docs.npmjs.com/files/package-lock.json)的說法。

奇怪的點是，異動竟然包含 Registry !?  
全部都被置換成公司的 Domain , WTF ?

![公司的 Domain ](https://i.imgur.com/KwwUqPV.jpg)

## 原因與坑

當執行 `npm install` 的時候，  
node 會去檢查你實際安裝的 node_modules，  
並產生 `package-lock.json` 檔。  
可以把 `package-lock.json` 想像成 node_modules 的映射。  
而恰巧之前所安裝過的檔案，是在 Registry 為 `npm.mycompany` 時下載的;
這導致產生 `package-lock.json` 的時候會映射出 `npm.mycompany` 的資訊。

小心有坑!!  
即使刪除了 node_modules 再重新 `npm install`，  
仍然會產生錯誤的 `package-lock.json`;  
這個原因是 npm 會有 `npm-cache`;  
這個資料夾位在 `%Users%\AppData\Roaming`。

## 解決方法

1. 設定 npm registry
   > npm set registry <https://registry.npmjs.org>
2. 刪除專案底下的 node_modules 資料夾
3. 刪除 npm-cache
4. 重新執行 `npm install`

(fin)
