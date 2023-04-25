---
title: "[AI 共筆] tsconfig 中的 tsx 設定"
date: 2023/03/12 20:40:10
---

## 前言

本篇是 AI Knows 的第一篇，ChatGPT 問世後，  
在學習開發上，我認為又到了一個新的領域，  
很多人開始作分享對 ChatGPT 的看法，在我看來還是社群時代的蹭流量作法。
更進一步的有人開始透過 AI 實作商業應用，商人果然是跑最快的。
另外一種，用嘴涮夢、涮 AI，開始賣書賣課的，我認為接近灰色地帶，就不予置評了。

## 問題

我正在替專案配置 EsLint 文件，過程當然直接請 AI 代勞，  
但是有一些設置我不那麼清楚也懶得查文件。
就一樣也請教 AI 囉。

## 對話記錄

### Q:tsconfig.json 中的 jsx 屬性為何？**

> jsx 屬性是用來指定 TypeScript 如何處理 JSX 語法的選項。  
> JSX 是一種 JavaScript 語法擴展，允許在 JavaScript 代碼中直接使用  
> HTML 標籤來描述 UI 組件，使得開發 Web 應用程式更為方便。
>
> 在 tsconfig.json 中，jsx 屬性可以設置為以下值：
>
> "react"：表示使用 React 的 JSX 語法。
> "preserve"：表示將 JSX 語法保留為原始的 JSX 代碼。
> "react-native"：表示使用 React Native 的 JSX 語法。  
> 當設置為 "react" 時，TypeScript 會將 JSX 轉換為對應的 JavaScript 代碼，  
> 這樣就可以在瀏覽器中運行。而當設置為 "preserve" 時，TypeScript 不會對  
> JSX 進行任何轉換，而是將其保留為原始的 JSX 代碼。

### Q:為什麼我看到的設定是 "jsx":"react-jsx"

> 在 TypeScript 4.1 之前，jsx 選項僅有兩個可選值，分別是 preserve 與 react。  
> 其中 preserve 會保留 JSX 並將其轉換為對應的 React.createElement 函式，  
> 而 react 則是直接轉換為對應的 JavaScript 代碼。
>  
> 在 TypeScript 4.1 中，又新增了一個選項 react-jsx，用來代表 TypeScript  
> 將會使用 @babel/plugin-transform-react-jsx 進行轉換，以此來與 Babel 兼容。
>
> 因此，如果你的 TypeScript 版本較新，通常會看到的 jsx 設定是 "jsx": "react-jsx"，  
> 表示 TypeScript 將會與 Babel 兼容。

## 小結

綜合以上問答，我們可以得知 JSX 是一種 JavaScript 語法擴展，讓開發 Web 應用程式更加方便，  
而 tsconfig.json 中的 jsx 屬性用來指定 TypeScript 如何處理 JSX 語法的選項，  
可以設置為 react、preserve 或 react-native 三種值，分別代表將 JSX 轉換為對應的 JavaScript 代碼、保留原始的 JSX 代碼或使用 React Native 的 JSX 語法。  

而在 TypeScript 4.1 中，新增了 react-jsx 選項，用來與 Babel 兼容。  
因此，如果 TypeScript 版本較新，通常會看到的 jsx 設定是 "jsx": "react-jsx"。  

## 寫在最後

通篇八成為 AI 牙慧，正確性未考究。

(fin)
