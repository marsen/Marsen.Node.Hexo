---
title: "[實作筆記] Github 結合 SonarCloud 作代碼質量檢查 - 測試覆蓋率篇"
date: 2020/04/30 15:30:43
tag:
  - CI/CD
  - 實作筆記
---

## 前情提要

承[上篇](https://blog.marsen.me/2020/04/27/2020/sonarqube_run_with_github_action/),
我已經將 SonarCloud 的代碼檢查與 Github Action 結合在一起了。  
透過專案首頁的 Dashboard 與專案的 Budget 我可以知道目前專案的一些狀況，  
壞味道、技術債等 ...

### 題外話，測試分類

在我的專案中有兩種測試，單元測試與整合測試。  
我的分類方法，單一類別的 public 方法就用單元測試包覆。
不同的類別之如果有組合的交互行為，就用整合測試包覆。

舉個簡單的例子，以購物流程來說，我有的類別如下 :

- Cart(購物車)
  - Add (加入商品)
  - Substract (移除商品)
  - CheckOut (結帳)
- Order (訂單)
  - Caculate (計算結帳金額)

另外有兩種類別如下，

- ECoupon
  - Caculate
- Promotion
  - Caculate

上面的所有方法我都會加上單元測試作保護，
但是 Order 在呼叫 Caculate 的時候，
會以不同的順序組合 ECoupon.Caculate 與 Promotion.Caculate，
這個時候就有可能會產生不同的結果。

## 執行測試

同上一篇，整體的流程我們只要在 Build 完後加上測試即可。

1. Begin
2. MSBuild
3. **Test**
4. End

開發環境執行測試

```shell
dotnet test ./test/Marsen.NetCore.Dojo.Tests/Marsen.NetCore.Dojo.Tests.csproj
```

產生測試報告

```shell
dotnet test ./test/Marsen.NetCore.Dojo.Tests/Marsen.NetCore.Dojo.Tests.csproj --logger:trx;LogFileName=result.trx
```

測試報告會產生一份 `result.trx` 檔，在測試專案目錄底下的 `TestResults` 資料夾裡。
如果要在 SonarCloud 上使用請設定 sonar.cs.vstest.reportsPaths (VSTest 適用)，更多資訊請[參考](https://docs.sonarqube.org/latest/analysis/coverage/)。

![result.trx](/images/2020/4/sonarqube_run_with_github_action_04.jpg)

```shell
dotnet sonarscanner begin /k:"Marsen.NetCore.Dojo" /o:"marsen-github" /d:"sonar.host.url=https://sonarcloud.io" /d:"sonar.login="$SONAR_LOGIN
dotnet build ".\Marsen.NetCore.Dojo.Integration.Test.sln"
dotnet test ./test/Marsen.NetCore.Dojo.Tests/Marsen.NetCore.Dojo.Tests.csproj --logger:trx;LogFileName=result.trx
dotnet sonarscanner end /d:"sonar.login="$SONAR_LOGIN
```

## 覆蓋率

1. Begin
2. MSBuild
3. **Test -產生報告**
4. **Test -覆蓋率**
5. End

這裡實作上很簡單，但是在選擇上有一些困難與試誤，稍微作個記錄。

1. dotconver (棄選)

   - 似乎要綁定 resharper 的 lincese
   - 有 30 天的限制，不知道會不會影響功能
   - 不知道怎麼用 commandline 下載執行程式至 Github Action 執行實體上。

2. vstest.console.exe (棄選)

   - ~~似乎只能在 windows 上執行~~
   - 可以透過參數開始功能 `--collect:"Code Coverage"`，但產生的 .cover 檔 SonarCloud 不支援需要轉換格式
   - 不知道如何將 .cover 轉換為 .coverxml ， 可能是 visual studio enter prise 才有的功能 ?

3. opencover (coverlet)
   - ~~`/p:CoverletOutputFormat=opencover /p:CoverletOutput=./coverage/` 的語法不 Work~~
   - `--collect:"XPlat Code Coverage"` 是比較新的參數用法，可以使用設定檔

因諸多原因，我最後選擇了 Opencover(Coverlet) 的作法，
記錄步驟如下，

首先要在測試專案上安裝 Nuget 套件 `coverlet.collector`。
然後執行以下語法產生覆蓋率報告。

```shell
dotnet test ./test/Marsen.NetCore.Dojo.Tests/Marsen.NetCore.Dojo.Tests.csproj --settings coverage.xml
```

coverage.xml 設定檔如下，這個檔案必須是 xml 檔，檔名並沒有限制 :  
Configuration > Format 請記得填寫 `opencover`

```xml
<?xml version="1.0" encoding="utf-8" ?>
<RunSettings>
  <DataCollectionRunSettings>
    <DataCollectors>
      <DataCollector friendlyName="XPlat code coverage">
        <Configuration>
          <Format>opencover</Format>
        </Configuration>
      </DataCollector>
    </DataCollectors>
  </DataCollectionRunSettings>
</RunSettings>
```

最後至 `SonarCloud` 上設定 `sonar.cs.opencover.reportsPaths` 的路徑

![result.trx](/images/2020/4/sonarqube_run_with_github_action_05.jpg)

最後的最後，就來個 Budget 大集合吧

[![Bugs](https://sonarcloud.io/api/project_badges/measure?project=Marsen.NetCore.Dojo&metric=bugs)](https://sonarcloud.io/dashboard?id=Marsen.NetCore.Dojo)
[![Code Smells](https://sonarcloud.io/api/project_badges/measure?project=Marsen.NetCore.Dojo&metric=code_smells)](https://sonarcloud.io/dashboard?id=Marsen.NetCore.Dojo)
[![Coverage](https://sonarcloud.io/api/project_badges/measure?project=Marsen.NetCore.Dojo&metric=coverage)](https://sonarcloud.io/dashboard?id=Marsen.NetCore.Dojo)
[![Duplicated Lines (%)](https://sonarcloud.io/api/project_badges/measure?project=Marsen.NetCore.Dojo&metric=duplicated_lines_density)](https://sonarcloud.io/dashboard?id=Marsen.NetCore.Dojo)
[![Lines of Code](https://sonarcloud.io/api/project_badges/measure?project=Marsen.NetCore.Dojo&metric=ncloc)](https://sonarcloud.io/dashboard?id=Marsen.NetCore.Dojo)
[![Maintainability Rating](https://sonarcloud.io/api/project_badges/measure?project=Marsen.NetCore.Dojo&metric=sqale_rating)](https://sonarcloud.io/dashboard?id=Marsen.NetCore.Dojo)
[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=Marsen.NetCore.Dojo&metric=alert_status)](https://sonarcloud.io/dashboard?id=Marsen.NetCore.Dojo)
[![Reliability Rating](https://sonarcloud.io/api/project_badges/measure?project=Marsen.NetCore.Dojo&metric=reliability_rating)](https://sonarcloud.io/dashboard?id=Marsen.NetCore.Dojo)
[![Security Rating](https://sonarcloud.io/api/project_badges/measure?project=Marsen.NetCore.Dojo&metric=security_rating)](https://sonarcloud.io/dashboard?id=Marsen.NetCore.Dojo)
[![Technical Debt](https://sonarcloud.io/api/project_badges/measure?project=Marsen.NetCore.Dojo&metric=sqale_index)](https://sonarcloud.io/dashboard?id=Marsen.NetCore.Dojo)
[![Vulnerabilities](https://sonarcloud.io/api/project_badges/measure?project=Marsen.NetCore.Dojo&metric=vulnerabilities)](https://sonarcloud.io/dashboard?id=Marsen.NetCore.Dojo)
[![SonarCloud](https://sonarcloud.io/images/project_badges/sonarcloud-black.svg)](https://sonarcloud.io/dashboard?id=Marsen.NetCore.Dojo)
[![Quality gate](https://sonarcloud.io/api/project_badges/quality_gate?project=Marsen.NetCore.Dojo)](https://sonarcloud.io/dashboard?id=Marsen.NetCore.Dojo)

## 參考

- [Test Coverage & Execution](https://docs.sonarqube.org/latest/analysis/coverage/)
- [Docker 教學 - .NET Core 測試報告 (Coverlet + ReportGenerator)](https://blog.johnwu.cc/article/docker-dotnet-coverage-report-generator.html)
- [Coverlet](https://discoverdot.net/projects/coverlet)
- [Collecting test coverage using Coverlet and SonarQube for a .net core project](https://medium.com/agilix/collecting-test-coverage-using-coverlet-and-sonarqube-for-a-net-core-project-ef4a507d4b28)
- [Code Coverage in .NET Core Projects](https://codeburst.io/code-coverage-in-net-core-projects-c3d6536fd7d7)

(fin)
