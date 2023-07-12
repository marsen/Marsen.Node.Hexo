---
title: "[實作筆記] 不要用 Homebrew 安裝 nvm"
date: 2023/03/22 15:48:38
tags:
  - 實作筆記
---

## 前言

在開發 Node.js 應用程式時，我們可能需要在不同的 Node.js 版本之間進行切換。  
在這種情況下，一個方便的解決方法是使用 Node Version Manager（nvm）。  
而 Homebrew 則是一個流行的 macOS 套件管理工具。  
在這篇文章中，我們將探討為什麼官方不建議使用 homebrew 安裝 nvm。

## 情況

首先，值得注意的是，nvm 官方文件明確表示不建議使用 Homebrew 安裝 nvm。  
主要是因為 Homebrew 的安裝方式可能會干擾 nvm 的運作，導致一些奇怪的問題。  
nvm 官方建議使用其提供的安裝方式，以確保 nvm 能夠正常運作。

那麼，標準的 nvm 安裝方式是什麼呢？  
您可以在 nvm 的 GitHub 頁面上找到完整的安裝指南，這裡不再贅述。  
簡而言之，您需要在終端機中運行一個安裝腳本，該腳本將安裝 nvm，並更新您的 shell 啟動腳本以使其能夠正常運行。

如果您仍然希望使用 Homebrew 安裝 nvm，可以參考一些額外的資源。  
這裡[有一篇詳細的文章](https://collabnix.com/how-to-install-and-configure-nvm-on-mac-os/)，介紹了如何使用 Homebrew 安裝並配置 nvm。  
不過，需要注意的是，這種方法仍然不是官方建議的方式。

## 實作

不管您選擇哪種方式安裝 nvm，一旦完成後，您需要在終端機中設置 nvm 的環境變數才能正常運行。  
如果您希望在不關閉終端機窗口的情況下讓變數生效，可以使用以下命令：

```terminal
source ~/.bashrc
```

如果您使用的是 zsh，則需要使用以下命令：

```terminal
source ~/.zshrc
```

這將重新載入您的 shell 啟動檔案，使 nvm 環境變數生效。

參考：

- [NVM](https://github.com/nvm-sh/nvm#installing-and-updating)
- [How to install and Configure NVM on Mac OS](https://collabnix.com/how-to-install-and-configure-nvm-on-mac-os/)

(fin)
