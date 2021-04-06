---
title: "[實作筆記] 怎麼建立一個網站？(五) - Hexo 的 404 頁面"
date: 2021/04/06 09:49:12 
tag:
  - 實作筆記
---

## What

什麼是 404 頁面呢 ?  
其實這是一個 HTTP 狀態碼，代表「網頁不存在」,  
與另外一種常見錯誤代碼(500/503)，代表的意義並不相同,  
500/503 通常是指服務整個掛了,  
而 404 是指所要的資源(頁面、檔案、圖片)並不存在.  

以下提供一些常見的狀態碼與代表意義

- 200 OK
- 403 Forbidden
- 500 Internal Server Error
- 503 Service Unavailable

## Why

為什麼我們需要一個 404 頁面呢 ? 
我直接引述
> 當使用者不小心進入你某些不存在或者有錯誤的頁面，就會跳出這個 404 頁面，(中略...)
> 而這個頁面最大的用途在於增加使用者體驗，例如畫面上會有 Logo or 返回首頁按鈕，確保使用者不會因為看到這個頁面立刻關閉。
> (中略...)
> 搜尋引擎也會依照你是否有這個當作一個加分評比 (中略...)
>

## How

首先執行 `hexo new page 404` , 這是 Hexo 用來建立新頁面的語法,  
上述的語法執行後, 會產生一個 404.md 檔, 如果你不喜歡這個檔名你也可以換掉,  
ex: `hexo new page page_not_found`

產生的頁面如下:

```text=

---
title: page_not_found
date: 2021-04-06 10:36:15
---

```

這個時候我們需要調整一個重要屬性 `permalink: /404.html`,  
這樣當 Github Page 找不到頁面時,  
才會出現我們設定的 404 頁面.  

另外我們可以設定 `layout:true` 的布林值來決定是否套用原本的樣式(預設為 true),  
這個主要也是為了讓使用者"感覺"他仍然在同一個網站之中,  
而減低跳轉率.  

## 參考

- [Creating a custom 404 page for your GitHub Pages site](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-custom-404-page-for-your-github-pages-site)
- [試著學 Hexo - SEO 篇 - 新增你的 404 頁面](https://hsiangfeng.github.io/hexo/20201006/174392200/)
- [RFC 7231](https://tools.ietf.org/html/rfc7231#section-6.5.4)

(fin)
