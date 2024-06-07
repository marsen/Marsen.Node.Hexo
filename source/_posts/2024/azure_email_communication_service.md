---
title: " [實作筆記] Azure Communication Services email"
date: 2024/06/07 17:58:53
tags:
  - 實作筆記
---


## 前情提要

隨著現代企業在數位轉型中的步伐加快，電子郵件仍然是最重要的溝通工具之一。  
公司現在的寄信服務採取了一個特別的解決方案，透過 App registrations 進行郵件發送。  
實作的細節不展開，但這樣作有幾個問題:  

- App registrations 首次需人工授權取得 refresh_token
- 有了 refresh_token 仍需交換到 access_token 才能寄信
- 更新 refresh_token 失敗時，就需要人工重新介入

而 Microsoft Azure 提供了強大的 Email Communication Service，可以幫助企業輕鬆、有效地發送大量電子郵件。  
優點有整合簡單、高可擴展、安全性高、可靠性強等等優點…  

本篇文章將記錄我如何在 Azure 中實作 Email Communication Service。

## 實作步驟

1. 建立 Azure Communication Services 資源
  首先，登入 Azure 入口網站，然後依序進行以下步驟：  
  點選「建立資源」按鈕，搜尋並選擇「Communication Services」。  
  點選「建立」按鈕，填寫必要的資訊如資源名稱、訂閱和資源群組等。  
  選擇地區並設定其餘選項後，點選「檢閱 + 建立」，檢查設定並點選「建立」。  

2. 配置電子郵件通道
  在 Communication Services 資源建立完成後，需要進行電子郵件通道的配置：  
  在資源概覽頁面中，找到並點選「Email」。  
  點選「新增郵件域」，並按照提示設定 SMTP 資訊及其他相關設定。  
  驗證郵件域並完成配置。  
  *這裡你需要有 domain 管理者的權限，用來在 DNS Records 建立相關的記錄(CNAME、TXT)*

3. 生成 API 金鑰
  接下來，我們需要生成 API 金鑰，以便應用程式能夠通過此金鑰進行認證和發送電子郵件：  
  在 Communication Services 資源頁面中，找到並點選「密鑰」。
  點選「生成/管理密鑰」，生成新的 API 金鑰並保存。

4. 發送電子郵件
  有了 API 金鑰和配置好的電子郵件通道，現在可以使用 Azure 提供的 SDK 或 REST API 發送電子郵件。  
  以下範例展示了如何使用 Nodejs 發送郵件：  

```javascript
  import { EmailClient, type EmailMessage } from '@azure/communication-email'
  //...
  sendMail = async (req: Request, res: Response): Promise<void> => {
    const connectionString = env('COMMUNICATION_SERVICES_CONNECTION_STRING')
    const senderAddress = env('EMAIL_SENDER_ADDRESS')
    const client = new EmailClient(connectionString)
    const { to, subject, html } = req.body
    const attachments = (req.files != null)
      ? (req.files as Express.Multer.File[]).map(file => ({
          name: file.originalname,
          contentType: file.mimetype,
          contentInBase64: file.buffer.toString('base64')
        }))
      : []
    const emailMessage: EmailMessage = {
      senderAddress,
      content: {
        subject,
        html
      },
      attachments,
      recipients: {
        to: [{ address: to }],
        bcc: [{ address: 'noreply@marsen.me' }]
      }
    }

    const poller = await client.beginSend(emailMessage)
    const result = await poller.pollUntilDone()
    console.log('result:', result)
    res.status(200).json({ message: `Email ${subject} Sent!` })
  }
```

### 在 Azure Portal 執行整合測試

在 Azure Portal > Communication Service > Email > Try Email，  
非常貼心的提供了 C#、JavaScript、Java、Python、cUrl　的範本，  
也可以取得連線字串。

### 使用上限與新增寄件帳號

原則上預設的使用量對開發人員來說，是非常足夠的  
但是如果想增加上限，或是新增其它的寄件者帳號，需要開 support ticket 進行升級，  
這是為了避免郵件的濫用，另外需新增 MX Record 可以避免當成垃圾郵件。  
升級後就可以在 Azure Portal > Email Communication Service > Provision domains > MailFrom addresses  
新增寄件者，實際上 Azure Communication Service 只會寄件無法收件，  
即使在 O365 有相同的帳號，在寄件備份中也看不到透過 ECS 寄出的郵件。　　

## 小結

實作上比較有風險大概是 DNS Records 的設定，最久大約需等到 1 小時生效。  
而開發上非常的容易，甚至程式範例都整合到 Portal，非常方便。 　
但是其它方面需要開票等待 Azure 協力升級與設定，就會比較麻煩。 　

## 參考

- [Overview of Azure Communication Services email](https://learn.microsoft.com/en-us/azure/communication-services/concepts/email/email-overview)
- [Quickstart: Create and manage Communication Services resources](https://learn.microsoft.com/en-us/azure/communication-services/quickstarts/create-communication-resource?tabs=windows&pivots=platform-azp)
- [Quota increase for email domains](https://learn.microsoft.com/en-us/azure/communication-services/concepts/email/email-quota-increase)
- [Create Support Ticket](https://azure.microsoft.com/en-us/support/create-ticket/)

(fin)
