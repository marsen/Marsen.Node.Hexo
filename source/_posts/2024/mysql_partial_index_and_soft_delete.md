---
title: "[學習筆記] SQL 軟刪除與索引 (MySQL/MSSQL/PostgreSQL)"
date: 2024/08/05 13:53:10
---

## 前情提要

最近需要實現軟刪除功能，但在設計索引時遇到了一些問題。  
商務情境如下，  
有效的 ACCOUNT 必須是唯一的。  
然而，帳戶有可能會被軟刪除，所以被刪除的 Account 可能會有多筆相同的資料。  

### 表格設計

```sql
CREATE TABLE User (
    Id SERIAL PRIMARY KEY,
    Username VARCHAR(255) NOT NULL,
    IsDeleted BOOLEAN DEFAULT TRUE
);
```

簡化設計的用戶表格包含 Id、Username 和 IsDeleted 欄位。  
依商業需求，當 IsDeleted 為 0 時，Username 必須唯一。  
而且很有可能會有多筆相同的帳號被刪除，當 IsDeleted 為 1 時，Account 不會限制只有一筆（允許多筆）

後端工程師建議使用觸發器（Trigger）或在應用層（Backend）實現這個約束，  
根據我的記憶，在微軟的 SQL Server 中，有一種稱為「條件約束」的設定，能夠針對特定條件創建索引。
可以大幅節省開發成本，我認為這類的功能不應該只有微軟專有，故作了一些搜尋後，特別以此篇記錄。

## 實作

