---
title: "[實作筆記] 怎麼建立一個網站？(四) - 自訂網域 EMail"
date: 2020/10/22 11:02:59
tag:
   - 實作筆記
---

## 前情提要

擁有自已域名的網站已經 4 年啦, 雖然我只有用在 Blog 上，
而且是用靜態網站的方式呈現，但也是算小小的一個成就.
不過其實心理還是有一個小小的缺憾，就是我沒有自已域名的 email.
之前沒有好好作功課，想到還要架設一個 Mail Server 就覺得不可能.
但其實並沒有那麼複雜.  
我使用 [Google Domain](https://domains.google.com/) 的服務結合 [CloudFlare](https://www.cloudflare.com/).  
大約只要 30 分鐘就能收到信啦.

## 實作

首先是進入 Google Domain 的後台,  
記得如果你人在台灣是不被允許使用 Google Domain 的服務,  
所以必需要透過 VPN 跳轉到美國才能操作後台喔.

真的是超簡單的,但是數量上限為 100 個,  
加上 Google 的服務常常說收就收, 有商務需求的人還是不建議這個方式喔.  
電子郵件 > 新增電子別名,輸入自已想要的名字,然後填入`現有收件者電子郵件地址`

![新增電子郵件別名](/images/2020/10/email/add_forward.jpgJ

![輸入轉寄電子郵件](/images/2020/10/email/add_name.jpg)

如果你全部使用 Google 的服務的話,這樣子的設定就好了,  
DNS 的 MX 記錄 Google 會自動幫你處理到好.  
但是如果你像我一樣, 使用了別的 DNS 服務就必須要自已手動將 MX 記錄註冊上去喔  
在 CloudFlare > DNS 就可以進行設定.  

|名稱/主機/別名|類型|存留時間 (TTL)|優先順序|值/回應/目的位置|
|---        |---|---    |---    |---    |
|空白或 @|MX|1 小時|5|gmr-smtp-in.l.google.com|
|空白或 @|MX|1 小時|10|alt1.gmr-smtp-in.l.google.com|
|空白或 @|MX|1 小時|20|alt2.gmr-smtp-in.l.google.com|
|空白或 @|MX|1 小時|30|alt3.gmr-smtp-in.l.google.com|
|空白或 @|MX|1 小時|40|alt4.gmr-smtp-in.l.google.com|

![MX](/images/2020/10/email/mx_record.jpg)

設定完成後依照 Google 的說法會收到一封信(但是有些情況收不到)就會生效,  
我實測上沒收到信,但是寄信馬上就生效了.
有了自已域名的信箱~~虛榮感~~專業度是不是又上升了幾個百分點呢?

## 補充 20201118

前面已經可以透過自已的域名收信啦，  
那麼怎麼寄信呢?  

想像一下，如果用 `admin@yourdomain.com` 收信，  
卻用 `youraccout@gmail.com` 回信，  
是不是整個 Low 掉了，而且收信者是不是會懷疑怎麼寄信給 A 卻是 B 回信呢?。

為了解決這個問題，請參考[在 Gmail 中使用別名傳送電子郵件](https://support.google.com/domains/answer/9437157)，  

1. 產生應用程式密碼
   - 登入Google帳戶後，前往[Google 帳戶](https://myaccount.google.com/)頁面。
    ![登入Google帳戶後，前往[Google 帳戶](https://myaccount.google.com/)頁面。](/images/2020/10/email/gmail_application_setting.jpg)
   - 左側安全性 > 應用程式密碼 > 選取應用程式(選擇郵件) > 選取裝置(其它) > **輸入自已的 Domain** > 產生
    ![登入Google帳戶後，前往[Google 帳戶](https://myaccount.google.com/)頁面。](/images/2020/10/email/gmail_application_add.jpg)
    ![輸入密碼](/images/2020/10/email/gmail_application_password.jpg)
   - 把系統產生的 16 個字元密碼記錄下來。等一下我們會需要用到這組密碼

2. 新增電子郵件別名
   - 登入 Gmail > 設定 > 帳戶和匯入 > 選擇寄件地址 > 新增另一個電子郵件地址
   - 電子郵件地址輸入 **自訂網域信箱** > 勾選「視為別名」
   - SMTP 伺服器輸入 **smtp.gmail.com** 通訊埠 **587**，選取**採用 TLS 的加密連線**
   ![SMTP 設定](/images/2020/10/email/gmail_application_smtp.jpg)
   - 使用者帳戶請輸入 Google 帳戶(不用包含`@gmail.com`)，密碼使用上一步的**應用程式密碼** > 新增帳戶
   - 系統會寄一封驗証信，驗証後即可使用新的信箱寄信了
   ![驗証信箱](/images/2020/10/email/gmail_application_verify.jpg)

3. 變更「寄件者」地址
   - 登入 Gmail > 設定 > 帳戶和匯入 > 選擇寄件地址
   - 設定回覆郵件時：來自和郵件傳送地址相同位置的回覆
   - 將自訂網域的信箱設定為預設郵件

## 參考

- [轉寄電子郵件](https://support.google.com/domains/answer/3251241?hl=zh-Hant#emailForwarding)
- [設定自訂名稱伺服器的電子郵件轉寄功能](https://support.google.com/domains/answer/9428703)
- [在 Gmail 中使用別名傳送電子郵件](https://support.google.com/domains/answer/9437157)

## 系列文章

- [怎麼建立一個網站？(一)](https://blog.marsen.me/2016/08/21/2016/setting_DNS_with_google/)
- [怎麼建立一個網站？(二)](https://blog.marsen.me/2016/08/28/2016/how_to_use_github_page/)
- [怎麼建立一個網站？(三)](https://blog.marsen.me/2016/09/04/2016/http2_by_cloudflare/)
- [怎麼建立一個網站？(四)](https://blog.marsen.me/2020/10/22/2020/google_domain_forward_mail/)

(fin)
