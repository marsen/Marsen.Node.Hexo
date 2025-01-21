---
title: " [實作筆記] 使用 Docker 一鍵生成 Sample 資料庫"
date: 2025/01/21 13:38:21
tags:
  - 實作筆記
---

## 前情提要

在開發或測試過程中，常常需要一個快速可用的資料庫環境來模擬真實操作。  
然而，手動安裝和配置 MySQL 不僅耗時，也容易出現版本兼容性問題。  
透過 Docker，我們可以輕鬆建立一個即時可用的環境，並在啟動時自動初始化資料庫的必要條件(表格或 root 帳密等…)。

本文將介紹如何使用 Dockerfile 和 SQL 初始化腳本，快速構建一個名為 sample 的測試資料庫。

## 實作記錄

### 步驟 1: 建立專案目錄與檔案

首先，創建一個新目錄，並在其中準備以下檔案：

Dockerfile

```dockerfile
# Dockerfile for setting up MySQL with a sample database

# 使用官方 MySQL 映像

FROM mysql:latest

# 設定環境變數 
ENV MYSQL_ROOT_PASSWORD={your root password}
ENV MYSQL_DATABASE=sample

# 複製初始化腳本

COPY init.sql /docker-entrypoint-initdb.d/

# 開放 MySQL 連接埠

EXPOSE 3306
```

init.sql

```sql
CREATE DATABASE IF NOT EXISTS sample;
USE sample;

```

### 步驟 2: 建構與啟動容器

執行以下指令來建構映像並啟動容器：

建構 Docker 映像

> docker build -t mysql-sample .

啟動容器

> docker run -d --name mysql-sample -p 3306:3306 mysql-sample

這將啟動一個名為 mysql-sample 的容器，並將 MySQL 的 3306 埠映射到本機。

### 步驟 3: 驗證資料庫

進入 MySQL 容器並確認資料庫與表格是否正確建立：

docker exec -it mysql-sample mysql -u root -p

在 MySQL shell 中執行：

```sql
SHOW DATABASES;
USE sample;
SHOW TABLES;
```

## 參考

開發用指令

```shell
❯ docker run -d \
--name mysql-container \
-e MYSQL_ROOT_PASSWORD=You don't need to try this password; I made it up. A7X3P \
-v mysql-data:/var/lib/mysql \
-p 3306:3306 \
mysql:latest
```

- [Docker 官方文件](https://www.docker.com/)
- [MySQL 官方文件](https://www.mysql.com/)

(fin)
