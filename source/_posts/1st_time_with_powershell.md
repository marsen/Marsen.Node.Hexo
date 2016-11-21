---
title: "[記錄]powershell 初體驗"
date: 2016/10/21 13:27:53 
tag:
  - powershell
  - Database
  - MsSQL
  - BulkInsert
  - 記錄
---
## 需求
1. 將指定的Log記錄,匯入資料庫,產生row data
2. 將row data 轉換成為需要的報表資料
3. 產生報表

## 規劃
1. powershell 讀取檔案
2. powershell 連接資料庫
3. powershell 執行SQL
4. powershell 作BulkInsert
5. powershell 寫入檔案

![](/images/102216_095355_PM.jpg)

**簡記要點**
- ***powershell 可以直接取用 .Net Framework 或 COM 元件***
- ***宣告變用要用`$`字號***
- ***`#` 是註解***

### 讀取檔案
```powershell
#用New-Object 建立.Net StreamrReader 物件
$reader = New-Object System.io.streamreader(get-item $filePath)
#使用`[]`建立靜態類別讀取檔案
$file = [System.IO.File]::ReadAllLines($filePath)  
#直接使用Get-Content讀取文檔
$file = Get-Content  "C:\filepath\file"
```
### 連線資料庫與執行語法
```powershell
$connection = New-Object System.Data.SQLClient.SQLConnection
$connection.ConnectionString = "server='$server';database='$database';uid='$user'; pwd='$pwd';Integrated Security=False;"
$connection.Open()
# do something 
$connection.Close()
```
### BulkInsert
- 從檔案建立DataTable

```powershell
$table = New-Object System.Data.DataTable
#建立欄位
$col_title = New-Object system.Data.DataColumn "Title",([string])
$table.Columns.Add($col_title);
$col_content = New-Object system.Data.DataColumn "Content",([string])
$table.Columns.Add($col_content);
$col_author = New-Object system.Data.DataColumn "Author",([string])
$table.Columns.Add($col_author);
#建立資料
foreach($file in $files){
  $dr = $table.NewRow();
  $dr["Title"] = $file["title"]
  $dr["Content"] = $file["content"]
  $dr["Author"] = $file["author"]
}
#寫入資料表
$table.Rows.Add($dr);
```

- 透過BulkCopy將DataTable寫入資料庫 

```powershell
$connection.Open()
$bulkCopy = New-Object (“Data.SqlClient.SqlBulkCopy”) -ArgumentList $connection
$bulkCopy.DestinationTableName = "tablename"
$bulkCopy.WriteToServer($datatable)
$connection.Close()
```

### 進度條
```powershell
Write-Progress -Activity "BulkInsert" -Status "載入百分比: 100 %" -PercentComplete 100;
```

### 產生報表

```powershell
$datatable | export-csv C:\Reports\20161026.csv -Encoding UTF8
```



## 參考
1. https://msdn.microsoft.com/en-us/powershell
2. https://msdn.microsoft.com/en-us/powershell/scripting/getting-started/cookbooks/using-static-classes-and-methods
3. https://cmatskas.com/execute-sql-query-with-powershell/
4. https://blogs.technet.microsoft.com/heyscriptingguy/

(fin)