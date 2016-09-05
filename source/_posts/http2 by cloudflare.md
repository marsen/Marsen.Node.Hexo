---
title: 怎麼建立一個網站？(三) - 簡單讓網站升級使用HTTP/2
tag:
  - HTTP/1.1
  - HTTP/2
  - TCP
---
## 應該知道的事
- HTTP/1.1是目前被應用得最廣泛的通訊協定，也可以說是最成功的一個
  - HTTP/1.1 基於TCP，需要三方交握。
  - HTTP/1.1本身存在得需多問題，安全性、效率等…。
- HTTP/2 可以有效改善HTTP/1.1的缺點，而且可以向下相容
  - 主流的瀏覽器都開始支援了。  
  - 只要設定好Web Server，Client 端看使用的[瀏覽器支不支援](http://caniuse.com/#feat=http2)。
- 一流公司制定標準，二流作品牌，三流賣技術，四流作產品
  - HTTP/2 是源自Google的SPDY。
  - Google下一代 QUIC推動中。
- HTTP/2 並不強制加密(HTTPS)
  - 在主流瀏覽器為了向下相容，保留了80 port的HTTP/1.1通訊協定，用443 port 實作HTTP/2。
  - 所以還是要走HTTPS，所以需要安全性憑証。
  - 安全性憑証要錢(現在也有免費的)。
  - [cloudflare](https://www.cloudflare.com/)提供免費的安全性憑証。

## 前置作業
1. 你要有[cloudflare](https://www.cloudflare.com/)帳號。
2. 看過我之網站的設定，簡單說就是一個github page 與 透過OpenShift 這個 PaaS服務建立的網站。
    - [怎麼建立一個網站？(一)](https://blog.marsen.me/2016/08/21/setting_DNS_with_google/)
    - [怎麼建立一個網站？(二)](https://blog.marsen.me/2016/08/28/how_to_use_github_page/)

## 配置
1. [cloudflare](https://www.cloudflare.com/) 設定DNS，在指定的domain下，設定為「DNS and HTTP Proxy(CDN)」。
![設定為「DNS and HTTP Proxy(CDN)」](/images/090516_021242_AM.jpg)
2. 設定Page-Rules，這個是為了開啟SSL
    1. 設定`SSL`為`Flexible`,輸入`*.username.com/*`。  
這個時候你就可以瀏覽`https://blog.username.com/`,但同時也可以瀏覽`http://blog.username.com/` 。  
我希望HTTP能自動跳轉HTTPS，這部份如果是Node建立的網站，我可以用程式作轉導，  
但是如果是github page建立網站，就要再仰賴 [cloudflare](https://www.cloudflare.com/) 了。
3. 設定Page-Rules，這個是為了開啟SSL
    1. 設定`ALWAYS USE HTTPS`,輸入`http://*.username.com/*`。  
![設定為「DNS and HTTP Proxy(CDN)」](/images/090516_023252_AM.jpg)

設定完大致如下，順利的話幾分鐘就生效了。  
![](/images/090516_023554_AM.jpg)

這時候瀏覽`http://blog.username.com/`就會轉導到`https://blog.username.com/`了 。

## 檢驗
使用support HTTP/2的瀏覽器，瀏覽網頁，觀察其protocol 。
![](/images/090516_024550_AM.jpg)

(fin)

## 參考
- [了解 HTTP/2 的特色與 HTTP/1.1 的差異](https://simular.co/knowledge/site-build/68-about-http2-and-http11.html)
- [你的網站升級到 HTTP/2 了嗎？](https://blog.alphacamp.co/2016/07/12/http2/)
- [為什麼我們需要HTTP/2？](http://www.ithome.com.tw/voice/94371)
- [HTTP/2登場，邁向新的紀元](http://www.ithome.com.tw/voice/94520)
- [Google：將推動QUIC成為網路標準](http://www.ithome.com.tw/news/95353)
