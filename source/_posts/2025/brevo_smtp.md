---
title: " [實作筆記] 怎麼建立一個網站？(四) - 自訂網域 EMail 收寄信(使用 Cloudflare 與 Brevo)"
date: 2025/08/27 22:19:02
tags:
  - 實作筆記
---

## 前情提要

四年前「[怎麼建立一個網站？(四) - 自訂網域 EMail](https://blog.marsen.me/2020/10/22/2020/google_domain_forward_mail/)」已過時了，
當時透過 Google Domain 設定了自己的網域信箱，那種擁有 `admin@marsen.me` 的專業度真的是滿到溢出來。

但是，現實總是殘酷的。

2023 年 Google Domain 宣布停止營運，我選擇轉移到了 Cloudflare。

更悲劇的是，Gmail 的應用程式密碼也在安全性考量下越來越不被推薦使用。

Google 對 Gmail SMTP 的使用限制越來越嚴格。

在開啟 2FA 的情況下，只能使用應用程式密碼，但是 Google 本身也不推薦這樣使用密碼。

## 問題分析

現有問題：

- Google Domain 停止服務，Cloudflare 雖然接管了域名，但沒有提供類似的信箱轉發服務
- Gmail 無法繼續使用 Email Forwarding 的密碼方式不再被推薦使用，我也沒有查到替代方案

需求：

- 能夠使用 `admin@marsen.me` 收信，已完成(在 Cloudflare 設定 Email Forwarding)
- 能夠以 `admin@marsen.me` 寄信，而且不會被標記為可疑郵件
- 免費或便宜的解決方案
- 設定要簡單快速

## 技術選擇的思考

經過一番調研，我把目標鎖定在幾個主流的 SMTP 服務：

1. **Brevo (原 Sendinblue)**：免費額度每天 300 封信，付費方案 $25/月
2. **Mailjet**：免費額度每天 200 封信，付費方案 $17/月
3. **SendGrid**：免費額度每天 100 封信
4. **Amazon SES**：按量計費，超便宜但設定複雜

綜合考量下，我選擇 **Brevo** 的理由：

- **價格最划算**：免費額度最高（300封/天），對個人使用綽綽有餘
- **功能完整**：不只是 SMTP，還有完整的郵件行銷功能
- **設定超快**：API 整合簡單，文件齊全
- **信譽良好**：歐洲公司，GDPR 合規，信件到達率高

備案是 Mailjet，但既然 Brevo 實際使用很順利，我就沒有去試了。

## 實作過程

### Step 1: 註冊 Brevo 帳號

前往 [Brevo 官網](https://brevo.com) 註冊免費帳號，過程很簡單，不需要信用卡。

可以用 Google 帳號註冊，但是填寫一些資料，總體來說並不冗長繁瑣。

### Step 2: 設定 Sending Domain

登入後台，進入「Senders & IP」→「Domains」，添加你的域名（如 `marsen.me`）。

Brevo 會要求你在 DNS 域名設定中添加幾筆記錄(我是使用 Cloudflare)來驗證域名擁有權：

設定過程也很傻瓜點擊跟著操作就會引導你登入 Cloudflare 的後台，不保証其他域名商有這麼方便，

會在 Cloudflare DNS 管理中添加：2 筆 CNAME 與 2 筆 TXT 記錄，應該是讓 Brevo 驗証網域所有權的。

DNS 生效後，Brevo 就會驗證通過。

### Step 3: 設定收信

設定 Email Forwarding 這部分我直接在 Cloudflare 設定：

「Email」→「Email Routing」→「Routes」

添加轉發規則：

從：`admin@marsen.me` 到我的 gmail

這樣就能收到寄往自訂域名的信了。

### Step 4: 設定 Gmail 使用 Brevo SMTP

進入 Gmail 設定 > 帳戶和匯入 > 新增另一個電子郵件地址：

- 名稱：Marsen
- 電子郵件地址：`admin@marsen.me`
- SMTP 伺服器：`smtp-relay.brevo.com`
- 通訊埠：587
- 使用者名稱：你的 Brevo 帳號 email
- 密碼：去 Brevo 後台「Account Settings」→「SMTP & API」生成的 SMTP Key
- 設定完成後，Gmail 會寄驗證信，確認後收到的信才不會有警告。

## 實測結果

設定完成實測：

- ✅ 收信正常：寄到 `admin@marsen.me` 的信都能在 Gmail 收到
- ✅ 寄信正常：從 Gmail 可以選擇用 `admin@marsen.me` 寄信
- ✅ 信譽良好：收件者不會看到「未驗證」警告

整個設定過程不到 20 分鐘。

## 一些要注意的小問題

**DNS 生效時間**
SPF、DKIM 記錄可能需要幾個小時才會完全生效，不要急著測試。

但是我實測約幾分鐘就生效了。

**SMTP Key 不是密碼**
Brevo 的 SMTP Key 是專門給 API 和 SMTP 用的，不是你登入密碼。

**免費額度限制**
每天 300 封信對我個人使用很夠，但如果你要大量寄信，記得升級付費方案。

## 參考

- [Brevo 官方網站](https://brevo.com)
- [重要事項：我們不建議使用應用程式密碼](https://support.google.com/accounts/answer/185833)
- [怎麼建立一個網站？(四) - 自訂網域 EMail](https://blog.marsen.me/2020/10/22/2020/google_domain_forward_mail/) (2020年舊文)


## 系列文章

- [怎麼建立一個網站？(一)](https://blog.marsen.me/2016/08/21/2016/setting_DNS_with_google/)
- [怎麼建立一個網站？(二)](https://blog.marsen.me/2016/08/28/2016/how_to_use_github_page/)
- [怎麼建立一個網站？(三)](https://blog.marsen.me/2016/09/04/2016/http2_by_cloudflare/)
- ~~[怎麼建立一個網站？(四)](https://blog.marsen.me/2020/10/22/2020/google_domain_forward_mail/)~~
- [怎麼建立一個網站？(四)](https://blog.marsen.me/2025/08/27/2025/brevo_smtp/)
- [怎麼建立一個網站？(五)](https://blog.marsen.me/2021/04/06/2021/create_404/)

(fin)
