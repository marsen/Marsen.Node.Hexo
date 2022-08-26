---
title: " [實作筆記] 自我簽署憑証, localhost run with https "
date: 2021/08/08 15:34:10
tag:
  - 實作筆記
---

## 前情提要

最近有一批學生在學習 Express 開發網站,  
他們需要透過 facebook 應用程式作登入。
遇到了以下的問題
![Facebook 偵測到 所使用的網連線並不安全](../../images/2021/fb_login_fail.png)

最簡單的解決方式，重新選擇你的 Facebook Application Type 為無

如下:
![Facebook Application Type 選擇無](../../images/2021/fb_application_type.png)
這樣足以解決學生們在練習開發上的問題了。

不過我有點好奇背後的真正原因是什麼 ?

## Facebook 的政策

Facebook 的[文件](https://developers.facebook.com/docs/facebook-login/security/#surfacearea)說明

> ### 使用 HTTPS
>
> 使用具有加密功能的 HTTPS 作為網際網路通訊協定，而非 HTTP。HTTPS 會維護傳送資料的隱私，保護其不受竊聽攻擊。  
> 此外，也能保護資料在傳送過程中不遭到置入廣告或惡意程式碼的竄改。
>
> 在 2018 年 10 月 6 日，所有應用程式都必須使用 HTTPS。

可能是一個原因，因為學生們的開發環境建置出來都是單純的 http ，同學也有作出相關的猜測。
那麼我們要怎麼快速驗証這個想法呢 ?

## https-localhost

這是一個最快速的驗証方法，不過需要修改部份的代碼，而且遮蓋了需多細節。
以下以 MacOS 操作為主

首先，需要安裝相依的 [NSS Tool](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS/tools/NSS_Tools_certutil)

> brew install nss

安裝 `https-localhost`

> npm i https-localhost

改寫在 Express 網站的 `app.js`

```javascript
//const app = express()
const app = require("https-localhost")();
```

基本這樣就完成了。

- 優點
  - 改動步驟少
  - 無需理解細節
- 缺點
  - 要改動原始代碼

## 自我簽署憑証

接下來我們會深入一點細節，並了解背後的機制。

### 憑証機制簡介

我不打算從 TCP、Http 講解到 Https，網路上已有相當多的資訊，  
或是未來有機會再補充。  
總而言之，當你使用 Https 時，有一個很重要的觀念是，  
你的 Server 應該提供一組`公鑰`給 User 作為資料的加解密。  
那延伸的問題就是**怎麼確認這組公鑰是可以信任的呢?**

簡單說就是透過第三方的公正單位進行認証，所謂憑証認証機構(CA)。  
我們相信這些機構，而機構也會幫我們進行審核。  
而一般來說你的應用程式，比如說瀏覽器會先安裝好可信任的根憑証。

一個經典的說明是這樣的:

> 鮑伯可以隨便把憑證向外發布。  
> 鮑伯與愛麗絲事先可能互不認識，但鮑伯與愛麗絲都信任伊凡，  
> 愛麗絲使用認證機構伊凡的公鑰驗證數位簽章，如果驗證成功，便可以信任鮑勃的公鑰是真正屬於鮑伯的。[1]  
> 愛麗絲可以使用憑證上的鮑勃的公鑰加密明文，得到密文並傳送給鮑伯。  
> 鮑伯可以可以用自己的私鑰把密文解密，得到明文。

以開發者而言，你可以自我簽署憑証，  
這裡你要同時扮演鮑伯，愛麗絲與伊凡，  
所以常有人會在其中迷失方向，角色錯亂。

### 伊凡(local CA)

伊凡是一個 CA，我們只要安裝 `mkcert`，
就可以建自一個本地端的 CA

> brew install mkcert

我們來對 `localhost` 發行憑証

> mkcert localhost

會產生下列兩個檔案，
分別為憑証 `localhost.pem`
與私錀 `localhost-key.pem`

### 鮑伯(Server)

這個時候我們角色切換至鮑伯(Server)
鮑伯在這個過程中最重要的發行公錀，
私錀是用來加密傳輸的資料的。

這裡需要修改一下 Server 的程式碼

```javascript
const fs = require("fs");
const https = require("https");

/// 中略…

https
  .createServer(
    {
      key: fs.readFileSync("localhost-key.pem"),
      cert: fs.readFileSync("localhost.pem"),
    },
    app
  )
  .listen(3000, function () {
    console.log(
      "Example app listening on port 3000! Go to https://localhost:3000/"
    );
  });
```

這樣我們就可以允許愛麗絲透過 Https 與我們溝通囉。

### 愛麗絲(Client)

基本你只要開啟瀏覽器，輸入 `https://localhost:3000/` 你就可以正常瀏覽了。
但我們要了解背後發生的過程，

安裝 `mkcert` 完畢後，其實會在你的機器(與瀏覽器)上安裝根憑証，
這樣愛麗絲才會信任伊凡  
可以透過以下語法查詢安裝路徑(不同 OS 會有差異)

> mkcert -CAROOT

這個作法改動的程式也不會比較少，
但至少不再有黑魔法(當然你也可以去追蹤 `https-localhost` 的代碼)

### Recap

快速回顧一下 `mkcert` 作了哪些事

1. 建立一個 local CA
2. 替你的瀏覽器安裝 CA
3. 對 localhost 發行憑証

而你的程式需要

- 改使用 https 啟動網站

## 別忘了解決學生問題

回到 facebook 應用程式登入的問題，  
別忘了去設定 callback 網址，並且確定是走 https  
**用戶端 OAuth 設定 > 有效的 OAuth 重新導向 URI**  
不然會得到以下錯誤

![忘了設定 callback URL](../../images/2021/fb_login_fail_wrong_domain.png)

## 小計

文末的聯結有需多是透過 OpenSSL 的作法供參考，  
但是會多一步驟需要安裝手動憑証(愛麗絲該作的事)。

另外仍然需要修改程式這點我不是很滿意，  
應該可以掛上其它的 Proxy Server，由 Proxy Server 處理 https 就好，  
內網單純走 http。
最近也有看到[零信任網路](https://www.youtube.com/watch?v=gC4wmZf7dAI)，  
不過那應該是另一個議題了

> Zero Trust 零信任架構近年來日受重視，在 Zero Trust 框架底下，  
> 即便使用者在內網環境，也不會給予沒有期效或過高之存取權限。

## 參考

- [https-localhost](https://www.npmjs.com/package/https-localhost?activeTab=readme)
- [[推薦] 快速產生本地端開發用 SSL 憑證工具-mkcert](https://xenby.com/b/205-%E6%8E%A8%E8%96%A6-%E5%BF%AB%E9%80%9F%E7%94%A2%E7%94%9F%E6%9C%AC%E5%9C%B0%E7%AB%AF%E9%96%8B%E7%99%BC%E7%94%A8ssl%E6%86%91%E8%AD%89%E5%B7%A5%E5%85%B7-mkcert)
- [mkcert, you changed my life!!](https://medium.com/@shriramsharma/mkcert-you-changed-my-life-b157466880bf)
- [一文搞懂 HTTP 和 HTTPS 是什麼？兩者有什麼差別](https://tw.alphacamp.co/blog/http-https-difference)
- [極速打造 https://localhost](https://onlinemad.medium.com/%E6%A5%B5%E9%80%9F%E6%89%93%E9%80%A0-https-localhost-431d89a0c2e4)
- [How To Create an HTTPS Server on Localhost using Express](https://medium.com/@nitinpatel_20236/how-to-create-an-https-server-on-localhost-using-express-366435d61f28)
- [Running express.js server over HTTPS](https://timonweb.com/javascript/running-expressjs-server-over-https/)
- [使用 localhost 建立憑證](https://letsencrypt.org/zh-tw/docs/certificates-for-localhost/)
- [如何使用 OpenSSL 建立開發測試用途的自簽憑證 (Self-Signed Certificate)
  ](https://blog.miniasp.com/post/2019/02/25/Creating-Self-signed-Certificate-using-OpenSSL)

(fin)
