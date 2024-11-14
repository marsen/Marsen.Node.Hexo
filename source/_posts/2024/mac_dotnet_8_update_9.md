---
title: " [實作筆記] Macbook dotnet 8 升級 dotnet 9"
date: 2024/11/14 16:06:48
tags:
  - 實作筆記
---

## 前情提要

dotnet 9 [官宣囉](https://devblogs.microsoft.com/dotnet/announcing-dotnet-9/) ! 雖然不是 lts 版本，但是仍然有許多有趣的新東西，先昇級並記錄一下。

## 小常識

.NET Runtime 和 .NET SDK 主要的差別在於它們的用途和所包含的工具：  

.NET Runtime：  

僅包含運行 .NET 應用所需的基本庫和執行環境。  
適合只需要執行已編譯應用程序的情況，像是使用者在安裝應用後直接執行時。  
不包含編譯、打包或開發所需的工具和命令。  
.NET SDK (Software Development Kit)：  

包含 .NET Runtime，以及用來開發、編譯、測試和打包 .NET 應用的工具和命令。  
包括開發者常用的命令，如 dotnet build（編譯）、dotnet publish（發布）、dotnet test（測試）等。  
必須安裝 SDK 才能進行應用程式的開發，而不僅僅是執行。  

簡單來說：

**如果你只是想執行 .NET 應用，安裝 Runtime 即可。如果你要開發或修改 .NET 應用，則需要 SDK。**

## 安裝步驟

<https://dotnet.microsoft.com/en-us/download/dotnet/9.0>
選擇 SDK 9.0.100 (左側) > macOS Installers > Arm64

> 我曾經裝到 X64，安裝過程沒有失敗，但是 dotnet version 仍為 8.0.100

檢查

>❯ dotnet --version  
9.0.100  
❯ dotnet --list-sdks  
8.0.100 [/usr/local/share/dotnet/sdk]  
8.0.101 [/usr/local/share/dotnet/sdk]  
9.0.100 [/usr/local/share/dotnet/sdk]

## 專案升級

專案有許多套件有 Warning 先進行 `restore`

❯ dotnet restore Marsen.NetCore.Dojo.Integration.Test.sln

>❯ dotnet restore Marsen.NetCore.Dojo.Integration.Test.sln
> (中略)
> 在 24.4 秒內還原 成功但有 9 個警告

IDE 我是使用 Rider

> Tools → NuGet → Manage NuGet Packages for Solution...

把能昇級的都昇一昇，再執行測試

> 測試摘要: 總計: 467, 失敗: 0, 成功: 466, 已跳過: 1, 持續時間: 4.7 秒  
在 6.5 秒內建置 成功但有 9 個警告

> Tools → NuGet → Upgrade Packages in Solution...

再執行測試

> 測試摘要: 總計: 467, 失敗: 0, 成功: 466, 已跳過: 1, 持續時間: 4.7 秒  
在 6.5 秒內建置 成功但有 9 個警告

警告如下

warning NU1903: 套件 'System.Text.RegularExpressions' 4.3.0 具有已知的 高 嚴重性弱點
warning NU1903: 套件 'Newtonsoft.Json' 9.0.1 具有已知的 高 嚴重性弱點
warning CS0618: 'SqlConnection' 已經過時: 'Use the Microsoft.Data.SqlClient package instead.'
warning SYSLIB0051: 'Exception.Exception(SerializationInfo, StreamingContext)' 已經過時

高風險可以透過 Nuget 確認有沒有新版本的或 patch

[SYSLIB0051: Legacy serialization support APIs are obsolete](https://learn.microsoft.com/en-gb/dotnet/fundamentals/syslib-diagnostics/syslib0051) 基本上都是在繼承舊版 Exception 時建立的方法  
可以刪除，但這次先標已過時標籤後並保留，相關程式都沒有呼叫這些方法，所以不會引發新的警告。  

[warning CS0618: 'SqlConnection' 已經過時: 'Use the Microsoft.Data.SqlClient package instead.'](https://learn.microsoft.com/en-us/dotnet/api/system.data.sqlclient.sqlconnection.-ctor?view=netframework-4.8.1&viewFallbackFrom=net-8.0)
很簡單，依據他的建議，改用 `using Microsoft.Data.SqlClient;` 即可。  
當然要先從 Nuget 安裝 package

(fin)
