---
title: "[實作筆記] 使用 Request Bin 測試第三方 Webhook 與 Callback "
date: 2020/01/14 15:31:17
tag:
    - 實作筆記
---

## 案例說明

最近串接 Stripe ，一個相當方便的國際化金流服務，  
情境是主子帳號的綁定，  
Stripe 可以透過 Oauth 的方式綁定不同的 Stripe 帳戶，  
對於電子商務的平台來說，相當的方便，  
它可以透過簡單的[授權機制](https://stripe.com/docs/connect/standard-accounts#revoked-access)，取得客戶的金流資訊，  

![Strip Oauth Flow](/images/2020/1/requestbin_01.jpg)

這時候問題來了，營運部門發現有的時候客戶會不小心將授權解除，  
這會導致帳務上的問題，所以需要在第一時間被通知，  
而 Stripe 其實也有提供 Event Driven 的解決方案。

透過 Webhook 監聽指定的事件 (Event)，可以串接各種通知服務。
ex: Line、E-mail、簡訊甚至是電話，
開發這樣的通知服務並不難但是要時間的。

而其中最大的風險在於 Event 送到 Webhook 再到通知服務的這段流程。
如果這段不通，就算 Event 確實會發生，就算通知務服務是正常的，  
仍然會收不到通知。

**有沒有辦法快速驗証Webhook呢?**
**有沒有辦法快速驗証Webhook呢?**
**有沒有辦法快速驗証Webhook呢?**

## 網路上的 Solution

這個時候就要推薦一下網路上的這個服務 [Request Bin](https://requestbin.com/)  
登入後只需要一鍵，就可以快速建立一個 EndPoint  
而且立即生效，有任 Request 進來都會完整記錄。

### 好處

1. 快速建立 EndPoint
2. 立即生效
3. 免費

如果有其它類似的服務，請推薦。

## 參考

- <https://stripe.com/docs/webhooks>
- <https://stripe.com/docs/api/issuing/authorizations>
- <https://requestbin.com/>

(fin)
