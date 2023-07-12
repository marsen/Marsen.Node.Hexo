---
title: " [實作筆記]SQL Server 與 Linked Server 注意事項"
date: 2018/10/17 18:05:33
tags:
  - 實作筆記
  - SQL
  - Database
---

### 原始 SQL

```SQL=4
USE NationalDB
GO
BEGIN TRY
---Select Cross Linked Server
Select Name From LinkedServer.Shop.dbo.Member
---DO SOMETHING
END TRY
BEGIN CATCH
  DECLARE @errorMessage NVARCHAR(MAX) = ERROR_MESSAGE();
  RAISERROR (@errorMessage, 16, 1);
END CATCH
```

1. 變數先行宣告
2. 非必要不要宣告`MAX`
3. 跨 DB 存取不使用四節式的查詢語法，改用 `sp_executesql`

### 修改後的語法

```SQL=4
USE NationalDB
GO
DECLARE @errorMessage NVARCHAR(4000) = ERROR_MESSAGE();
BEGIN TRY
---Select Cross Linked Server
EXEC  LinkedServer.Shop.dbo.sp_executesql N'
        SELECT @Name = Name
        FROM dbo.Member
        WHERE MemberId = @MemberId
        AND Valid = 1',N'@MemberId bigint,@Name varchar(20) OUTPUT',@MemberId,@Name OUTPUT
---DO SOMETHING
END TRY
BEGIN CATCH
  RAISERROR (@errorMessage, 16, 1);
END CATCH
```

(fin)
