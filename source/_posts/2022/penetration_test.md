---
title: " [實作筆記] 滲透測試報告、原因與調整方針"
date: 2022/05/24 11:33:04
---

## 前情提要，測試報告

測試廠商使用 Acunetix 掃描與 Burp Suite 攔截封包進行測試:
報告為以下幾個 Header 未加而使網站存在風險

1. X-Frame-Options
2. X-XSS-Protection
3. X-Content-Type-Options
4. Strict-Transport-Security
5. Content-Security-Policy

除了第五項，以下將會介紹各個 Header 賦於瀏覽器的行為，以及防範何種可能的攻擊

## Header 的用意

### X-Frame-Options

- 新版的瀏覽器使用 `frame-ancestors`
- 用意：防範 [ClickJack](https://developer.mozilla.org/en-US/docs/Web/Security/Types_of_attacks#click-jacking) 攻擊
  > Click Jacking 直譯為點擊劫持，常見的攻擊手段為，
  > 攻擊者透過 frame 嵌入受害網站;當受害者誤入攻擊者網站時(常用社交攻擊)，
  > 會誤以為是正常網站。
  > 接下來就可以誘騙受害者，ex:輸入帳密，或是點擊特定連結或下載惡意軟體。

#### 選項

- `DENY`
- `SAMEORIGIN`
- `ALLOW-FROM <uri>`

### X-XSS-Protection

- 新版的瀏覽器透過 `CSP (Content-Security-Policy)` 來防止 XSS 攻擊
- 用意:為提供舊版的瀏覽器防範 XSS 網站攻擊，這不屬於任何正式的規格書或草案之中
  > XSS(Cross-Site Scripting)，直接跨站腳本攻擊，手段非常的多樣，
  > 大多是在受害網站注入腳本語法(javascript)，使用者瀏覽時就會遭受攻擊。
  >
  > ex:在留言區留下無限 loop 的 alert ，如果對方的網站沒有防範，
  > 使用者瀏覽留言區的時候，就會彈出關不完的 alter 視窗。
  > (真正惡意的攻擊會更神不知鬼不覺)
  > 參考
  >
  > - [1](https://tech-blog.cymetrics.io/posts/huli/xss-attack-and-defense/)
  > - [2](https://tech-blog.cymetrics.io/posts/jo/zerobased-cross-site-scripting/)

#### 選項

- `0`
  禁止 XSS 過濾。
- `1`
  啟用 XSS 過濾（通常瀏覽器是默認的）。如果檢測到跨站腳本攻擊，瀏覽器將清除頁面（刪除不安全的部分）。
- `1;mode=block`
  啟用 XSS 過濾。如果檢測到攻擊，瀏覽器將不會清除頁面，而是阻止頁面加載。
- `1; report=<reporting-URI>` (Chromium only)
  啟用 XSS 過濾。如果檢測到跨站腳本攻擊，瀏覽器將清除頁面並使用 CSP report-uri (en-US)指令的功能發送違規報告。

### X-Content-Type-Options

- 通知瀏覽器，不要自行猜測(MIME type sniffing)Content Type

> 原先 MIME type sniffing 是上古時代被設計來避免開發人員誤設 Content Type，
> 比如說明明是 javascript 檔， content Type 卻被設定 text/plain，
> 這時瀏覽器 sniff 判斷為 javascript 後，可以執行該 js 檔
> 但是被用來惡意攻擊，比如說連結 `src/fake.jpg` 看起來像一張圖片
> 實際上 Response 卻回傳可執行的 javascript (內含惡意的程式)，
> 瀏覽器的 sniffing 行為會自行解析為腳本(javascript)並執行攻擊

#### 選項

- `nosniff`

### Strict-Transport-Security

`HTTP Strict-Transport-Security` 回應標頭（簡稱為 HSTS (en-US)）告知瀏覽器應強制使用 HTTPS 以取代 HTTP。
現代的瀏覽器優先使用 HTTPS 已是共識，不加這個 Header 也會出現不安全的警告

#### 選項

- `max-age` 指定秒數
- `includeSubDomains` 包含子網域

### Content-Security-Policy

這個坑很深，我另外再撰文說明，
可以透過另一個 Header [Content-Security-Policy-Report-Only](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy-Report-Only)，
產生報告逐一修正 Policy。

## 補充

### 如何使用 How to use testssl

testssl.sh 是一個免費的 command tool，  
它檢查任何端口上的服務器服務是否支持 TLS/SSL 、  
協議以及最近的密碼缺陷等。

安裝

```shell
brew install testssl
```

使用

```shell
testssl.sh {uri}
```

### 如何在 GCP Bucket 加上 header

GCP > Network Services > Load Balancing

- Edit
- Backend Configuration
- 找到特定的 Bucket 點擊鉛筆符號
- Edit backend bucket
- 展開 ADVANCED CONFIGURATIONS (RESPONSE HEADERS)
- Custom response headers > ADD HEADER
- 輸入 Header Key 與 Value
  - X-Frame-Options: DENY
  - X-XSS-Protection: 1;mode=block
  - X-Content-Type-Options: nosniff

## 補充: 如何使用 Burp Suite 觀察

概念說明：
Burp 可以在你的本機建立一個 Proxy Server,  
並透過它提供的瀏覽器(Chromium)連線到你指定的 uri 並進行欄截。

> You ←→ Burp(Proxy) ←→ Internet。

**注意**: 網路上大多數的文章都會有 Proxy 的設定，其實這是可以省略的，  
新版本的 Burp 用提供的瀏覽器就會連結 Burp Proxy。

1. 下載並安裝 [Burp Suite](https://portswigger.net/burp/communitydownload)，Burp Suite Community Edition 版本即可
2. 選擇 Temporary project > Next
3. 選擇 Use Burp defaults > Start Burp
4. 開啟攔截器: Proxy > Intercept is on
5. 開啟網站: Proxy > Open Browser(Chromium) > Key In \<target site URL\>
6. 決定欄截或丟棄 Proxy > Forward / Drop
7. 查看歷史記錄 Proxy > HTTP / WebSockets history

## 參考

- CSP 相關(留作下篇文章的養份)

  - <https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP>
  - <https://csper.io/blog/csp-report-filtering>
  - <https://content-security-policy.com/>
  - <https://devco.re/blog/2014/04/08/security-issues-of-http-headers-2-content-security-policy/>
  - <https://ithelp.ithome.com.tw/articles/10272394>
  - <https://github.com/WebAssembly/content-security-policy/blob/main/proposals/CSP.md>
  - <https://hackmd.io/@Eotones/BkOX6u5kX>
  - <https://runebook.dev/zh-CN/docs/http/headers/content-security-policy>
  - <https://mmazzarolo.com/blog/2021-12-14-lessons-learned-publishing-a-content-security-policy/>
  - <https://www.appsecmonkey.com/blog/content-security-policy>
  - <https://bugs.chromium.org/p/chromium/issues/detail?id=915648>
  - <https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy>

- <https://hackercat.org/burp-suite-tutorial/web-pentesting-burp-suite-total-tutorial>

(fin)
