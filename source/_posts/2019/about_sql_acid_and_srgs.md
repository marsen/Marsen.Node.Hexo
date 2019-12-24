---
title: "[學習筆記] SQL 大小事 Isolation Level 與 SARGs"
date: 2019/12/25 02:25:55
tag:
 - Database
 - SQL 
---

## ACID 與 Isolation Level

### 什麼是 ACID

- Atomicity（原子性）：
    一個交易（transaction）中的所有操作，要嘛全部完成，要嘛全部不完成，  
    不會在中間某個環節中斷。交易在執行過程中發生錯誤，會被回滾（Rollback）到交易開始前的狀態，  
    就像這個交易從來沒有執行過一樣。即，交易不可分割、不可約簡。  
- Consistency（一致性）：
    在交易開始之前和交易結束以後，資料庫的完整性沒有被破壞。  
    這表示寫入的資料必須完全符合所有的預設約束、觸發器、級聯回滾等。  
- Isolation（隔離性）：
    資料庫允許多個並發交易同時對其數據進行讀寫和修改的能力，  
    隔離性可以防止多個交易並發執行時由於交叉執行而導致數據的不一致。  
    交易隔離分為不同級別，  
    包括未提交讀（Read uncommitted）、  
    提交讀（read committed）、  
    可重複讀（repeatable read）和串行化（Serializable）。
- Durability（持久性）：
    交易處理結束後，對數據的修改就是永久的，即便系統故障也不會丟失。

### 問題，欄位 price 現在為 100，兩筆交易同時發生，A 交易為 price+10, B 交易為 price-15 請問最終的值為何

MsSQL 預設的 Isolation Level 為 READ COMMITTED

