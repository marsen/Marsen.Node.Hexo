---
title: "如何讓 windows 也有美美命令提示視窗"
date: 2018/11/25 17:58:59
---

## 目的
![就是要美美命令提示視窗](/images/2018/colorfull_cmder/result.jpg)
常常在一些社群看到，大神都超會下 command，  
開始學習使用各種 command 之後，才發現那個美美的 terminal 不只是華麗而已，   
實際上也可以加速閱讀，而且潮指數也是怒加一波(畫錯重點)，  
當然要研究一下如何讓自已擁有一個賞心悅目的 commander 囉

## 作法
使用 [Cmder](http://cmder.net/) ， 這是一個 Windows 的開發人員常用的 terminal 介面，  
他可以執行一般的 CMD、Bash 與 PowerShell ，別想得太複雜，就是一個命令提示視窗。  

### 安裝字型 on Windows 10 

1. 下載 [powerline](https://github.com/powerline/fonts)，選擇 Clone or download > Download ZIP
2. 解壓縮後，選擇使用 AnonymousPro 字型(主要是要那些git icon的圖案，如果別的字型有 也可以使用)
3. 控制台>字型 把 ttf 拖進去安裝

### 設定 Cmder

1. Win + Alt + P 開啟設定畫面
2. Cmder > Settings > General > Fonts  > 下拉選單選 Anonymice PowerlineCmder 
![](/images/2018/colorfull_cmder/settings.jpg)

### 使用 lua Config
1. 下載[cmder-powerline-prompt](https://github.com/AmrEldib/cmder-powerline-prompt)，選擇 Clone or download > Download ZIP
2. 開啟 Cmder 安裝檔所在位置，找到 `Config` 資料夾  
3. 將下載的所有 *.lua 檔放入 `Config` 資料夾  

最後重啟 Cmder 就可以有一個美美的命令提示視窗了。  
[lua](https://www.lua.org/) 也是一個程式語言，你大可開啟文字編輯器，看一下裡面作了什麼。  


## 參考
- 整篇都是參考[cmder-powerline-prompt](https://github.com/AmrEldib/cmder-powerline-prompt)的作法，原始 Repo 可能隨時會更新異動，不保証有效

(fin)