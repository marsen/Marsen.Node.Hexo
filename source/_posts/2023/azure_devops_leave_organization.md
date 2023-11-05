---
title: " [工作筆記] 退出 Azure DevOps 的組織"
date: 2023/11/05 19:27:29
---

## 前情提要

Azure DevOps 是一個由微軟開發的服務，提供開發團隊協作和快速交付軟體的解決方案。  
它包含了版控、工作追蹤、持續整合/持續部署 (CI/CD) 等功能。  
其中，我將重點使用其 Board 功能來進行工作項目的管理和追蹤。  
然而，有一天我發現自己被不知名的人邀請進了一個 Azure DevOps 的 Organizations，  
所以我開始尋找如何自行退出的方法。

## 解法

首先進入 dev.azure.com，點擊 user setting icon (這在畫面的右上角，是一個人的右下角有齒輪的圖案)。  
點擊 Profile 進到以下頁面，可以看到提示如下:  

```text
You are currently subscribed to other communication regarding Visual Studio and/or Visual Studio Subscriptions.   
Please visit the complete Azure DevOps Profile Page to change those settings.
```

點擊連結到 [Azure DevOps Profile Page](https://aex.dev.azure.com/me)  

這裡就可以看到你的 Organization 並選擇離開

## 誤區

在 [my account](https://myaccount.microsoft.com/organizations) 也有 Organization 但是不可以隨意離開。  
簡單來說，在微軟的 my account 中，`Organizations` 代表的是你所屬於的 Azure Active Directory (Azure AD) 組織。  
這些組織可能是你的工作場所、學校或其他團體，他們使用 Azure AD 來管理員工或成員的身份和訪問權限。  
這裡的組織並不等同於 Azure DevOps 中的 Organizations，並且不能隨意離開。

## 參考

- <https://learn.microsoft.com/en-us/entra/external-id/leave-the-organization>

(fin)
