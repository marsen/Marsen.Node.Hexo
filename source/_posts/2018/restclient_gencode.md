---
title: " [實作筆記] 用 Restclient GenCode"
date: 2018/12/30 11:58:43
tags:
  - 實作筆記
---

[REST Client](https://github.com/Huachao/vscode-restclient) 是一套 Visual Studio Code 的套件。
可以讓你不離開編輯器(Visual Studio Code)的情況下，發送一些 Request。
使用方法可以參考最後的聯結，本文僅介紹 Code Gen 的功能。

## 作法

### Step 1

取得 Request 連結資訊，這個部份可以直接從瀏覽器取得

大致上如下:

```text
GET https://***.****.com.my/Invoice/DownloadMyInvoicePdf?shopId=44&tradesOrderGroupCode=MG180929L00002
Cookie: ai_user=zazJ/|2018-04-11T10:01:21.161Z; _ga=GA1.3.1260482009.1526374223; __zlcmid=p9hkxHsXzC5Ka5; _ym_uid=15450340861049673384; _ym_d=1545034086; _gid=GA1.3.233068902.1545616884; _fbp=fb.2.1545992230107.2040829768; .AspNet.ApplicationCookie=1ly--ntvPSpcJIIDvk2PBFfuyy744Wvwx4ezT0c0BEl1t4Vw3ahOOMwSczzBezkE0dIPWBwQt12KPN8IrFj8eV2ZgfO6HYKuiw7cUNgS37Gr1FOH28o-l5EZuOYGd4uuqRduaBBPbZrJop5nUso1oPS4fOs-mFO0I17QWHtfB_BI2Lzd6rQQRl6_IevI_EPbsh0EDahqvR4wvF2QFMH2ycECSzR4pgEmm3hcQRJ9COWoc-DtZjxxa11yfghghmReLbe1cYtx1G7ST9zakc_qGmnMV0IyZoRFqEmHrzEb1b8fDO35UkxsiP2_mzGY-Oy4e3fV12Q0N7eGVblEkJYZkrtXADP8h9iGToPwAUI5rbnX2o32Z0_4zbg7x_GSF_HWsW22SWlkRCAZEKhvhEB9Qk56JPSRSJwqmpGDzm8a807-6lRP-JPOo_F43eYLgztH6k6imlUseUyDyYBTwJeIgF5gdyLKMUSivKs-SOivyPofDiLMf0HItB9IgzRg-M94FjZROtOWeGXOYW4wqBMyABTelYgjfjgNtoW05SN7npnSYKG1JZeRwrT5KgaRXJy7KHkguyN8vhoYTjSX0cG4VaOrJqgGY-jsQov1lJvZTok_YAkiDRbOhDu3ebdJdi313sGSkUjjkW73Fn9ztmpexLN3OxfmhUlDu0MHhEqgLJMg8koLWBJjpmVC0YdNPCXNofGgNECkiWCZHTZw8u9jnLhtP686CG_JTQFE33YTdh3mJOp7KwFdFvQefEX5Jb2vUXmfFXMBP3MhD5civiPg4Q0WLU4; ai_session=XYYeY|1545989192862|1545993911682
```

### Step 2

Ctrl+P，輸入 `>`，找到 `Rest Client: Generate Code Snippet`
選擇你的語言。

![選擇你的語言](/images/2018/restclient.gif)

## 參考

- [REST Client: 官方載點](https://marketplace.visualstudio.com/items?itemName=humao.rest-client)
- [REST Client: Github 專案](https://github.com/Huachao/vscode-restclient)
- [VSCode 實用套件： REST Client](https://blog.rex-tsou.com/2017/10/vscode-%E5%AF%A6%E7%94%A8%E5%A5%97%E4%BB%B6-rest-client/)

(fin)
