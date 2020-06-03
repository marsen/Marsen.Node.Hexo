---
title: "[實作筆記] Elasticsearch Insert Data with .Net  "
date: 2020/06/03 10:11:06
tag:
    - 實作筆記
---

[前篇](https://blog.marsen.me/2020/05/13/2020/dotnet_logger_with_elasticsearch_kibana/)

上次使用 Nlog 直接與 ElasticSearch 作結合，
這次來看看怎麼賽入資料給 ElasticSearch 。

## NEST

一般來說，ElasticSearch 只要透過呼叫 API 就可
但是我將使用 C# 的 Nuget 套件 [NEST](https://www.elastic.co/guide/en/elasticsearch/client/net-api/current/introduction.html) 來簡化呼叫 API 的行為。

## 步驟

### 安裝套件

```shell
Install-Package NEST
```

### 建立連線

```csharp
var node = new Uri("http://localhost:9200");
var settings = new ConnectionSettings(node);
var client = new ElasticClient(settings);
```

### 寫入資料

```csharp
var json = new
{
    name = "test name",
    timestamp = DateTime.Now.ToString("yyyy-MM-ddTHH:mm:ss.fffffffK")
};
string indexName = $"test-index-{DateTime.Now:yyyy.MM.dd}";
client.Index(json, idx => idx.Index(indexName));
```

### Create Index Pattern

連線進入 Kibana (<http://localhost:5601/>),  

Setting > Kibana > Index patterns > Create index pattern  

Step 1 of 2: Define index pattern  
輸入 `test-index-*`  

Step 2 of 2: Configure settings  
記得選取時間軸(x軸)為 `timestamp`，這裡會透過 automap 對應欄位，  
故在產生測試資料時，記得先產出時間資料，  
這個範例我使用的格式為

```csharp
DateTime.Now.ToString("yyyy-MM-ddTHH:mm:ss.fffffffK")
```

## 設計共用型別

```csharp
    public class Record
    {
        public DateTime Created { get; }
        public string Type { get; }
        public Object Record { get; }
    }
```

想法是通用類別，或許用抽像類別繼承也可以 ?
Created 欄位用來記錄資料產生時間，
Type 用來記錄 Record 的型別，
Record 用來記錄實際的資料。

預計未來可能會需要取出 Json 資料再轉成物件操作。

大概就醬。

(fin)
