---
title: " [實作筆記] SendGrid 設定 on Appsmith"
date: 2021/07/08 14:43:10
tag:
  - 實作筆記
---

## 前言

appsmith 是一個簡單網站應用, 可以快速客製表單應用,  
背後使用的技術棧有

- docker
- nginx
- redis
- mongodb
- certbot
- watchtower
- sendgird

## Sendgird

1. [註冊](https://sendgrid.com/)建立帳號
   - 需準備手機
   - 需安裝 Authy 作兩階段驗証
2. 前往 [Dashboard](https://app.sendgrid.com/)
   - 至少設立一個 [Sender](https://app.sendgrid.com/settings/sender_auth/senders)
3. 建立一組 API Key

   - [Integrate 選擇 SMTP](https://app.sendgrid.com/guide/integrate)
   - API Key 大概長這樣（僅供參考已作廢）：`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`
   - 驗証[你的 Sender 信箱](https://app.sendgrid.com/settings/sender_auth)
   - 驗証 Application 的寄信功能, 通常應使用你的應用程式或系統進行測試
   - **這裡我使用 Appsmith 登入頁面的「忘記密碼」功能進行這個驗証, 參考下面 [Appsmith Email 設定](https://docs.appsmith.com/setup/docker/email/sendgrid)**

4. 補充，可以[管理(CRUD)你的 API Key](https://app.sendgrid.com/settings/api_keys)

## 設定 Appsmith

1. 使用 [docker 部署你的環境](https://docs.appsmith.com/setup/docker)
2. 設定 [sendgird](https://docs.appsmith.com/setup/docker/email/sendgrid),也可以參考上面的說明
3. 設定 `docker.env`

```dockerfile

## **\*** Email ****\*\*****

APPSMITH_MAIL_ENABLED=true
APPSMITH_MAIL_FROM=YOUR_SENDER_IDENTITY_EMAIL_ID
APPSMITH_REPLY_TO=YOUR_SENDER_IDENTITY_EMAIL_ID
APPSMITH_MAIL_HOST=smtp.sendgrid.net
APPSMITH_MAIL_PORT=587

## **\*** Set to true if providing a TLS port **\*\***

APPSMITH_MAIL_SMTP_TLS_ENABLED=true
APPSMITH_MAIL_USERNAME=apikey
APPSMITH_MAIL_PASSWORD=YOUR_SENDGRID_API_KEY #把 sendgrid 的 api key 放這裡
APPSMITH_MAIL_SMTP_AUTH=true
　　``` 4. 重啟 docker 與服務

```shell
sudo docker-compose rm -fsv appsmith-internal-server nginx && sudo docker-compose up -d
```

## 參考

- [certbot](https://certbot.eff.org/)
- [mongodb](https://www.mongodb.com/live)
- [appsmith](https://docs.appsmith.com/)
- [sendgrid](https://sendgrid.com/)

(fin)
