---
title: "[踩雷筆記] Hexo 網站跑版"
date: 2023/06/02 16:02:33
tags:
  - CI/CD
  - 踩雷筆記
---

## 前情提要

我的部落格是基於 Hexo 建立的靜態網站，相關的有三個儲存庫：

![Gitlab Group](/images/2023/github_page_hexo-action.png)

### [Github Page](https://github.com/marsen/marsen.github.io)

這是部落格的主要內容，使用 Github Page 部署。  
通常情況下，我們不會直接對它進行修改，因為大部分的更動都是透過串接的 Github Action 自動完成的。  
這個儲存庫代表了整個流程的最終成果。

### [hexo action](https://github.com/marsen/hexo-action)

這是我們的自動化工具，結合 Github Action 可以實現持續集成與部署（CI/CD）。  
雖然平時不太需要特別注意，但它是相當重要的一環。

### [Source](https://github.com/marsen/Marsen.Node.Hexo)

這是我們撰寫文章的主要地方，也是整個部落格的功能和版面設計的編排處。

前幾天，我發佈了一篇新文章，但發現樣式出現了錯亂。

## 問題追蹤記錄

首先檢查了網頁載入的樣式，發現找不到 `style.css` 檔案，但有 `style.styl`。  
為了緊急處理，我先提供一個正確的 `style.css`。

幸好我在本地端能夠生成正確的 `style.css` 檔案，所以我先緊急上傳了一個版本到 [Github Page](https://github.com/marsen/marsen.github.io)。  
重新執行本地端的測試，一切正常。再次進行部署，結果樣式又跑掉了。  
於是我再次手動修正了 [Github Page](https://github.com/marsen/marsen.github.io) 的問題。

追查了一下 Github Action，發現在這個 commit 之後，異常情況開始發生：

```git
  -"hexo-renderer-stylus": "^2.1.0",
  +"hexo-renderer-stylus": "^3.0.0",
```

從這個記錄中可以得知，問題出在 `hexo-renderer-stylus` 的更新，而且是一個較大的版本變動。  
接下來我嘗試在本地端測試，但卻沒有發現問題。  
我開始思考是不是我的部署環境(CI/CD)的 Node 版本太舊了（12.x.x），所以我先更新了本機的 Node 版本到 20.x.x。
所以我修改了 [hexo action](https://github.com/marsen/hexo-action)，使用最新的 Node 20.x 版本的映像檔。  
然而，重新執行部落格的持續部署作業仍然失敗。

## 解決方式

在這個問題上，我卡住了很久，始終找不到原因。  
我決定再次追蹤持續部署的流程，結果發現我使用的 hexo-action 版本為 v1.0.5。  
原來我忽略了一個步驟，需要為 [hexo action](https://github.com/marsen/hexo-action) 添加新的標籤。  
此外，在我們的 [Source](https://github.com/marsen/Marsen.Node.Hexo) 的流程中，也需要修改為新的 action 標籤 uses: marsen/hexo-action@v1.0.6。

(fin)
