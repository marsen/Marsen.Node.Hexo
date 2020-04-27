---
title: "[實作筆記] Github 結合 SonarCloud 作代碼質量檢查"
date: 2020/04/27 17:01:46
---

## 前情提要

大概一年前我曾寫過一篇 Blog [[實作筆記] 讓 SonarQube 檢查你的代碼](https://blog.marsen.me/2019/05/16/2019/opensource_with_sonarcloud/)，  
沒什麼含金量，只是我個人用來記錄的筆記。  
當初有一些問題沒有排除，加上工作一忙就沒有後續了。  

我的理想目標是，每當我上 Code 到線上 Repo 時(Github)，  
SonarCloud 可以幫我檢查代碼，跑跑測試覆蓋率，刷新一下 Budget，  
如果有異常(覆蓋率下降、壞味道等…)最好再發個通知給我。
這些功能要怎麼作到呢 ?

然後我會實際用在我的[SideProject](https://github.com/marsen/Marsen.NetCore.Dojo)上，  
這個 Project 單純只是為了練習而生，  
專注於我個人的測試項目，主要語言為 Csharp 也有一些 TypeScript 。

## 分析

首先先排序一下優先序吧。

1. 一定要可以執行代碼檢查
2. 要能結合 CI ，我以 Github Action 作為我的主要 CI 工具
3. 能夠跑測試並輸出測試報告
4. Buget 刷新
5. 發通知

這篇主要會說明如何結合 CI 執行代碼檢查。  
執行代碼檢查就發現有一個問題， SonarCloud 一次只能對一種語言作檢查，  
雖然我的專案裡有兩種語言，但是以 C# 佔大宗(93%)，所以調整一下目標，  
先優先完成 C# 的代碼檢查、結合 CI 與輸出測試報告。  
之後再進行 Typescript 的檢查與測試，最後通知有的話很棒，沒有也沒關係啦 :)。  

## 本機執行代碼檢查

我本機的環境有兩個，一個是 Windows 一個 macOS，我這裡只討論 Windows 的作法，  
然後我就要直上 CI 了，Github Action 我並不熟悉，但是我知道上面應該是執行 Linux like 的作業系統。  

首先要在 SonarCloud 上建立 Project ，  
可以參考[Get started with GitHub.com](https://sonarcloud.io/documentation/integrations/github/)快速建立。  

Administrator > Analysis Method  

![Analysis Methodn](/images/2020/4/sonarqube_run_with_github_action_02.jpg)  

這裡要把 SonarCloud Automatic Analysis 的功能關掉。  
SonarCloud 支援自動分析語言只有以下

ABAP, Apex, CSS, Flex, Go, HTML, JS, Kotlin, PHP, Python, Ruby, Scala, Swift, TypeScript, XML.  

雖然有很多，但可惜並沒有 C# ，所以要先關掉，不然 Github Action 執行時會收到下面的錯誤。

`You are running CI analysis while Automatic Analysis is enabled. Please consider disabling one or the other.`

另外目前支援的 CI 服務有 Circle CI 與 Travis CI ，  
一樣殘念的是沒有支援 Github Action 。  
另外兩個選項目是 Other CI 與 Manunlly(手動) 。  
我的前一篇文章就是使用手動的方式把檢查報告打到 SonarCloud。  
雖然只隔一年，但 UI 介面上已經有些差距，我還是再作一次介紹。

![Analysis Methodn](/images/2020/4/sonarqube_run_with_github_action_01.jpg)

首先先下載 SonarScanner，選擇正確的語言(Others)與OS(Windows)後下載，
接著設定環境變數  

![Setting Path](/images/2020/4/sonarqube_run_with_github_action_03.jpg)  

最後開啟 CMD 切換到專案目錄底下後。
執行語法，如果照著上述步驟，你可以在 Download 的按鈕下方找到語法，同時它會幫你填好 Token。  

Begin

```shell
dotnet sonarscanner begin /k:"$ProjectKey" /o:"$Oragnization" /d:"sonar.host.url=https://sonarcloud.io" /d:"sonar.login="$Sonar_Login
```

MSBuild

```shell
dotnet build ".\Marsen.NetCore.Dojo.Integration.Test.sln"
```

End

```shell
dotnet sonarscanner end /d:"sonar.login="$Sonar_Login
```

`$ProjectKey` 與 `$Oragnization` 這兩個變數可以在 SonarCloud 的 Overview 介面的右下角找到，  
`$Sonar_Login` 則可以透過 [Security](https://sonarcloud.io/account/security) 設定。

執行命名完成後，大概幾秒內就可以在 SonarCloud 中看到結果了。

簡單總結一下

1. 你要有 SonarCloud
2. 要下載 SonarSanner
3. 依序執行 Begin > MSBuild > End

另外有一些雷包，在這裡也記錄一下

- 要安裝 Java (Java8)
- 執行語法的目錄底下不能有`sonar-project.properties`
  - 不然會報錯 (sonar-project.properties files are not understood by the SonarScanner for MSBuild.)
  - 我覺得應該是我的檔案內容有誤，但是還不知道怎麼修正。總之直接移除對我來說是可以 work 的。  

## 參考

- [Get started with GitHub.com](https://sonarcloud.io/documentation/integrations/github/)

(fin)
