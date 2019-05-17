---
title: "[實作筆記]查找與重建 Table 的索引值"
date: 2019/01/05 13:13:29
tag:
    - SQL
---

查詢指定 Table 目前的索引值

DBCC CHECKIDENT (yourtable)

Example

```sql
USE AdventureWorks2012;  
GO  
DBCC CHECKIDENT ('Person.AddressType');  
GO  DBCC CHECKIDENT (memberAccount)
```

重新建立你的索引值

DBCC CHECKIDENT (yourtable, reseed, new index)

```sql
USE AdventureWorks2012;  
GO  
DBCC CHECKIDENT ('Person.AddressType', RESEED, 10);  
GO  
```

## 參考

- [DBCC CHECKIDENT (Transact-SQL)](https://docs.microsoft.com/zh-tw/sql/t-sql/database-console-commands/dbcc-checkident-transact-sql?view=sql-server-2017)


(fin)