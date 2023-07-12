---
title: "[記錄]VS2015 StyleCop 誤判SA0102"
date: 2017/01/11 14:51:25
tags:
  - .Net Framework
  - StyleCop
---

## 環境

- 作業系統:Windows 10
- 開發工具:Visual Studio 2015 (Professional ver14.0.25431.01 Update 3)
- StyleCop 4.7.50.0

## 情境

當使用 C# 6 的`INTERPOLATION STRING`組合字串時，在 StyleCop 4.7 會誤判並回報 SA0102 的警告。

```csharp
return new ExcelResult
{
  Data = exportData,
  FileName = $"{shopId}_Code_{DateTime.Now.ToString("yyyyMMddHHmm")}",
  SheetName = "ECouponCode"
};
```

## 修正目標

1. 將 StyleCop 4.7 更新至 StyleCop 5.0
2. 編輯客製化的 StyleCop Rules(直接將 4.7 的設定檔覆蓋 VS 會報錯)

## 修正記錄

1.  移除 StyleCop
    控制台 > 新增移除程式
    ![](https://i.imgur.com/PCYZxIK.png)

2.  移除 Visual Studio 上的 StyleCop
    工具 > 擴充功能和更新
    ![](https://i.imgur.com/1HdVoko.png)

3.  自你的程式碼移除所有對 StyleCop 的參考
    `我的程式碼中並不包含對StyleCop的參考，故略過此步驟`

4.  搜尋`StyleCop*.dll` 並且移除(非必要)

5.  重新安裝 [StyleCop Visual Studio Extension](https://marketplace.visualstudio.com/items?itemName=ChristopheHEISER.VisualStyleCop) 1. 安裝完成檢視安裝記錄檔
    ![](https://i.imgur.com/Kny5aU9.png)

        2.  可以透過安裝記錄檔取得安裝路徑

    ![](https://i.imgur.com/edSDREN.png)

6.  客製修改 StyleCop 設定檔
    5-2 的步驟可以找到`Settings`子資料夾，內含預設 StyleCop 設定檔，
    複制這個檔案到上一層，重新命名為`Settings.StyleCop`,
    透過`StyleCop.SettingsEditor`開啟`Settings.StyleCop`調整設定值。
    ![](https://i.imgur.com/DJjoEYJ.gif)

## 參考

- [Github StyleCop](https://github.com/StyleCop)
- [Github Visual-StyleCop](https://github.com/Visual-Stylecop/Visual-StyleCop/wiki)
- [Marketplace Visual StyleCop ](https://marketplace.visualstudio.com/items?itemName=ChristopheHEISER.VisualStyleCop)

(fin)
