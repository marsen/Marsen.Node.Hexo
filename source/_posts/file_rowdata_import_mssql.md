---
title: 匯入文字資料到MSSQL
tag:
- DBA
- MSSQL
- 記錄
---
## 前置作業
1. 一個有權限寫入的MSSQLServer與SSMS管理工具
2. 準備好你的檔案資料(Row Data)
3. 這份記錄僅供參考(DB版本使用Microsoft SQL Server 2014)

## 步驟記錄
1. 開啟SSMS，連線測試SQL Server
2. 對測試資料庫右鍵>工作>匯入資料，開啟「SQL Server匯入匯出精靈」
3. 「開始畫面」> 下一步
4. 「資料來源」選擇一般檔案來源，「檔案名稱」選取Row Data的檔案路徑
5. 請依實際狀況選擇下列欄位
     - 「略過的標頭資料列數」: 預設為0，可透過設定此欄位略過 Row Data內含註解，標頭等資料
     - 「第一個資料列的資料行名稱」:若Row Data第一行為標頭，請勾選此欄位
     - 左側選單選擇「資料行」>「資料列分隔符號」選擇「{CR}{LF}」、「資料行分隔符號」選擇「Tab鍵{t}」
6. 左側選單選擇「預覽」確認匯入的資料無誤後，點擊下一步
7. 「目的地」請選擇「SqlServer」，設定ConnectionString
     - Data Source=1**.***.***.***;Initial Catalog=TestDB;Persist Security Info=True;User ID=tester;Password=********;
8. 「選取來源資料表和檢視畫面」確認無誤後，點擊下一步。
9. 「檢閱資料類型對應畫面」請忽略資料類型的警告，點擊下一步。
10. 「執行封裝畫面」勾選「立即執行」，點擊下一步。
11. 「完成精靈畫面」點選完成。
12.  執行時間長短，依Row Data大小而有所差異。

(fin)
