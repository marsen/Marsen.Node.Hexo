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


## 2018/12/02 補充

追加讓 PowerShell 在 cmder 裡面也美美的方法

### 步驟
1. 下載[cmder-powershell-powerlin-prompt](https://github.com/AmrEldib/cmder-powershell-powerline-prompt)，選擇 Clone or download > Download ZIP
2. 解壓縮 ZIP 檔
3. 開啟 Cmder 所在的資料夾，如下圖 > 開啟 config 資料夾
![](/images/2018/colorfull_cmder/cmder_folder.jpg)
4. 將壓縮檔內的 `user_profile.ps1`，取代 config 內的 `user_profile.ps1`
5. 將壓縮檔內的 `profile.d` 資料內的所有檔案，全數貼到 config 內的 `profile.d` 資料夾內，如果不存在就建立一個。
6. 開啟 config\profile.d 資料夾，重新命名 goToFolder.config.example 為 goToFolder.config 
    - 這個檔案內會設定一些目錄與 alias 。
    - 使用方法，在使用 cmder 開啟 powershell 的情況下，輸入 g + `alias` 
    - 比如說在 goToFolder.config 中，設定一組 `m, C:\User\Marsen`
    - 在 cmder 輸入 `g m` 就會自動切到 `C:\User\Marsen` 的路徑底下
7. 還有很多 Alias 請自行研究。
8. 重啟 cmder 後就會載入新的設定，變成美美的 powershell 了

## 參考
- 整篇都是參考[cmder-powerline-prompt](https://github.com/AmrEldib/cmder-powerline-prompt)的作法，原始 Repo 可能隨時會更新異動，不保証有效
- [cmder-powershell-powerlin-prompt](https://github.com/AmrEldib/cmder-powershell-powerline-prompt)

(fin)