![Isolation Level](https://miro.medium.com/max/1438/1*zAaBTdOtFJzISu4KGcwtvQ.png)

### 情境題 & SQL

#### 建立測試資料

```sql
CREATE TABLE [dbo].[ACIDSample](
    [ACIDSample_Id] [bigint] IDENTITY(1,1) NOT NULL,
    [ACIDSample_Name] [nvarchar](100) NULL,
    [ACIDSample_Price] [Decimal] NULL)

INSERT INTO [dbo].[ACIDSample]
           ([ACIDSample_Name]
           ,[ACIDSample_Price])
     VALUES
        ('Toy',150),
        ('Shoes', 120)

GO
```

#### Read Committed(MsSQL 預設值) vs Read Uncommitted

差別在於會不會讀到髒資料，
Read Uncommitted 不會使用 locked 來避免未 commit 的資料被讀取，
參考以下範例:

假設目前 ACIDSample_Price = 100

### Sample READ COMMITTED

Session 1，執行 Update 後延遲 10 秒後 Rollback

```sql
BEGIN TRANSACTION
UPDATE ACIDSample
SET ACIDSample_Price = ACIDSample_Price + 10
WHERE ACIDSample_Id = 1

WaitFor Delay '00:00:10'
Rollback Transaction
```

Session 2，
設定 TRANSACTION ISOLATION LEVEL 為 READ COMMITTED，  
目前 ACIDSample_Price = 100  
執行查詢，在 READ COMMITTED 的情境況下，會等待延遲結束後取得資料  

```sql
-- SET TRANSACTION ISOLATION LEVEL
SET TRANSACTION ISOLATION LEVEL READ COMMITTED
SELECT ACIDSample_Price FROM ACIDSample WHERE ACIDSample_ID = 1
```

### Sample READ UNCOMMITTED

Session 1，與前一個範例相同，執行 Update 後延遲 10 秒後 Rollback  

```sql
BEGIN TRANSACTION
UPDATE ACIDSample
SET ACIDSample_Price = ACIDSample_Price + 10
WHERE ACIDSample_Id = 1

WaitFor Delay '00:00:10'
Rollback Transaction
```

Session 2，執行查詢，在 READ UNCOMMITTED 的情境況下，  
會立即取得未 Commit 的資料(Dirty Data)，不需要等待10秒  

```sql
-- SET TRANSACTION ISOLATION LEVEL
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED
SELECT ACIDSample_Price FROM ACIDSample WHERE ACIDSample_ID = 1
```

### Sample Repeatable Read

Session 1  
設定 TRANSACTION ISOLATION LEVEL 為 Repeatable Read
執行以下SQL後，立即執行 Session 2 的 SQL，
等待 10 秒(Delay)後執行 Session 3 的 SQL

```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ
BEGIN TRANSACTION
SELECT * FROM ACIDSample

WaitFor Delay '00:00:10'
SELECT * FROM ACIDSample
WaitFor Delay '00:00:10'
SELECT * FROM ACIDSample
COMMIT Transaction
```

Session 2 新增一筆資料

```sql
INSERT INTO ACIDSample (ACIDSample_Name,ACIDSample_Price)
VALUES ('Cat',200)
```

Session 3 刪除一列資料

```sql
DELETE ACIDSample WHERE ACIDSample_Name = 'Cat'
```

等待 Delay 結束後，可以看到查詢結果如下
第2次的查詢會與第3次一致，但是實際上的資料已經被刪除了

```text
ACIDSample_Id        ACIDSample_Name                                                                                      ACIDSample_Price
-------------------- ---------------------------------------------------------------------------------------------------- ---------------------------------------
1                    Toy                                                                                                  132
2                    Shoes                                                                                                120

(2 rows affected)

ACIDSample_Id        ACIDSample_Name                                                                                      ACIDSample_Price
-------------------- ---------------------------------------------------------------------------------------------------- ---------------------------------------
1                    Toy                                                                                                  132
2                    Shoes                                                                                                120
7                    Cat                                                                                                  200

(3 rows affected)

ACIDSample_Id        ACIDSample_Name                                                                                      ACIDSample_Price
-------------------- ---------------------------------------------------------------------------------------------------- ---------------------------------------
1                    Toy                                                                                                  132
2                    Shoes                                                                                                120
7                    Cat                                                                                                  200

(3 rows affected)
```

試著把 Session 的 ISOLATION LEVEL 調成 READ COMMITTED

Session 1 (READ COMMITTED)

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED
BEGIN TRANSACTION
SELECT * FROM ACIDSample

WaitFor Delay '00:00:10'
SELECT * FROM ACIDSample
WaitFor Delay '00:00:10'
SELECT * FROM ACIDSample
COMMIT Transaction
```

結果在同一筆交易內會相同的會查詢到不同的結果

```text
ACIDSample_Id        ACIDSample_Name                                                                                      ACIDSample_Price
-------------------- ---------------------------------------------------------------------------------------------------- ---------------------------------------
1                    Toy                                                                                                  132
2                    Shoes                                                                                                120

(2 rows affected)

ACIDSample_Id        ACIDSample_Name                                                                                      ACIDSample_Price
-------------------- ---------------------------------------------------------------------------------------------------- ---------------------------------------
1                    Toy                                                                                                  132
2                    Shoes                                                                                                120
9                    Cat                                                                                                  200

(3 rows affected)

ACIDSample_Id        ACIDSample_Name                                                                                      ACIDSample_Price
-------------------- ---------------------------------------------------------------------------------------------------- ---------------------------------------
1                    Toy                                                                                                  132
2                    Shoes                                                                                                120

(2 rows affected)
```

### Serializable

上面的範例，雖然確保了查詢的一致性，但實際上會查詢到不存在的資料(Phantom)，
為了避免這樣的情況可以考慮使用 Serializable，但實際上會帶來效能的耗損，
在這樣嚴格的限制下，所有查詢將有序的執行。

Session 1

```sql
SET TRANSACTION ISOLATION LEVEL Serializable
BEGIN TRANSACTION
SELECT * FROM ACIDSample

WaitFor Delay '00:00:10'
SELECT * FROM ACIDSample
WaitFor Delay '00:00:10'
SELECT * FROM ACIDSample
COMMIT Transaction
```

也就是說在執行了 Sesson 1 的 SQL 之後 ，
Sesson 2 即使在 Delay 的時間中執行也必須等待 Sesson 1 結束才會執行，
這樣確保了資料一致性同時也不會拿到 Dirty Data 或 Phantom

Session 2 新增一筆資料

```sql
INSERT INTO ACIDSample (ACIDSample_Name,ACIDSample_Price)
VALUES ('Cat',200)
```

Session 3 刪除一列資料

```sql
DELETE ACIDSample WHERE ACIDSample_Name = 'Cat'
```

## SARGs (Search Arguments)

SQL 效能調校的一些 GuideLine

### 建議使符合「查詢參數(Search ARGument SARGs)」規則

- SARGs的格式：
  - 「資料欄位 部份運算子 <常數或變數>
  - 常數或變數> 部份運算子 資料欄位」
  - 運算子：=、<、>、>=、<=、BETWEEN、LIKE '關鍵字%'

- 非SARGs格式：
  - 對資料欄位做運算
  - 負向查詢：NOT、!=、<>、!>、!<、NOT EXISTS、NOT IN、NOT LIKE
  - 對資料欄位使用函數
  - 使用OR運算子

---

## 參考

- <https://medium.com/getamis/database-transaction-isolation-a1e448a7736e>
- <http://vito-note.blogspot.com/2013/05/blog-post_1930.html>
- <https://dotblogs.com.tw/ricochen/2011/07/31/32374>
- <https://www.sqlservercentral.com/articles/isolation-levels-in-sql-server>
- <https://medium.com/@chester.yw.chu/%E5%B0%8D%E6%96%BC-mysql-repeatable-read-isolation-%E5%B8%B8%E8%A6%8B%E7%9A%84%E4%B8%89%E5%80%8B%E8%AA%A4%E8%A7%A3-7a9afbac65af>
- <https://www.youtube.com/watch?v=7nv-7XQI7p0>
- <https://www.youtube.com/watch?v=ZtPj09tJjnQ>

(fin)
