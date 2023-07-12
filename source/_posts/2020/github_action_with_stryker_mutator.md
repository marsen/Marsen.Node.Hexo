---
title: " [實作筆記] Github 結合 Stryker 作變異測試"
date: 2020/05/06 09:03:30
tags:
  - CI/CD
  - 實作筆記
---

## 前情提要

前年初次接觸[變異測試](https://blog.marsen.me/2018/03/20/2018/mutation_testing/)，去年在看[重構](https://blog.marsen.me/2019/03/01/2019/book/refactoring/refactoring_Ch1/)時偷換了概念，  
將兩者結合了。

這次我想更進一步，將 CI Server 與之結合。

## Stryker.Net

根據 [Stryker Handbook](https://github.com/stryker-mutator/stryker-handbook/blob/master/dashboard.md)，
Stryker 主要有三個專案，

- Stryker (Javascript & TypeScript)
- Stryker4s (Scala)
- Stryker.NET (.NET)

我會使用我的練習用[專案](https://github.com/marsen/Marsen.NetCore.Dojo/)作為目標，
所以很理所當然的我會選擇 `Stryker.NET`，那我們就開始吧。

## 開始

### 安裝 Stryker

```shell
dotnet tool install -g dotnet-stryker
```

### 執行 Stryker

```shell
dotnet stryker -tp "['./test/Marsen.NetCore.Dojo.Tests/Marsen.NetCore.Dojo.Tests.csproj','./test/Marsen.NetCore.Dojo.Integration.Tests/Marsen.NetCore.Dojo.Integration.Tests.csproj']" -p="Marsen.NetCore.Dojo.csproj" -dk=$STRYKER_DASHBOARD_API_KEY
```

注意幾個 cli 參數，  
`-tp` 明確指定測試的專案有哪些，可以用中括號`[]`傳入多個測試專案名稱，使用`,`作為分隔符  
`-p` 專案名稱  
`-dk` dashboard-api-key 這組 key 是用來與 `https://dashboard.stryker-mutator.io/` 互動的，  
最主的功能是將報告上傳。  
$STRYKER_DASHBOARD_API_KEY 是 [Github 的 Secrets](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets)

你可以執行 `dotnet stryker -h` 查看更多原始說明

```shell
  -tp|--test-projects Specify what test projects should run on the project under test.
  -p|--project-file <projectFileName> Used for matching the project references when finding the project to
                                                mutate. Example: "ExampleProject.csproj"
  -dk|--dashboard-api-key <api-key> Api key for dashboard reporter. You can get your key here: https://dashboard.stryker-mutator.io
```

### 專案設定

新增一個 `stryker-config.json`  
對我來說，最重要的是要記得設定 `stryker-config > reporters > dashboard` 這筆資料。  
[詳細的說明的可以參考這篇](https://github.com/stryker-mutator/stryker-net/blob/master/docs/Configuration.md)

```json
{
  "stryker-config": {
    "dashboard-project": "github.com/marsen/Marsen.NetCore.Dojo",
    "dashboard-version": "master",
    "reporters": ["json", "dashboard"]
  }
}
```

### 在 Github 上工作

首先你必須在 [Dashboard](https://dashboard.stryker-mutator.io/)中 Enable Repository

![Enable Repository](/images/2020/5/github_action_with_stryker_mutator_01.jpg)

然後產生一組 Key

![產生一組 Key](/images/2020/5/github_action_with_stryker_mutator_02.jpg)

接下來到 Github ， 我們將這組 Key 設定到 Secrets 之中

![將 Key 設定至 Github](/images/2020/5/github_action_with_stryker_mutator_03.jpg)

再到 Github Actions 中，把 workflow 指令設定完成，

![Github Actions](/images/2020/5/github_action_with_stryker_mutator_04.jpg)

觸發 CI 完成後，就可以在 [Dashboard](https://dashboard.stryker-mutator.io/reports/github.com/marsen/Marsen.NetCore.Dojo/master) 上看到報表啦。

![結果呈現](/images/2020/5/github_action_with_stryker_mutator_05.jpg)

## 參考

- <https://blog.marsen.me/2019/03/01/2019/book/refactoring/refactoring_Ch1/>
- <https://blog.marsen.me/2018/03/20/2018/mutation_testing/>
- <https://github.com/stryker-mutator/stryker-handbook/blob/master/dashboard.md>
- <https://github.com/stryker-mutator/stryker-net/blob/master/docs/Reporters.md#dashboard-reporter>

(fin)
