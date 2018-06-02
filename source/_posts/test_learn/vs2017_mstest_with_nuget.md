---
title: "Visual Studio 2017 MSTest Framework 異常修正"
date: 2017/12/11 11:17:08
tag:
  - MsTest
  - Visual Studio
---
## 應該要知道的事

- 這是踩雷筆記
- 2017的筆記可能會隨時間變得沒有參考價值
- Visual Studio 2017的問題,並不一定適用其他版本
- 最後面會有不定時補充

## 情境

![載入測試時發生例外狀況](https://i.imgur.com/FDDtc9V.jpg)
原本使用 Visual Studio 2015 建立的測試專案,
升級到 Visual Studio 2017 後, 發生以下錯誤
```
[2017/12/11 上午 02:09:59 Error] 測試探索程式 'SpecRunTestDiscoverer' 
載入測試時發生例外狀況。例外狀況: 無法載入檔案或組件 
'Microsoft.VisualStudio.QualityTools.UnitTestFramework,
Version=10.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a'
或其相依性的其中之一。 系統找不到指定的檔案。
```

## 導致結果

![原本的測試數量為1942,變成459,遺失了7成5的測試案例.](https://i.imgur.com/2REPRzG.jpg)
1. 測試專案會找不到測試,或是測試數量不正確.
2. 可以使用 Visual Studio 2015 重新執行探測索測試,即可排除問題.

## VS2015 已移除或未安裝該怎麼辦？

透過MsTest直接加入 
`Microsoft.VisualStudio.QualityTools.UnitTestFramework` 
的參考已經是舊的方法了, 
#### 在 vs2017 建議的解決方案如下:
* 移除方案中所有對 `Microsoft.VisualStudio.QualityTools.UnitTestFramework` 的參考
* 透過 Nuget 安裝 MSTest.TestAdapter 
* 透過 Nuget 安裝 MSTest.TestFramework
* 關閉 vs2017
* 移除 `%temp%\VisualStudioTestExplorerExtensions`內所有檔案
* 重啟 vs2017 並建置以觸發探索測試
![透過 Nuget 安裝 MSTest.TestAdapter/MSTest.TestFramework](https://i.imgur.com/RPI77KN.jpg)
![重啟 vs2017 並建置以觸發探索測試](https://i.imgur.com/JQ7zf2S.jpg)

## 參考
- [Unit test fail - cannot load Microsoft.VisualStudio.TestPlatform.TestFramework.Extensions](https://developercommunity.visualstudio.com/content/problem/14673/unit-test-fail-cannot-load-microsoftvisualstudiote.html)

(fin)

## 補充 
- 2018/06/02 : 
visual studio 2017 15.7.* 的版本之後 ,
`%temp%\VisualStudioTestExplorerExtensions` 消失了 ,
不過正常情況建置後 , visual studio 探索測試仍然可以正確找到測試。
