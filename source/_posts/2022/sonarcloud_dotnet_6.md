---
title: " [實作筆記] SonarCloud with .NET 6.0.x"
date: 2022/02/15 10:00:57
tags:
  - 實作筆記
---

## 前情提要

[承上篇](https://blog.marsen.me/2022/02/03/2022/CI_update_with_stryker_1.3.1/),在升級了 .NET 6.0.x 後發現,  
SonarCloud 的測試含概率不知何時變成 0 ,  
相對應的 Code Smell 也沒有作檢查, 加上原本的 github/workflow 並沒特別分門別類,
所以真正的錯誤原因已經不可考, 修復 SonarCloud 的過程中特別撰文記錄.

## 實作

第一件事是在升級 .NET 6.0 後, 試著在本地環境上執行檢查並上傳到 SonarCloud.

### 前置條件

- 使用的電腦為 MacBook Pro (M1 晶片)
- 安裝 [.NET 6.0.x](https://dotnet.microsoft.com/en-us/download/dotnet/6.0) 以後的版本
- 安裝 [Java 11](https://www.oracle.com/java/technologies/downloads/)
- 安裝 [dotnet-sonarscanner](https://www.nuget.org/packages/dotnet-sonarscanner/)
- [註冊 sonarcloud.io 並建立專案](https://sonarcloud.io/projects/create)

### 本機的執行步驟

- dotnet sonarscanner begin

  ```shell
  dotnet sonarscanner begin \
  /o:"marsen-github" \
  /k:"Marsen.NetCore.Dojo" \
  /n:"Marsen.NetCore.Dojo" \
  /d:sonar.host.url="https://sonarcloud.io" \
  /d:sonar.cs.opencover.reportsPaths="./test/_/TestResults/_/coverage.opencover.xml" \
  /d:sonar.login="****"
  ```

  參數說明
  `/o`: Organization Key
  `/k`: Project Key
  `/n`: Project Name
  `/d:sonar.host.url`: sonar server url
  `/d:sonar.cs.opencover.reportsPaths`: OpenCover coverage report
  `/d:sonar.login`: 登入 sonar server 的 token

- dotnet build

  ```shell
  dotnet build Marsen.NetCore.Dojo.sln
  ```

- dotnet test

  ```shell
  dotnet test Marsen.NetCore.Dojo.sln \
  --logger trx --collect:"XPlat Code Coverage" \
  -- DataCollectionRunSettings.DataCollectors.DataCollector.Configuration.Format=opencover
  ```

- dotnet sonarscanner end

  ```shell
  dotnet-sonarscanner end /d:sonar.login="****"
  ```

### Github Workflow 的設定

目前並沒有[官方](https://github.com/SonarSource/sonarcloud-github-action)的解決方案 ,
我們使用 [SonarScanner for .NET](https://github.com/highbyte/sonarscan-dotnet) 來執行 workflow ,  
這個專案十分簡潔，可以快速看一下內容，了解到怎麼實作一個 workflow ,  
因為非官方解決方案，所以我 [fork 這個方案](https://github.com/marsen/sonarscan-dotnet)避免意外發生.

而在 workflow 中設定的也是使用 fork 的方案
設定參考如下

```yml
- name: SonarCloud Scan
  uses: marsen/sonarscan-dotnet@v2.1.2
  with:
    # The key of the SonarQube project
    sonarProjectKey: Marsen.NetCore.Dojo
    # The name of the SonarQube project
    sonarProjectName: Marsen.NetCore.Dojo
    # The name of the SonarQube Organization
    sonarOrganization: marsen-github
    # Optional extra command arguments the the SonarScanner 'begin' command
    sonarBeginArguments: /d:sonar.cs.opencover.reportsPaths="./test/*/TestResults/*/coverage.opencover.xml"
    # Optional. Set to 1 or true to not run 'dotnet test' command
    # dotnetDisableTests: true
    dotnetTestArguments: Marsen.NetCore.Dojo.Integration.Test.sln --logger trx -p:CoverletOutputFormat="opencover" --collect:"XPlat Code Coverage" -- DataCollectionRunSettings.DataCollectors.DataCollector.Configuration.Format=opencover
    dotnetBuildArguments: Marsen.NetCore.Dojo.sln
```

## 其它

注意到 [opencover](https://github.com/OpenCover/opencover) 的開發者已經不再維護(2021 年 6 月)，並建議使用 [AltCover](https://www.nuget.org/packages/altcover/).
但是 sonarcloud 並未支援 altcover 的報表, 這裡有相關的 [issue](https://community.sonarsource.com/t/add-support-for-altcover/56208) 追蹤中.

## 參考

- [sonarscan-dotnet](https://github.com/highbyte/sonarscan-dotnet)
- [Use code coverage for unit testing](https://docs.microsoft.com/en-us/dotnet/core/testing/unit-testing-code-coverage?tabs=linux)
- [How to Setup SonarQube](https://techblost.com/how-to-setup-sonarqube-locally-on-mac/)
- [dotnet-sonarscanner](https://www.nuget.org/packages/dotnet-sonarscanner/)

(fin)
