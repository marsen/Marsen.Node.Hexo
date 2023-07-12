---
title: "SQL Compatibility Level 對MsSQL時間查詢的影響"
date: 2017/09/18 11:30:13
tags:
  - Entity Framework
  - Database
  - MsSQL
  - 記錄
---

## 大綱

這次透過 Entity Framework(EF5)作了一個簡單的資料庫更新,  
恰巧的是這次更新的 Table 因為某些需求,使用 datetime 作為 Key 值.  
進而引發一連串的錯誤, 最後才找到 SQL Compatibility Level 對 MsSQL 時間查詢影響.

### 正式環境 SQL 版本 13.0.4422.0

### 程式碼

```csharp
public void UpdateBatchUploadData(BatchUploadDataEntity batchUploadDataEntity)
{
    using (WebStoreDBEntitiesV2 context = this.LifetimeScope.Resolve<WebStoreDBEntitiesV2>())
    {
        var item = (from batchUploadData in context.BatchUploadData.Valids()
                    where batchUploadData.BatchUploadData_Id == batchUploadDataEntity.BatchUploadData_Id
                    select batchUploadData).FirstOrDefault();

        this.MapBatchUploadData(batchUploadDataEntity, item);

        context.SaveChanges();
    }
}
```

如上面程式所示, `item` 是透過 Key 值 `BatchUploadData_Id` 取回來的物件.  
而 `MapBatchUploadData` 是一段簡單的程式碼,  
單純的將 `batchUploadDataEntity` 的值 mapping 到 item  
再呼叫 SaveChanges , 卻引發了 [dbupdateconcurrencyexception](<https://msdn.microsoft.com/en-us/library/system.data.entity.infrastructure.dbupdateconcurrencyexception(v=vs.103).aspx>)

### 錯誤畫面

![](https://i.imgur.com/8kBIYRr.jpg)

### 錯誤訊息

```
存放區更新、插入或刪除陳述式影響到非預期數目的資料列 (0)。
這些實體載入之後可能被修改或刪除了。請重新整理 ObjectStateManager 實體。
```

這一段訊息的意思就是: Entity Framework 預期更新了`0`筆資料，與它所預期的不符, 所以拋出錯誤。

### 原因

透過用**Sql Profiler**我們錄製到了以下的 SQL

```sql
exec sp_executesql N'update [dbo].[BatchUploadData]
set [BatchUploadData_StatusDef] = @0, [BatchUploadData_UpdatedTimes] = @1, [BatchUploadData_UpdatedDateTime] = @2
where (([BatchUploadData_Id] = @3) and ([BatchUploadData_CreatedDateTime] = @4))
select [BatchUploadData_Rowversion]
from [dbo].[BatchUploadData]
where @@ROWCOUNT > 0 and [BatchUploadData_Id] = @3 and [BatchUploadData_CreatedDateTime] = @4',N'@0 varchar(30),@1 tinyint,@2 datetime2(7),@3 bigint,@4 datetime2(7)',@0='ProcessFailed',@1=1,@2='2017-09-16 11:29:35.3720061',@3=52,@4='2017-09-05 18:53:36.3530000'
```

請注意到 **@4 datetime2(7) ... @4='2017-09-05 18:53:36.3530000'**  
如果將 `datetime2(7)` 改為 `datetime`或是將查詢語句改為 `@4='2017-09-05 18:53:36.353` 就能正確更新資料.

![](https://i.imgur.com/8pGTYL4.gif)

### 本機實測 (SQL 版本 12.0.4459.0)

透過本機寫了一小段的 SQL 作測試,  
![](https://i.imgur.com/iJntV1i.gif)  
**竟然不會有問題!!!**

這跟 SQL Compatibility Level 有關,  
mssql 2014 預設是 120, 2016 預設是 130,  
Datetime2 在 120 跟 130 的結果會不一樣.

### 解決方法

主要的查詢與更新 SQL 是 Entity Framework 產生的,  
所以我無法透過修改 SQL 的方式解決這個問題,  
而正式環境的 SQL Compatibility Level 調整將會牽一髮動全身  
且基於版本演進, 往新的版本靠攏是合理的選擇  
暫時的解法是透過修改 edmx ,  
不讓 datetime 作為整個 table 的 Key 值.  
較好的解法是升級 Entity Framework  
透過 Entity Framework 的機制, 指定查詢時間的精準度.  
實作的部份未來再補上.

## 參考資料

- [ALTER DATABASE (Transact-SQL) Compatibility](https://docs.microsoft.com/en-us/sql/t-sql/statements/alter-database-transact-sql-compatibility-level)
- [Change in datetime2 implementation in SQL Server 2016](https://social.msdn.microsoft.com/Forums/silverlight/en-US/de5dbf3e-8c95-40f4-9e31-b71f1f31983d/change-in-datetime2-implementation-in-sql-server-2016?forum=transactsql)
- [檢視或變更資料庫的相容性層級](https://docs.microsoft.com/zh-tw/sql/relational-databases/databases/view-or-change-the-compatibility-level-of-a-database)

(fin)
