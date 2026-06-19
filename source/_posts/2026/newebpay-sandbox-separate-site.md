---
title: "[實作筆記] 藍新金流沙盒是獨立測試站，要另外申請"
date: 2026/06/20 00:42:00
tags:
  - 實作筆記
---

## 前情提要

在串接藍新金流時，以為在正式站註冊完就能直接測試，結果找了半天都找不到沙盒設定。
記錄一下這個讓人繞路的細節。

## 沙盒和正式站是兩個獨立網站

| | 正式站 | 沙盒（測試站） |
|--|--|--|
| 後台網址 | https://www.newebpay.com | https://cwww.newebpay.com |
| API endpoint | https://core.newebpay.com/MPG/mpg_gateway | https://ccore.newebpay.com/MPG/mpg_gateway |

帳號、商店、金鑰完全獨立，在正式站申請的帳號無法登入沙盒，反之亦然。

## 申請流程

1. 去 **https://cwww.newebpay.com** 另外註冊一個帳號
2. 審核通過後，到「商店管理」→「開立商店設定」建立測試商店
3. 進商店後台取得 `MerchantID`、`HashKey`、`HashIV`

個人就可以申請，不需要公司行號或統一編號。

## 測試信用卡

```text
卡號：4000-2211-1111-1111
有效期：任意未來日期（例如 12/30）
CVV：任意三碼（例如 123）
```

## 小結

藍新正式站和沙盒完全分開，要測試就要去 `cwww.newebpay.com` 另開一套帳號。
不像綠界有公開固定的沙盒憑證，藍新要自己申請才有測試用的 MerchantID / HashKey / HashIV。

(fin)
