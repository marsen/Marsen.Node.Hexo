---
title: "[實作筆記] 貢獻微軟文件 MSDN 記錄"
date: 2022/05/26 20:51:33
---

## 起因

專案上我使用 .net core 並且有需要使用到 Config 相關的設定，  
在看文件時不小心，查錯了文件(查到了[這一篇 ContextInformation.GetSection(String) Method](https://docs.microsoft.com/en-us/dotnet/api/system.configuration.contextinformation.getsection?view=dotnet-plat-ext-6.0))。

這裡寫到 configuration section 是區分大小寫的(case-sensitive)

> ## Remarks
>
> When specifying a section within the configuration,  
> note that the name of the configuration section is **case-sensitive**.

而我實測時，才發現並沒有區分大小寫，才注意到我**查錯了文件**。

## 起念

進一步我去微軟 MSDN 找到正確的文件查詢 [`ConfigurationManager.GetSection`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.configuration.configurationmanager.getsection?view=dotnet-plat-ext-6.0)，發現並沒有註明是否區分大小寫。

所以我就想~~解成就~~貢獻一下一已力，  
幫微軟加上說明，並好奇多久可以將修改的文件上線

## 歷程

首先只要點選右上角的鉛筆圖示，就可以連線至 [Github](https://github.com/dotnet/dotnet-api-docs/blob/main/xml/Microsoft.Extensions.Configuration/ConfigurationManager.xml) 頁面，  
很快的我完成了修改並且發出了 [Pull Request](https://github.com/dotnet/dotnet-api-docs/pull/8064) 。  
發出後很快就有自動化的機器人 dotnet-issue-labeler 與 opbld33 進行審查。

接下來有人工審核，[Gewarren](https://github.com/gewarren) 很友善的給我一些建議，  
大概就在我發出 PR 的幾個小時內，不過我自已的動作沒那麼快，  
我過了 5 天才發現，並依照建議完成了修改(我想他們應該有一些文件修改的 Guide Line)，  
修改完後送出，一樣會有自動化審核，Gewarren approve 後，再有 3 個自動化的檢查，

- snippets-build
- OpenPublishing.Build Validation status: passed
- license/cla All CLA requirements met.

接下來就 merged，這是 2022/05/17 ，不知道多久才會更新文件。

## 結尾

2022/05/23 我已經在微軟官方文件見到更新，小小成就達成 🎉

(fin)
