---
title: " [實作筆記] Hexo Generate Error 處理"
date: 2022/07/22 11:23:17
tag:
  - 實作筆記
---

## 前情提要

我的 Blog 是透過 Hexo 這套系統建立出來的，
流程是：

1. 撰寫文章
2. 執行 `hexo g` 建立靜態檔
3. 如果想在本地看 Blog 的效果，可以用 `hexo s`
4. 執行 `hexo d` 部署到 Github

更多細節可以參考我以前寫得[相關文章](https://blog.marsen.me/2016/08/28/2016/how_to_use_github_page/)

## 問題

在上述的第 2 步驟，執行命令後，雖然可以產生靜態檔，  
但會伴隨著以下的錯誤訊息，這就非常擾人了。

```cmd
FATAL {
  err: [OperationalError: EPERM: operation not permitted, unlink '/Users/marklin/Repo/Marsen/Marsen.Node.Hexo/public/2022/03/18/2022/https_and_Brave_Browser'] {
    cause: [Error: EPERM: operation not permitted, unlink '/Users/marklin/Repo/Marsen/Marsen.Node.Hexo/public/2022/03/18/2022/https_and_Brave_Browser'] {
      errno: -1,
      code: 'EPERM',
      syscall: 'unlink',
      path: '/Users/marklin/Repo/Marsen/Marsen.Node.Hexo/public/2022/03/18/2022/https_and_Brave_Browser'
    },
    isOperational: true,
    errno: -1,
    code: 'EPERM',
    syscall: 'unlink',
    path: '/Users/marklin/Repo/Marsen/Marsen.Node.Hexo/public/2022/03/18/2022/https_and_Brave_Browser'
  }
} Something's wrong. Maybe you can find the solution here: %s https://hexo.io/docs/troubleshooting.html
```

這個錯誤訊息 `OperationalError: EPERM: operation not permitted, unlink…` .  
是一個非常籠統的錯誤訊息，來自作業系統的底層，中間經過 hexo 與 node 的流程，  
如果不深入鑽研(但是**我沒有要深入**)，我們難以知道錯誤的細節。

好在，我發現當刪除了 `public` 資料夾之後，  
再次執行 `hexo g` 就不會有錯誤訊息。

## 解決方法

簡單來說，我可以在每次 `hexo -g` 之前刪除 `public` 資料夾就可以了。  
以下可以用一行指令替代。

```cmd
rm -rf public | hexo -g
```

再進一步，我修改了 `alias` 指令如下

```cmd
alias hxg="rm -rf public | hexo g"
```

如此一來，每次我執行 `hxg` 的時，就會依上述步驟先刪除再建立 `public` 靜態資料。  
有關 alias 的設定，可以參考我之前寫的[文章](https://blog.marsen.me/2021/07/09/2021/alias_terminal/)

## 參考

- [【解决方法】hexo g 报错 OperationalError: EPERM: operation not permitted, unlink ...](https://blog.csdn.net/weixin_43871500/article/details/109163456)
- [errno(3) — Linux manual page](https://man7.org/linux/man-pages/man3/errno.3.html)

(fin)
