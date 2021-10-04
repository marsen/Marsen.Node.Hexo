---
title: "[踩雷筆記] .Net專案升級 SpecRun.Runner 與調整 CI "
date: 2018/08/27 18:42:16
tag:
  - Unit Testing
  - Specflow
  - 踩雷筆記
---

## 前情提要

- 專案使用 Specflow 寫 BDD
- Visual Stuio 找不到測試所以才升級 SpecRun 與 Specflow
- Visual Studio 2017 升級到 15.8.1 後突然無法正常搜尋到測試
- SpecRun.Runner 升級(1.2 → 1.8.2)

去年12月跟 Visual Studio 折騰了許久，才讓專案的[測試項目重見光明](https://blog.marsen.me/2017/12/11/test_learn/vs2017_mstest_with_nuget/)，  
在今年與其它專案合併後，測試又從我們團隊的眼前消失了。

## 調整項目

升級 Specflow 與 SpecRun 相關套件
![升級 Specflow 與 SpecRun 相關套件](https://i.imgur.com/nwl6xFp.jpg)

取得 SpecRun 位於專案的 `packages` 資料夾中。
![取得 SpecRun 位於專案的 `packages` 資料夾中。](https://i.imgur.com/EyaFEFI.jpg)

## 設定 CI

由於 SpecRun 版本不符會導致 CI Job Error , 需要設定以新的 SpecRun 執行 CI

>D:\SpecRun.Runner.1.8.2\tools\SpecRun.exe run D:\Project\Core.Test\bin\Debug\Core.Test.dll /baseFolder:Core.Test\bin\Prod /toolIntegration:vs201 0 /reportFile:Core.Test.TestResult.html

## 參考

- VS2017的測試總管找不到測試項目，或僅有部分資料  
  說明：Something changed in the NUnit and XUnit Test Adapter.  
  But I am not sure, if it really worked before the Visual Studio update.  
  I think I saw this behavior already in earlier versions of Visual Studio and NUnit3/XUnit2. (此原因尚未被實證)

  - 解法1(暫解)：
        工具>擴充功能和更新>停用"Dotnet Extensions for Test Explorer"
  - 解法2(暫解):
        改用VS2015
  - 參考:
    - [Unable to fetch source Information for test method](https://developercommunity.visualstudio.com/content/problem/158160/unable-to-fetch-source-information-for-test-method.html)  
    - [Open SpecFlow test from Text Explorer goes to the feature class file not the feature file #990](https://github.com/techtalk/SpecFlow/issues/990)

- We could not find a data exchange file at the path System.ArgumentNullException
  - 說明: 聽說SpecFlow 2.2 fixed
  - 解法(暫解): Tools > Options > SpecFlow >Data Exchange Path，隨便設定一個路徑去 overwrite the default value
  - 參考:
    - [Build fails: We could not find a data exchange file](https://github.com/techtalk/SpecFlow/issues/1234)
    - [Visual Studio 2017 Generation error: Could not load file or assembly 'Microsoft.Build.Framework, Version=15.1.0.0](https://github.com/techtalk/SpecFlow/issues/857)
    - [Visual Studio Plugin for VS2017 does not always use Specflow Generator 2.2.0.0](https://github.com/techtalk/SpecFlow/issues/896)
    - [SpecFlow with SpecRun and NUnit fails with ArgumentNullException #639](https://github.com/techtalk/SpecFlow/issues/639)

(fin)
