---
title: "[實作筆記] 讓 SonarQube 檢查你的代碼 "
date: 2019/05/16 17:13:18
tag:
    - .Net Framework
---

## 前情提要

1. SonarQube 是一個開源的代碼品質(Quality)管理系統
2. 我目前的公司 N 社是自架 SonarQube Server 再與 CI 結合
3. 能夠透過工具讓代碼品質提昇，我訂定的目標如下
   - 免費
   - 能夠與 CI 結合，持續檢查代碼品質
   - 與 Side Project 結合

## 應該要知道的事

1. 掃瞄環境為 Windows
2. 掃瞄專案為 .Net Core 2.2 版
3. 使用 PowerShell 執行 Command
4. 也許不需要知道

## 實作筆記

1. 申請[SonarCloud](https://sonarcloud.io)帳號，我直接使用 Github

   - 需要允許 SonarCloud 存取 Github 的專案 Repo
   - 建立一組 Token, 用來作身份驗証，可以重複使用請勿外流
   - 如果要刪除 Token 請至 My Account > Security 找到並 Revoke
      ![建立 token](/images/2019/5/sonarcloud_gen.jpg)  
      ![建立 token2](/images/2019/5/sonarcloud_gentoken.jpg)  
   - 下載 SonarQube 執行檔，請選擇你的語言
      ![執行命令](/images/2019/5/sonarcloud_command.jpg)

2. 執行掃瞄前的準備作業

   - 設定 Path (實務上我沒有設定)
   - 切換到專案目錄底下

3. 啟動掃瞄，以 .Net Core 為例

```shell
> dotnet "{path of sonar scanner}\SonarScanner.MSBuild.dll" begin /k:"{project name}" /o:{group name} /d:sonar.host.url="https://sonarcloud.io" /d:sonar.login="{your token}"
```

輸出結果

```shell
Using the .NET Core version of the Scanner for MSBuild
Pre-processing started.
中略...
16:40:31.855  Pre-processing succeeded.
```

1. 建置專案

```shell
dotnet build
```

輸出結果

```shell
Microsoft (R) Build Engine version 15.9.20+g88f5fadfbe for .NET Core
Copyright (C) Microsoft Corporation. All rights reserved.
中略...
Build succeeded.
```

1. 上傳結果

```shell
> dotnet "{path of sonar scanner}\SonarScanner.MSBuild.dll" end /d:sonar.login="{your token}"
```

輸出結果

```bash
SonarScanner for MSBuild 4.6.1
Using the .NET Core version of the Scanner for MSBuild
Post-processing started.
中略...
INFO: Analysis total time: 21.446 s
INFO: ------------------------------------------------------------------------
INFO: EXECUTION SUCCESS
INFO: ------------------------------------------------------------------------
INFO: Total time: 2:27.021s
INFO: Final Memory: 24M/72M
INFO: ------------------------------------------------------------------------
The SonarQube Scanner has finished
```

最後到 SonarCloud 的網站上就可以看到報告結果，  
下一步就是將這整段流程結合 CI ，官網推薦是使用 Travis CI,
也有相同的文件與資源，我會試試看或是使用 Jenkins,
如果有機會能更進一步，我想結合 [OpenShift](https://www.openshift.com/) ,
讓部署的過程中結合代碼品質檢查。

![結果上傳](/images/2019/5/sonarcloud_result.jpg)

## 參考

- [Static Code Analysis of .NET Core Projects with SonarCloud](https://dotnetthoughts.net/static-code-analysis-of-netcore-projects/)
- [SonarQube - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/SonarQube)
- [Continuous Inspection | SonarQube](https://www.sonarqube.org/)

(fin)
