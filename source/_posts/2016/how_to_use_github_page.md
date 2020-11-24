---
title: "[實作筆記]怎麼建立一個網站？(二) - 簡單用github page 建立靜態網站"
date: 2016/08/28 4:56:27 
tag:
   - 實作筆記
   - Git
---

## 前置作業

- 你要有一個 GitHub 帳號

## 建立github page

如果不排斥看原文，可以直接[參考](https://pages.github.com/)  

1. 建立一個 Repository , 並且命名為 `username.github.io` , 這裡的 username 請使用你的 GitHub 帳號的 username.  
2. clone `username.github.io` 到你的本機上.

        ```shell
         git clone https://github.com/username/username.github.io
        ```

3. 建立一個靜態網頁 `index.html` , 隨便打點什麼。  

        ```html
        <!DOCTYPE html PUBLIC "-//IETF//DTD HTML 2.0//EN">
        <HTML>
                <HEAD>
                        <TITLE>
                                Hello world
                        </TITLE>
                </HEAD>
        <BODY>
                <H1>Hello world</H1>
                <P>This is my github page</P>
        </BODY>
        </HTML>
        ```

4. commit之後,push 到github上
        > git add --all
        > git commit -m "Initial commit"
        > git push -u origin master

5. 瀏覽  <http://username.github.io> 即完成

## 使用自訂的 Domain

1. 首先準備好一個domain ex: username.xyz
2. 需要在根目錄底下，放入一個 CName file
檔案的內容只需要你的domain即可  
ex:
        blog.username.xyz  
        username.xyz
3. 在Name Servers (例如[cloudflare](https://www.cloudflare.com/))上設定`CNAME`到github page,  
將`blog.username.xyz` 綁定到 `username.github.io`
ex:

| TYPE | NAME | VALUE | TTL |
|---|---|---|---|---|
| CName  | * | username.xyz | auto |
| CName  | blog | username.github.io | auto |
更多請參考「[購買網域到設定DNS](http://blog.marsen.me/2016/08/21/setting_DNS_with_google/)」.

## 透過Hexo部署

Hexo基本概念可以參考[官方中文文件](https://hexo.io/zh-tw/docs/index.html) .

1. 重點在於`_config.yml`的設定
        deploy:
          type: git
          repository: <https://github.com/username/username.github.io>
          branch: master

2. 執行`hexo d`進行部署，這個動作會將hexo 建立出來的靜態網站(html+css+javascript+圖片等…)部署到github page上。

## 系列文章

- [怎麼建立一個網站？(一)](https://blog.marsen.me/2016/08/21/setting_DNS_with_google/)
- [怎麼建立一個網站？(二)](https://blog.marsen.me/2016/08/28/how_to_use_github_page/)
- [怎麼建立一個網站？(三)](https://blog.marsen.me/2016/09/04/2016/http2_by_cloudflare/)
- [怎麼建立一個網站？(四)](https://blog.marsen.me/2020/10/22/2020/google_domain_forward_mail/)

(fin)
