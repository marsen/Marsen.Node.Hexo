---
title: "[實作筆記]SQL Server 與 Linked Server 注意事項"
date: 2018/10/17 18:05:33
tag:
  - SQL
  - Database
---
1. 變數先行宣告

```SQL=5
BEGIN TRY
END TRY
BEGIN CATCH
  DECLARE @errorMessage NVARCHAR(MAX) = ERROR_MESSAGE();
  RAISERROR (@errorMessage, 16, 1);
END CATCH
```
