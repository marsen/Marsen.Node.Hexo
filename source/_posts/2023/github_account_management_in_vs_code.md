---
title: " [踩雷筆記] 解決 GitHub 帳戶在 VS Code 中的管理問題"
date: 2023/10/27 19:08:57
tags:
  - 踩雷筆記
---

## 前情提要

當我使用 VS Code 時，我需要同時存取多個 GitHub 帳戶，
自已的 Github - 主要存取 VS Code 一些地端的設定值
公司的 Github - 有免費的 Copilot 這很好用。
兩個都我需要，我要作出大人的選擇。

## 實務上的困境

我可以同時登入兩組帳戶，但是無法達到我要的需求

### 登入兩組帳戶的方法

1. 確保安裝了 Github Copilot 與 Github Pull Request and Issues 套件  
2. 如果有其它 Github 帳號已經在登入狀態的，都先登出，並重啟 VS Code  
3. 準備好兩個瀏覽器，分別登入公司與自已的 Github Account  
4. 點擊在左下的頭像 Icon (沒有頭像 Icon 的話在左側欄任一處點右鍵，將`帳戶`打勾)  
5. 選擇"使用 Github 登入，以使用 Github Copilot"  
6. 選擇登入公司帳戶的瀏覽器，此時會提示要開啟 VS Code，選擇開啟  
7. 回到 VS Code，重複第 4 步  
8. 選擇"使用 Github 登入，以使用 Github 提取要求和問題"  
9. 選擇登入自已帳戶的瀏覽器，此時會提示要開啟 VS Code，選擇開啟  

如果你的預設瀏覽器是登入公司 Github 的帳號，就複製網址，貼到登入自已的 Github 帳號瀏覽器，反之亦然。  
但是這個作法，實際上沒有什麼幫助，*備份與同步設定與 Copilot 會共用同一組帳號*。  
我並不需要 Github Pull Request and Issues 相關功能。  

### 手動組態同步

早期 VSCode 有人寫了組態同步的擴充套件，但是目前內建在 VSCode 內部後不再維護了。
但是我需要的就是自已長期以來的使用的設定，所以可以先登入自已的帳號。
再登出後，登入公司帳號。缺點是需要手動處理。

### 安裝第二個 VSCode

這個方法也有很多不同的實作方式。  
簡單說一個官方直接支援的方式。 VS Code Insider  
這算是一個搶先試用版的 VSCode，也有其對應的風險。  
但是你就可以在系統上安裝兩個 VSCode，並登入不同的帳戶。  

### 花錢為自已的帳戶買 Copilot

這或許才是最大人的解法。  
我們要的是"同步組態設定"與"Copilot"使用權。  
根帳號本身其實沒有什麼關係。  
資本主義大好，但就失去了一些童趣了。  

## 參考

-<https://tutorials.tinkink.net/zh-hant/vscode/copilot-usage-and-shortcut.html>
-<https://code.visualstudio.com/docs/?dv=osx&build=insiders>

(fin)
