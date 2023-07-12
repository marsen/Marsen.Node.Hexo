---
title: " [實作筆記] ASP.NET 專案部署地雷-消失的靜態檔"
date: 2019/04/12 16:38:16
tags:
  - .Net Framework
  - 實作筆記
---

## 應該知道的事

- 這個是 Debug 的筆記
- 用的是 .Net Framework 4.6 不是 .Net Core
- 對你可能沒有幫助

## 問題

方案裡面有三個 Web 專案 Web1 、Web2 、Web3,
因開發某功能需要加入一個文字靜態檔 `Iamfile.txt`  
在部署的時候卻無法將檔案部署至網站根目錄。

## 誤解

對檔案按右鍵 > 屬性 > 複製的輸出目錄 > 下拉選取一律複製。

![複製的輸出目錄](/images/2019/4/copy_file_to_bin.jpg)

很可惜，這個設定的調整會讓這個檔案在**建置**的時候輸出到指定資料夾中( Ex: \bin )，  
這個操作會影響 `csproj` 如下:

```xml
    <None Include="Iamfile.txt">
    +  <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
```

## 正解

注意 Tag 的名稱為 `None`，需要調整為 `Content`，
目前不確定如何用 IDE 操作。

```xml
    <Content Include="Iamfile.txt" />
```

## 補充

找到你的 MSBuild 版本

```sh
cd C:\Program Files (x86)\Microsoft Visual Studio\2017\Professional\MSBuild\15.0\Bin 
```

執行建置

```sh
λ MSBuild.exe D:\Projects\IsASolution.sln /p:Configuration=QA;DeployOnBuild=true;PublishProfile=Mall.QA.pubxml;MvcBuildViews=false;AutoVersion=True
```

執行後輸出的位置需要看你的部署檔 `*.pubxml` 如下範例  
可以在 `D:\Archives\QA\IsASolution` 找到我的輸出。

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--
此檔案是由您 Web 專案的發行/封裝處理程序所使用。您可以編輯此 MSBuild 檔案，
以自訂此處理程序的行為。若要深入了解，請造訪 http://go.microsoft.com/fwlink/?LinkID=208121。
-->
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <WebPublishMethod>FileSystem</WebPublishMethod>
    <LastUsedBuildConfiguration>QA</LastUsedBuildConfiguration>
    <LastUsedPlatform>Any CPU</LastUsedPlatform>
    <SiteUrlToLaunchAfterPublish />
    <ExcludeApp_Data>False</ExcludeApp_Data>
    <publishUrl>D:\Archives\QA\IsASolution</publishUrl>
    <DeleteExistingFiles>True</DeleteExistingFiles>
    <PrecompileBeforePublish>True</PrecompileBeforePublish>
    <EnableUpdateable>True</EnableUpdateable>
    <DebugSymbols>False</DebugSymbols>
    <WDPMergeOption>DonotMerge</WDPMergeOption>
  </PropertyGroup>
</Project
```

(fin)
