---
title: "使用 DateInterceptor 攔截SQL 語法"
date: 2017/09/24 10:55:30
tag:
  - Entity Framework
  - Database
  - MsSQL
---

## 前言
在[SQL Compatibility Level 對MsSQL時間查詢的影響](https://blog.marsen.me/2017/09/18/sql_compatibility_level_with_datetime2/)這篇文章裡,  
遇到了一個令難以處理的問題, 簡單的複述一下,  

1. 我的資料表以一個 datetime 欄位當作 PK
2. 我的SQL DB版本為 2016 以上 (Compatibility Level預設值為130)

這個時候我所有的 Entity Context 更新語法在呼叫 SaveChange 時會拋出錯誤;  
原因在於 EF 所產生的 SQL Query 語法 , 會以 PK 值作為查詢條件,   
而在這個時候 查詢時間的條件會精準到微秒(μs=1/1000000秒),  
但是 datetime 欄位只會記錄到豪秒(ms=1/1000秒),  
於是精準不足的部份就會補 0 ,  
EX:  
`2017-09-24 11:55:35.3720000` 與 `2017-09-24 11:55:35.372`  
這會導致查無資料進而引發 [dbupdateconcurrencyexception](https://msdn.microsoft.com/en-us/library/system.data.entity.infrastructure.dbupdateconcurrencyexception(v=vs.103).aspx)

## 解決方針
### 修改資料欄位
 將資料庫的欄位datatype datetime 改成 datetime2 ,  
 這或許是最理想的解法了, 你不需要更動程式碼,  
 而且會提昇你資料的時間精準度.  
 不過實務上,你必須考慮到你是否有dba的權限與即有的資料量,  
 當產品的核心功能依賴著這張表的時候且有巨量資料存在時,  
 更新欄位的資料型態的衝擊與風險或許是難以承受的.   
 更不用說還要考慮到整個 DB Server Cluster 的架構之類的問題.

### 修改 SQL Server Compatibility Level 從 130 至 120 
非常不建議的作法, 除了要考慮上述權限、衝擊與風險的問題外,  
如果降轉 Compatibility Level 
或許會使得其他有使用到 datetime2 型態的資料欄位發生異常,  
或是是失去原本精度的意義. 

### 使用 DateInterceptor 攔截SQL 語法
在考量上述兩種情況, 為了不增加~~DBA的工作量~~無謂的風險與權責問題, 
(其實是實務上我沒有DB Server的異動權限),  
我們可以透過 [IDbInterceptor](https://msdn.microsoft.com/en-us/library/system.data.entity.infrastructure.interception.idbcommandinterceptor(v=vs.113).aspx) 來欄截 Entity Framework 對DB 存取時執行的 Query

以下是個簡單的範例,

```csharp
    public class DateInterceptor : IDbInterceptor
    {
        public void ReaderExecuting(DbCommand command,
            DbCommandInterceptionContext<DbDataReader> interceptionContext)
        {
            var dateParameters = command.Parameters.OfType<DbParameter>()
                .Where(p => p.DbType == DbType.DateTime2);
            foreach (var parameter in dateParameters)
            {
                parameter.DbType = DbType.DateTime;
            }
        }
    }
```
 
我們實作了一個 IDbInterceptor 的類別, 
用來將 datetime2 的資料型別轉型成 datetime,     
接下要將它掛載在 Entity Context之中 

```csharp
    public partial class EF6Entities : DbContext
    {
        public EF6Entities()
            : base("name=EF6Entities")
        {
            DbInterception.Add(new DateInterceptor());
        }
    
        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            throw new UnintentionalCodeFirstException();
        }
    
        public virtual DbSet<BatchUploadData> BatchUploadData { get; set; }
    }
 ```
如此一來就會以 datetime 的精準度產生 SQL Query


 ## 參考   
- [程式碼](https://github.com/marsen/EFDemo)
- [IDbInterceptor](https://msdn.microsoft.com/en-us/library/system.data.entity.infrastructure.interception.idbcommandinterceptor.aspx)
- [IDbCommandInterceptor](https://msdn.microsoft.com/en-us/library/system.data.entity.infrastructure.interception.idbcommandinterceptor.aspx)
- [How to change how Entity Framework generates SQL precision for Datetime](https://stackoverflow.com/questions/46387565/how-to-change-how-entity-framework-generates-sql-precision-for-datetime)

(fin)