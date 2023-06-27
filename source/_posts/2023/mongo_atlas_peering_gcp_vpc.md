---
title: "[實作筆記] MongoDB 解決方案評估(未完成)"
date: 2023/06/27 18:19:33
---

## 前情提要

1. 方案的比較

   - Mongo Atlas(Start from GCP Marketing)
   - GCP VM with MongoDB
   - Azure VM with MongoDB
   - Mongo Atlas(Start from other Cloud Service:AWS、Azure)

2. 決策的考量

   - 成本與帳務
   - 維運的代價

## 實作隨筆：Mongo Atlas(Start from GCP Marketing)

### 測試規劃

![mongo atlas 連線的測試架構](../../images/2023/mongo_atlas_test_infra.png)

如圖，為了不影響原有的環境，我打算建一組新的 VPC Network 進行實驗，  
並在這個網路中建立一台實體的 VM 機器，待 Mongo Atlas 設定完成後，  
進行連線的測試。

測試的方法，由於 MongoDB Atlas 採用一種安全性較高的連線政策，  
必需使用以下連線集群的三種方式才可以連到資料庫:  
第一種，IP Access List，使用 GUI 建立 Cluster 的當下會自動加入一組你所在網路對外的 IP，  
如果不是固定 IP 可能會有問題  
第二種，Peering ，要付錢的版本 M10 以上才可用，也是我們這次實作的重點目標  
最後一種，Create a Private Endpoint，也是　 M10 以上才可用，但是不是我們這次的主要實作項目，所以不過多的展開。

### 從 GCP Marketing 建立 Mongo Atlas

在 GCP 的 Marketing 搜尋並訂閱 Mongo Atlas 後點擊 `MANAGE ON PROVIDER`  
在 Mongo Atlas 建立 Cluster，也可以建立 Project 與 User 作更細緻的管控。  
這時候可以到 Network Access 查看 IP Access List 的清單，應該會有你網路上設定的對外 IP,  
這個流程是自動化的，但是我個人認為不是固定 IP 的話可能會有問題，如果有人可以給我一些提點會十分感激。

![IP Access List](../../images/2023/mongo_atlas_ip_access_list.png)

### 建立 VM

[GCP 建立 VM](https://cloud.google.com/compute/docs/instances/create-start-instance) 是十分簡單的，就不多作說明。  
同時記得安裝我們的[測試工具 - mongosh](https://www.mongodb.com/docs/mongodb-shell/install/)

### Mongo DB 相關

#### 切換 db

```sql
use mydb
```

#### Create User

```sql
db.createUser({ user: "username", pwd: "password", roles: [{ role: "roleName", db: "databaseName" }] });
```

#### 查詢 User

```sql
db.getUsers()
```

#### User 加入角色

```shell
db.grantRolesToUser("username", [{ role: "readWriteAnyDatabase", db: "mydb" }])
```

#### User 移除角色

```shell
db.revokeRolesFromUser("usernmae", [{ role: "readWrite", db: "admin" }])
```

### 前置作業:GCP VPC 　與 Firewall Rules 設定

為了避免影響原有的系統，  
建立 GCP 一組新的 VPC Network : `vpc-lab`，一般來說 GCP 的專案會有自動建立一組 `default` VPC Network，
而 `default` VPC Network 會預設建立以下的防火牆規則

- default-allow-icmp – 允許來自任何來源對所有網路 IP 進行存取。ICMP 協議主要用於對目標進行 ping 測試。
- default-allow-internal – 允許在任何埠口上的實例之間建立連接。
- default-allow-rdp – 允許從任何來源連接到 Windows 伺服器的 RDP 會話。
- default-allow-ssh – 允許從任何來源連接到 UNIX 伺服器的 SSH 會話。

與此對應，我也建立相同的規則給`vpc-lab`，如下:

- vpc-lab-allow-icmp
- vpc-lab-allow-internal
- vpc-lab-allow-rdp
- vpc-lab-allow-ssh

可以用以下的語法測試一下網路是否能連，如果可以連線再進行 mongodb connection 的測試

```shell
ping {ip address}
```

```shell
telnet {ip address} {port}
```

```shell
nc -zv {ip address}
```

### 測試連線用的語法

如果網路測試沒有問題，再進行 mongodb 的連線，由於目前沒有設定 Peering 連線，  
所以在開發機上可以(網路環境需要在 IP Access List 內)，而使用 GCP VM 會無法連線，  
開發機連線 GCP MongoDB 語法

```shell
mongosh mongodb://{user:pwd}@{mongodb_ip}:27017/my_db
```

開發機連線 MongoDB Atlas 語法

```shell
mongosh mongodb+srv://{user:pwd}@{atlas_cluster_name}.mongodb.net/my_db
```

### Peering 實作

1. GCP 的防火牆設定

   1. SSH 連線
   2. Tags
      1. mongo-atlas-0614
      2. mongo-atlas-req-0614

## 參考

- [Set Up a Network Peering Connection](https://www.mongodb.com/docs/atlas/security-vpc-peering/)
- [在 GCP 中使用 MongoDB Atlas 服務](https://blog.cloud-ace.tw/database/gcp-mongodb-atlas/)
- [How to Configure Firewall Rules in Google Cloud Platform(GCP)](https://geekflare.com/gcp-firewall-configuration/)

(fin)
