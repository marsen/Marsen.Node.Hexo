---
title: "[學習筆記] 允許 IAM User 存取 AWS Billing Console"
date: 2018/04/04 17:08:44
tag:
  - AWS
---

## 前情提要

設定了 IAM User Account 也給予了 Administrator 的權限,  
不過仍然看不到 Billing 的頁面資訊 .  

![Billing](https://i.imgur.com/1Ge6pGi.jpg)

這帶來了很大的不方便, 因為如果要看 Billing 的資訊就要切換到 Root Account  
而建立 Administrator IAM Account 的用意本來就是要儘可能不使用 Root Account 作登入.  
檢查了權限,明明就有設定 Read Billing 但是仍然看不到.  

## 解決方法

實際上要進入 Billing Console 其實要有兩個步驟  

1. 權限要設定,更多細節可以參考這篇[文章](https://aws.amazon.com/blogs/security/enhanced-iam-capabilities-for-the-aws-billing-console/)(2014)
2. 要透過 Root Account 在 [Account Settings](https://console.aws.amazon.com/billing/home#/account) 頁面設定, 允許 IAM user 存取 Billing Console  

![Root Account](https://i.imgur.com/yBXaLPJ.jpg)

## 參考

- [Don’t Forget to Enable Access to the Billing Console!](https://aws.amazon.com/blogs/security/dont-forget-to-enable-access-to-the-billing-console/)

(fin)
