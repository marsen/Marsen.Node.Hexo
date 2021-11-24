---
title: "[踩雷筆記] MacBook M1 silicon X Docker X MySQL Image"
date: 2021/11/05 11:31:07
---

## 前情提要

新工作配發了新的 MacBook 搭載最新的 M1 晶片,  
沒想到反而遇到了許多問題, 不會有太多的文字說明,  
僅用來記錄我實作有效的解決方法

### 情境:在 Docker 跑 MySQL

當執行以下語法時，

```shell
docker pull mysql 
```

會得到錯誤訊息如下

no matching manifest for linux/arm64/v8 in the manifest list entries

## 解決方案

### 拉取映像檔

> docker pull mysql/mysql-server:latest

### 建立 mysql container

> docker run -d --name mysql -p 3306:3306 mysql/mysql-server:latest

### 取得原始 root 密碼

> docker logs mysql 2>&1 | grep GENERATED  

**[Entrypoint] GENERATED ROOT PASSWORD: mysql_password_shows_here** 

### 連線至 container 的 image 執行 mysql 指令

> docker exec -it mysql bash

### 使用原始 root 密碼登入 mysql  

> mysql -u root -p
> Enter password:

輸入原始的 root 密碼後，
會看到畫面如下:

> Welcome to the MySQL monitor.  Commands end with ; or \g.  
Your MySQL connection id is 25  
Server version: 8.0.27  
>
> Copyright (c) 2000, 2021, Oracle and/or its affiliates.  
>
> Oracle is a registered trademark of Oracle Corporation and/or its  
affiliates. Other names may be trademarks of their respective owners.  
>
> Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.  

### 重設密碼

> ALTER USER USER() IDENTIFIED BY 'password';

### 授權

> CREATE USER 'root'@'%' IDENTIFIED BY 'root';
> GRANT ALL ON *.* TO 'root'@'%';
> flush privileges;

### 修改 root 密碼

> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
> flush privileges;

(fin)
