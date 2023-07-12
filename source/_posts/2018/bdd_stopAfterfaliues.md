---
title: "[踩雷筆記] BDD 跑3個失敗就停止"
date: 2018/11/19 11:40:45
tags:
  - Testing
  - 踩雷筆記
---

## 原文描述

> Visual Studio 的 Test Explorer 在跑測試，
> 如果遇到失敗，會直接中斷，
> 比如說有 2000 個測試案例 跑到第 1300 個時測試失敗就會中斷
> 剩下的 700 個案例會不執行直接略過。

## 情境

### 實際狀況

並不是一跑到錯誤就會中斷測試，而是跑了 3 個失敗後中斷測試，
另外執行的測試專案，是使用 SpecRun.Runner 撰寫 Cucumber 語法跑 BDD 測試。

### 原因

當使用測試專案使用 SpecRun.Runner，預設 stopAfterFailures 是 3，
所以 3 個失敗就會略過。

### 更多

正常的測試都是會全跑完的。因為 Test Framework 標示 Test Method 的部份，  
都是幫你內建 Try/Catch 攔下所有 Exception 的。

MsTest 的 [TestMethod] 或是 NUnit 的[Test](XUnit 是 [Fact])，其實就是層 Wrapper  
你在方法裡面寫的內容，只要有引發 Exception 的例外，
都會被這層 Wrapper 攔下來，轉成測試結果的內容。

## 解決方案

測試專案根目錄應該有一份 `Default.srprofile` 檔，如下
將 `stopAfterFailures` 調整至適當的值(ex:500)，將會顯示所有失敗測試。
本案例開啟後，實際失敗的測試有 37 個

```xml
  <Execution stopAfterFailures="3" testThreadCount="1" testSchedulingMode="Sequential" />
```

## 參考資料

- [SpecRun skipping testing rather than running them when executing more than 16](https://groups.google.com/forum/#!topic/specrun/yR6VVH8bDKg)
- [SpecFlow+ Runner Profiles](https://specflow.org/plus/documentation/SpecFlowPlus-Runner-Profiles/)

特別感謝 91、Green、余小章的協助

(fin)