我找到一個測試的[網站](https://sqlfiddle.com)，你可以直接在這裡測試，不需要花費額外心力建立 SQL Server

### MySQL

```sql
-- 創建表格
CREATE TABLE Users (
    Id INT AUTO_INCREMENT PRIMARY KEY,
    Username VARCHAR(255) NOT NULL,
    IsDeleted BOOLEAN DEFAULT FALSE
);

-- 設置 AUTO_INCREMENT 起始值為 1000
ALTER TABLE Users AUTO_INCREMENT = 1000;

-- 創建唯一索引，只針對 IsDeleted = FALSE 的行
CREATE UNIQUE INDEX unique_active_account ON Users ((CASE WHEN IsDeleted THEN Username END));
-- 插入數據
INSERT INTO Users (Username, IsDeleted) VALUES ('user1', FALSE);  -- 成功
INSERT INTO Users (Username, IsDeleted) VALUES ('user2', FALSE);  -- 成功
INSERT INTO Users (Username, IsDeleted) VALUES ('user3', TRUE);   -- 成功，因為 IsDeleted = TRUE 不受唯一索引限制
INSERT INTO Users (Username, IsDeleted) VALUES ('user4', FALSE);  -- 成功
INSERT INTO Users (Username, IsDeleted) VALUES ('user5', TRUE);   -- 成功，因為 IsDeleted = TRUE 不受唯一索引限制

-- 測試重複的 Username 插入，應該失敗
INSERT INTO Users (Username, IsDeleted) VALUES ('user1', FALSE);  -- 失敗，因為 user1 已經存在且 IsDeleted = FALSE
-- 測試重複的 Username 插入，但 IsDeleted = TRUE，應該成功
INSERT INTO Users (Username, IsDeleted) VALUES ('user1', TRUE);   -- 成功，因為 IsDeleted = TRUE 不受唯一索引限制

-- 檢查插入結果
SELECT * FROM Users;

```

### PostgreSQL

```sql
-- 創建表
CREATE TABLE Users (
    Id SERIAL PRIMARY KEY,
    Username VARCHAR(255) NOT NULL,
    IsDeleted BOOLEAN DEFAULT FALSE
);

ALTER SEQUENCE users_id_seq RESTART WITH 1000;
-- 創建部分索引，只針對 IsDeleted = FALSE 的行
CREATE UNIQUE INDEX unique_active_username ON Users (Username)
WHERE IsDeleted = FALSE;


-- 插入範例數據
INSERT INTO Users (Username, IsDeleted) VALUES ('user1', FALSE);  -- 成功
INSERT INTO Users (Username, IsDeleted) VALUES ('user2', FALSE);  -- 成功
INSERT INTO Users (Username, IsDeleted) VALUES ('user3', TRUE);   -- 成功，因為 IsDeleted = TRUE 不受唯一索引限制
INSERT INTO Users (Username, IsDeleted) VALUES ('user4', FALSE);  -- 成功
INSERT INTO Users (Username, IsDeleted) VALUES ('user5', TRUE);   -- 成功，因為 IsDeleted = TRUE 不受唯一索引限制

-- 測試重複的 Username 插入，應該失敗
-- INSERT INTO Users (Username, IsDeleted) VALUES ('user1', FALSE);  -- 失敗，因為 user1 已經存在且 IsDeleted = FALSE
-- 測試重複的 Username 插入，但 IsDeleted = TRUE，應該成功
INSERT INTO Users (Username, IsDeleted) VALUES ('user1', TRUE);   -- 成功，因為 IsDeleted = TRUE 不受唯一索引限制

-- 檢查插入結果
SELECT * FROM Users;

```

### MSSQL

```sql
-- 設置正確的 SET 選項
SET QUOTED_IDENTIFIER ON;
SET ANSI_NULLS ON;

-- 創建表
CREATE TABLE [User] (
    Id INT IDENTITY(1000, 1) PRIMARY KEY,
    Username VARCHAR(255) NOT NULL,
    IsDeleted BIT DEFAULT 0
);

-- 創建篩選唯一索引
CREATE UNIQUE INDEX unique_active_account ON [User](Username)
WHERE IsDeleted = 0;

-- 插入範例數據
INSERT INTO [User] (Username, IsDeleted) VALUES ('user1', 0);  -- 成功
INSERT INTO [User] (Username, IsDeleted) VALUES ('user2', 0);  -- 成功
INSERT INTO [User] (Username, IsDeleted) VALUES ('user3', 1);  -- 成功，因為 IsDeleted = 1 不受唯一索引限制
INSERT INTO [User] (Username, IsDeleted) VALUES ('user4', 0);  -- 成功
INSERT INTO [User] (Username, IsDeleted) VALUES ('user5', 1);  -- 成功，因為 IsDeleted = 1 不受唯一索引限制

-- 測試重複的 Username 插入，應該失敗
INSERT INTO [User] (Username, IsDeleted) VALUES ('user1', 0);  -- 失敗，因為 user1 已經存在且 IsDeleted = 0

-- 測試重複的 Username 插入，但 IsDeleted = 1，應該成功
INSERT INTO [User] (Username, IsDeleted) VALUES ('user1', 1);  -- 成功，因為 IsDeleted = 1 不受唯一索引限制

-- 檢查插入結果

SELECT * FROM [User];
```

## 小結

不太需要複雜的後端程式或是 DB Trigger，只需要在建立索引時加上條件，  
特別注意 MySQL 的語法是使用 CASE WHEN，其他 DB 是使用 WHERE，  
這與 DB 版本也有關係，使用前應該進一步去查詢官方文件。

另外關於遞增欄位，在不同的 DB 也有不同的實作方式。  
實務上通常不用了解這麼多 DB 的差異，僅僅是我個人的好奇補充罷了,  
業界主推還是 PostgreSQL，我個人不夠專業，但三種都略有碰過，最熟的還是 MSSQL。  
僅為個人學習記錄，如果要在三者中擇一還是需要多方考慮自身的 Context 再作決定。

## 參考

- [MSSQL 唯一條件約束及檢查條件約束](https://learn.microsoft.com/zh-tw/sql/relational-databases/tables/unique-constraints-and-check-constraints?view=sql-server-ver16)
- [SQL Fiddle](https://sqlfiddle.com)

(fin)
