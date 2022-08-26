---
title: " [實作筆記] 使用 IIS 作為 Reverse Proxy Server"
date: 2020/04/24 10:31:22
tag:
  - 實作筆記
---

## 情境

1. 作業系統為 Windows 10
2. IIS 的版本為 10.0.18362.1
3. 前端使用 nodejs 開發了一個網站，會在 localhost:3000 執行，提供 UI
4. 使用 dotnet core 開發了一個網站，在 IIS 上執行，用來提供 Api
5. 透過鎖 Host 的方式 dotnet core 的網站綁定在 `http://dev.api.test`

## 目標

1. 即使只有開發階段，我也不想看到 localhost:3000 作為我的網址
2. 我想看到 `http://dev.site.test` 作為我的站台

## 本文

首先，鎖 Host `127.0.0.1 dev.site.test`，
Host 的檔案路徑為 `C:\Windows\System32\drivers\etc`。

接下來請下載並安裝 [URL Rewrite](https://www.iis.net/downloads/microsoft/url-rewrite) 與 [Application Request Routing](https://www.iis.net/downloads/microsoft/application-request-routing)。

IIS 建立網站，繫結我設定為 `dev.site.test:80`，
應用程式集區我沒有特別處。
到 IIS 選取站台的 Url Rewrite 新增規則  
![到 IIS 選取站台的 Url Rewrite 新增規則](/images/2020/4/iis_reverse_proxy_01.jpg)  
選取 Reverse Proxy 規則

![選取 Reverse Proxy 規則](/images/2020/4/iis_reverse_proxy_02.jpg)

填寫 `localhost:3000`

![填寫 `localhost:3000`](/images/2020/4/iis_reverse_proxy_03.jpg)

這個時候前往 dev.site.test 就可以看到站台囉。

## 參考

- [URL Rewrite](https://www.iis.net/downloads/microsoft/url-rewrite)
- [Application Request Routing](https://www.iis.net/downloads/microsoft/application-request-routing)
- [Hosting a Node.js application on Windows with IIS as reverse proxy](https://dev.to/petereysermans/hosting-a-node-js-application-on-windows-with-iis-as-reverse-proxy-397b)
- [How to Setup Reverse Proxy on IIS with URL-Rewrite](https://tecadmin.net/set-up-reverse-proxy-using-iis/)
- [如何利用 IIS7 的 ARR 模組實做 Reverse Proxy 機制](https://blog.miniasp.com/post/2009/04/13/Using-ARR-to-implement-Reverse-Proxy)

(fin)
