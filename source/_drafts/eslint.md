---
title: "[實作筆記] 建立團隊可用的 ESLint(一)"
date: 2023/03/12 18:10:41
tags:
  - 實作筆記
---

## 前言

在寫 JavaScript 的時候，我們經常會遇到語法錯誤、不一致的風格、以及其他潛在的問題。  
為了避免這些問題，我們可以使用 ESLint，一個在程式碼寫作期間檢查和分析錯誤的工具。  
在本篇文章中，我將介紹如何使用 ESLint，並且設定一個基本的風格指南，以便在開發過程中提高程式碼的品質。

### 以終為始

最終的目標是實現團隊代碼的一致性，並在此之上建立一定的品質，  
我的範例專案是一個 React 專案，並且使用 Typescript，  
所以程式碼的檢查要包含 React 與 Tsx;  
接下來要同步相關的 Eslint，讓任何取得專案的人，都必須遵守其規範。  
進一步的話，我希望可以使用大公司的基本設定，站在巨人的肩膀之上。  
最後，我要能實作客制化 Lint 規則，並整合至 CI/CD 之後。

## 實作流程

以下是在安裝和設定 ESLint 的基本步驟。

### 步驟 1：安裝 ESLint

首先，我們需要在本機端安裝 ESLint，這可以通過 npm（Node.js 的套件管理器）來進行。打開終端並執行以下命令：

```terminal
npm install eslint --save-dev
```

上述命令將 ESLint 安裝為開發依賴項。

### 步驟 2：初始化設定

安裝 ESLint 後，下一步是初始化設定。這可以使用以下命令進行：

```terminal
npx eslint --init
```

上述命令將提示我們回答一系列問題，以便根據我們的需求來生成設定檔。一些常見問題包括：

- 我們想使用 ESLint 來檢查什麼？
- 我們的專案使用的模塊是哪種類型？
- 我們使用哪個框架？
- 我們的代碼運行在哪裡？
  根據我們的答案，ESLint 將生成一個設定檔，以便在代碼檢查期間使用。

以下是我的設定:

> ❯ npx eslint --init  
> You can also run this command directly using 'npm init @eslint/config'.
>
> ✔ How would you like to use ESLint?  
> To check syntax only  
> To check syntax and find problems  
> ❯ To check syntax, find problems, and enforce code style
>
> ✔ What type of modules does your project use?  
> ❯ JavaScript modules (import/export)  
> CommonJS (require/exports)  
> None of these
>
> ✔ Which framework does your project use?  
> ❯ React  
> Vue.js  
> None of these
>
> ✔ Does your project use TypeScript?  
> Yes
>
> ✔ Where does your code run? (Press \<space\> to select, \<a\> to toggle all, \<i\> to invert selection)  
> ✔ Browser  
> ✔ Node
>
> ✔ How would you like to define a style for your project?  
> ❯ Use a popular style guide  
> Answer questions about your style
>
> ✔ Which style guide do you want to follow?  
> ❯ Standard: <https://github.com/standard/eslint-config-standard-with-typescript>  
> XO: <https://github.com/xojs/eslint-config-xo-typescript>
>
> ✔ What format do you want your config file to be in?  
> ❯ JavaScript  
> YAML  
> JSON
>
> Checking peerDependencies...
> ✔ Would you like to install them now? Yes

安裝完套件後，我們可以打開 eslintrc.js 檔案，並根據需要對其進行修改。我們可以根據自己的風格偏好，自訂不同的規則

### 步驟 3：設定自訂風格

安裝和初始化設定後，我們可以使用以下命令來安裝其他與我們選擇的風格指南相關的套件：

## 參考

- [eslint-config-airbnb-typescript](https://www.npmjs.com/package/eslint-config-airbnb-typescript)

## 備註

本文相關知識，節錄與 ChatGpt 的對話，不保証正確性。

(fin)
