---
title: .NET PDB File 介紹
date: 2016/11/28 23:54:55
tags:
  - .Net Framework
---

### PDB(Program Database Files )

**.Net 原始碼在 Debug mode 建置後，將產生.pdb 檔案，其中記錄了.dll、.exe 與原始碼之間的對應關係。**

#### MSDN 這樣說

程式資料庫 (.pdb) 檔案也稱為符號檔，可將您在原始程式檔中為類別、  
方法和其他程式碼建立的識別項對應至專案之編譯的可執行檔中使用的識別項。

原文:

> _A program database (.pdb) file, also called a symbol file,_  
> _maps the identifiers that you create in source files for classes,_  
> _methods, and other code to the identifiers that are used in the_  
> _compiled executables of your project._

### PDB 記什麼

- 原始碼的檔案名稱 (source code file name)
- 變數與行號 (lines and the local variable names.)

### 偵錯工具搜尋 .pdb 檔案的順序？

1. DLL 或可執行檔內部指定的位置
2. 可能和 DLL 或可執行檔存在相同資料夾中的 .pdb 檔案。
3. 任何本機符號快取資料夾。
4. 任何在像是 Microsoft 符號伺服器 (如果啟用) 等位置指定的網路、網際網路或本機符號伺服器和位置。

### .pdb 檔案與.dll(或可執行檔)需要完全符合

偵錯工具只會載入與可執行檔建置時所建立的 .pdb 檔案完全相同之可執行檔的 .pdb 檔案  
\*相同原始碼在不同機器建置的執行檔與 PDB，偵錯工具將無法進行偵錯

### 看看.pdb 檔的內容

1. 開啟 %ProgramFiles(x86)%\Microsoft Visual Studio 14.0\DIA SDK\Samples\DIA2Dump\dia2dump.sln

2. 執行以下命令

```cmd
> Dia2Dump pdbfilepath >> dumpfileName.txt
> Dia2Dump C:\myproj\bin\debug\myproj.pdb >> myproj_dump.txt`
```

1. 輸出的結果大致如下

```text
Function       : In MetaData, [00001F16][0001:00001F16], len = 000000ED, GetBookList
Function attribute:
Function info:


//中略

Function: [0001F71A][0001:0001F71A] GetSalePage b_231
Function: [00001F16][0001:00001F16] GetBookList
Function: [0001F4F0][0001:0001F4F0] GetLayoutByAd b_0

//中略

** GetBookList

line 46 at [00001F16][0001:00001F16], len = 0x1	 D:\Repo\Mark_Lin\PROJ\WebAPI\Controllers\BookController.cs
line 47 at [00001F17][0001:00001F17], len = 0x6
line 48 at [00001F1D][0001:00001F1D], len = 0x1C
```

#### 參考

- http://anferneehardaway.pixnet.net/blog/post/6273453
- https://msdn.microsoft.com/zh-tw/library/ms241903(v=vs.100).aspx
- http://www.wintellect.com/devcenter/jrobbins/pdb-files-what-every-developer-must-know
- https://blogs.msdn.microsoft.com/jimgries/2007/07/06/why-does-visual-studio-require-debugger-symbol-files-to-exactly-match-the-binary-files-that-they-were-built-with/

(fin)
