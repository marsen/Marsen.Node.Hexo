---
title: "[實作筆記] Stryker 升級與 CI 調整"
date: 2022/02/03 20:48:54
---

## 前情提要

最近換了工作，一頭栽進了前端的領域，  
生活又開始忙錄了起來，過年檢視了一下自已的 Side Project ，
意外發生 CI 竟然壞了一陣子，
這是我使用的變異測試工具 Stryker 從原本的 0.17 升級到了 1.3.1 版本
這是一個蠻大幅度的升版，所以設定上相當的不一樣。

我的 OS 是 macOS Monterey 12.1
安裝了 .NET 6.0

## Local 環境

首先全域安裝 dotnet-stryker

> dotnet tool install -g dotnet-stryker

接下安裝 tool-manifest

> dotnet new tool-manifest

這時在你專案底下應該會建立一個 .config 資料夾，
內含 dotnet-tools.json 檔，內容如下

```json
{
  "version": 1,
  "isRoot": true,
  "tools": {
    "dotnet-stryker": {
      "version": "1.3.1",
      "commands": ["dotnet-stryker"]
    }
  }
}
```

下一步，在專案資料下執行以下命令

> dotnet tool install dotnet-stryker

建立 stryker-config.json 檔案
最後設定你的 config 檔 stryker-config.json

```json
{
  "stryker-config": {
    "project": "Marsen.NetCore.Dojo.csproj",
    "test-projects": [
      "./test/Marsen.NetCore.Dojo.Tests/Marsen.NetCore.Dojo.Tests.csproj",
      "./test/Marsen.NetCore.Dojo.Integration.Tests/Marsen.NetCore.Dojo.Integration.Tests.csproj"
    ],
    "reporters": ["progress", "json", "dashboard"],
    "project-info": {
      "name": "github.com/marsen/Marsen.NetCore.Dojo",
      "version": "main"
    }
  }
}
```

與之前最大的不同是，大部份的參數你都可以設定在 stryker-config 之中，
而不用在執行命令中傳入，不僅可以為其提供版本控制，更能有序的組織你的設定檔，
使其更好閱讀。
如果你的 `reporters` 中有設定 `dashboard` 需要額外加上 `--dashboard-api-key`
ex:

> dotnet stryker --dashboard-api-key=$STRYKER_DASHBOARD_API_KEY

## 參考

- [Announcing Stryker.NET 1.0](https://stryker-mutator.io/blog/2021-30-10-announcing-stryker-net-1.md/)
- [Install prerequisites](https://stryker-mutator.io/docs/stryker-net/getting-started)

(fin)
