---
title: "[實作筆記] Stripe 串接"
date: 2019/06/17 12:56:17
tag:
  - API
---

## Agenda

- Stripe 簡介
- 註冊
- 付款

## Stripe 簡介

> Stripe是一家提供讓個人或公司在網際網路上接受付款服務的科技公司。  
> Stripe提供在網上接受付款所需的技術、避免信用卡詐騙技術及銀行基礎設施  

– 引述自 Wiki

> Our mission is to increase  
> the GDP of the internet  

– 引述自 Stripe 官網

業務範圍大多為歐美，亞洲方面支援香港、新加坡與日本等國…

## 註冊

只需要透過信箱即可[註冊](https://dashboard.stripe.com/register)，  
註冊後需要到信箱收取確認信以開通帳戶，
如果真的要在線上使用需要提供公司相關的資訊，  
但以開發者而言，此時的 Stripe 已經提供一組測試 api 供你使用，  
隨後即可以登入後台操作。  
**請在 Dashboard 的左邊側欄>開發者>API 密鑰，取得Secret key** ，  
在後面呼叫 API 中都會使用這組 Secret Key 請特別留意。

## 付款

這裡只介紹 Stripe 信用卡的付款方法，
並根據 [Stripe 文件](https://stripe.com/doc)整理一些資訊給大家。  

如下圖，這是一個標準的 Stripe 結帳流程，  
![stripe overview](/images/2019/6/01-overview.png)

主要的兩個步驟在 `Create Source` 與 `Create Charge` ，  
這裡會透過呼叫 [Stripe API](https://stripe.com/docs/api) 以完成付款流程。  

下面會介紹幾種信用卡的付款方式， 僅供參考，實際作業請以最新的 Stripe 文件為準。  
過程中如有呼叫 API 都會用 curl 帶過，
Secret Key 一律以 `sk` 表示， Public Key 以 `pk` 表示  
Stripe 有提供多種語言的範例或是提供 SDK 或 Libary， 請親自去看它們的文件囉。

### 使用 Checkout Session

#### Step 1. 建立 Session

```bash
curl https://api.stripe.com/v1/checkout/sessions \
  -u sk_test_dAa6L6BL4gZDuscgJcl3an8K00aJL2yIaW: \
  -d payment_method_types[]=card \
  -d line_items[][name]=T-shirt \
  -d line_items[][description]="Comfortable cotton t-shirt" \
  -d line_items[][images][]="https://example.com/t-shirt.png" \
  -d line_items[][amount]=500 \
  -d line_items[][currency]=hkd \
  -d line_items[][quantity]=1 \
  -d success_url="https://example.com/success" \
  -d cancel_url="https://example.com/cancel"
```

#### Step 2. 建立 CheckOut 頁面

Step 1 會取得一組 session_id ，請填入頁面中的`{session_id}`，
`pk` 請填入 `public key`

```html
<script src="https://js.stripe.com/v3/"></script>
<script>
var stripe = Stripe('pk');
stripe.redirectToCheckout({
  // Make the id field from the Checkout Session creation API response
  // available to this file， so you can provide it as parameter here
  // instead of the {{CHECKOUT_SESSION_ID}} placeholder.
  sessionId: '{session_id}'
}).then(function (result) {
  // If `redirectToCheckout` fails due to a browser or network
  // error， display the localized error message to your customer
  // using `result.error.message`.
});
</script>
```

#### Step 3. 載入頁面

調整你的付款流程，引導消費者到 Step 2. 的頁面，  
會自動轉導到 Stripe 的標準頁，並且出現填寫信用卡的資訊，  
消費者需要手動輸入卡號後，確認付款。  
如果確認會引導至 Step 1 的 `success_url`  
消費者取消的話會引導至 Step 1 的 `cancel_url`  

#### 說明

這是標準的第三方串接步驟，可以發現在 Step 3 的時候，  
消費者會被帶離你原本的站台到 Stripe 的付款頁面，  
這樣的好處是你不需要經手敏感的資料，像是信用卡卡號，  
但是有時候，轉導到外部頁面會讓消費者不安進而中斷結帳，  
那我們可以參考其它的作法。  

### 使用 Source

一般來說，Source 是 Stripe 最常用的付款方式，  
但在歐洲相關規定調整後，信用卡不再建議使用這個 API。  
可以參考官方文件的[說明](https://stripe.com/docs/sources/cards)

> Card Payments with Sources  
> Use Sources to accept card payments from around the world.  
>
> Use of this API is no longer recommended. We recommend adopting the Payment Intents API.  
> This new integration lets you benefit from Dynamic 3D Secure and helps you prepare for  
> Strong Customer Authentication regulation in Europe.


不過理論上您的客戶中沒有歐洲人的話，還是可以呼叫這個 API ，  
作法如下:

#### Step 1. Create Source 並指定 Type 為 Card

```bash
curl https://api.stripe.com/v1/sources
  -u sk
  -d type=card
  -d currency=hkd
  -d owner[email]="jenny.rosen@example.com"
  -d card[number]={{cardNumber}}
  -d card[exp_month]=12
  -d card[exp_year]=2020
  -d card[cvc]=123  
```

#### Step 2. Charge With Source

Step 1 可以取得 source id，利用 source id 呼叫 charge API 付款   

```bash
curl https://api.stripe.com/v1/charges
  -u sk
  -d source={{source id}}
  -d amount=411
  -d currency=hkd
  -d description="Charge for jenny.rosen@example.com"
  -d metadata[a]="b"
  -d metadata[b]=123
```

### 使用 Token

不過 Sorce 信用卡在[官方文件](https://stripe.com/docs/payments/payment-methods#transitioning)上不再被建議使用，
我們可以看看另一個類似的方法 *Token*

#### Step 1. Create Token 並傳入卡號

```bash
curl https://api.stripe.com/v1/tokens
  -u sk
  -d card[number]={{cardNumber}}
  -d card[exp_month]=12
  -d card[exp_year]=2020
  -d card[cvc]=123
```

#### Step 2. Charge With Token

Step 1 可以取得 token id，利用 source id 呼叫 charge API 付款

```bash
curl https://api.stripe.com/v1/charges
  -u sk
  -d source={{token id}}
  -d amount=412
  -d currency=hkd
  -d description="Charge for jenny.rosen@example.com"
  -d metadata[a]="c"
  -d metadata[b]=663
```

### 使用 Payment Intent

終於來到 Payment Intent 了，  
實際上這是目前 Stripe 最推薦的信用卡支付方式，  
呼叫的作法也很類似於 Source 與 Token，  
沒什麼特別考量的話，建議使用這個付款方式。

#### Step 1. Create Payment Method 並傳入卡號

```bash
curl https://api.stripe.com/v1/payment_methods
  -u sk
  -X POST  
  -d type=card  
  -d card[number]={{cardNumber}}
  -d card[exp_month]=12  
  -d card[exp_year]=2020  
  -d card[cvc]=123  
```

#### Step 2. Create Payment Intent With Payment Method

Step 1 可以取得 payment method id，  
利用 payment method id 呼叫 payment intents API 付款

```bash
curl https://api.stripe.com/v1/payment_intents  
  -u sk  
  -d payment_method={{payment method id}}
  -d amount=555  
  -d currency=hkd  
  -d confirmation_method=manual  
  -d confirm=true  

```

## 小結

想快速成立訂單請用 Session 的作法，  
想要避免轉換率下降，請使用 Payment Intents。

## 參考

- [Online payment processing for internet businesses - Stripe](https://stripe.com/en-hk)
- [DashBoard](https://dashboard.stripe.com/)
- [Documentation | Stripe](https://stripe.com/docs)
- [Stripe API Reference](https://stripe.com/docs/api)
- [Stripe Wiki](https://zh.wikipedia.org/wiki/Stripe)

(fin)
