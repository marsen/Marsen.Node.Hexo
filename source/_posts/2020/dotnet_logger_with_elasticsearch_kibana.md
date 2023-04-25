---
title: " [實作筆記] Dotnet Logger 整合 Kibana 與 Elasticsearch"
date: 2020/05/13 15:47:03
tag:
  - CI/CD
  - 實作筆記
---

## 前情提要

我將 ASP.Net Core 的 API 服務加上 Log。
Logger 使用 NLog ，載體我使用 Elasticsearch ，  
使用者操作介面使用 Kibana。

然後面對開發者，我希望用 AOP 方式，
讓 Logger 與主邏輯分離。

- 這裡會用到簡單基本的 docker 技術
- 透過 docker-compose 建立 Kibana 與 Elasticsearch
- 假設你已經會使用 Dotnet Core DI 注入 Logger
- 用最原生的方法實作 AOP
- 對你可能沒有幫助

## Run ElasticSearch & Kibana with docker

在本機建立 ElasticSearch & Kibana  
首先建立 docker-compose.yaml 檔如下，  
簡單說明一下內容:

- 起一個 elasticsearch，port : 9200
- 起一個 kibana ，port : 5601，設定環境變數 `ELASTICSEARCH_URL` 為 `http://localhost:9200`
- 網路名命為 elastic 使用 bridge 讓兩個 container 連起來
- `elasticsearch-data:` 實務上我想需要指定一個 storage(硬碟或 File System 之類的)

```yaml
version: "3.1"

services:
  elasticsearch:
    container_name: elasticsearch
    image: docker.elastic.co/elasticsearch/elasticsearch:7.6.2
    ports:
      - 9200:9200
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
    environment:
      - xpack.monitoring.enabled=true
      - xpack.watcher.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.type=single-node
    networks:
      - elastic

  kibana:
    container_name: kibana
    image: docker.elastic.co/kibana/kibana:7.6.2
    ports:
      - 5601:5601
    depends_on:
      - elasticsearch
    environment:
      - ELASTICSEARCH_URL=http://localhost:9200
    networks:
      - elastic

networks:
  elastic:
    driver: bridge

volumes:
  elasticsearch-data:
```

執行

```shell
docker-compose up -d
```

完成後在本機瀏覽器瀏覽以下網址。  
確定功能正常。

- Elasticsearch URL <http://localhost:9200/>
- Kibana URL <http://localhost:5601/>

### 雷包

第一次啟動 Kibana 要 5 分鐘左右，但是我不確定是 docker 或是 Kibana 的問題

## Dotnet Core NLog with ElasticSearch

首先必需安裝相關套件

```shell
Install-Package NLog.Web.AspNetCore

Install-Package NLog

Install-package NLog.Targets.ElasticSearch
```

~~設定 `appsettings.json`~~
(請見 [20200518 的補充](#20200518-補充))

```json
{
  "ConnectionsString": {
    "ElasticSearchServerAddress": "http://localhost:9200/"
  }
  ///...
}
```

設定 nlog.config，注意以下路徑

- `nlog > extenstions > assembly`
- `nlog > targets > target`
- `nlog > rules > logger`

~~這裡的重點是 `ElasticSearchServerAddress` 這組字串需設定成你的 ElasticSearchServerAddress~~  
(請見 [20200518 的補充](#20200518-補充))

```xml
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      autoReload="true"
      internalLogLevel="Info"
      internalLogFile="c:\temp\internal-nlog.txt">

  <!-- enable asp.net core layout renderers -->
  <extensions>
    <add assembly="NLog.Web.AspNetCore"/>
    <add assembly="NLog.Targets.ElasticSearch"/>
  </extensions>

  <!-- the targets to write to -->
  <targets>
    <!-- write logs to file  -->
    <target xsi:type="File" name="allfile" fileName="${currentdir}\log\nlog-${shortdate}.log"
            layout="${longdate}|${event-properties:item=EventId_Id}|${uppercase:${level}}|${logger}|${message} ${exception:format=tostring}" />
    <!-- ElasticSearch -->
    <target name="ElasticSearch"
            xsi:type="ElasticSearch"
            ConnectionStringName="ElasticSearchServerAddress"
            index="dotnetcore-nlog-elk-${date:format=yyyy.MM.dd}"
            documentType="logevent"
            includeAllProperties="true"
            layout="[${date:format=yyyy-MM-dd HH\:mm\:ss}][${level}] ${logger} ${message} ${exception:format=toString}">
      <field name="MachineName" layout="${machinename}" />
      <field name="Time" layout="${longdate}" />
      <field name="level" layout="${level:uppercase=true}" />
      <field name="logger" layout=" ${logger}" />
      <field name="message" layout=" ${message}" />
      <field name="exception" layout=" ${exception:format=toString}" />
      <field name="processid" layout=" ${processid}" />
      <field name="threadname" layout=" ${threadname}" />
      <field name="stacktrace" layout=" ${stacktrace}" />
      <field name="Properties" layout="${machinename} ${longdate} ${level:uppercase=true} ${logger} ${message} ${exception}|${processid}|${stacktrace}|${threadname}" />
    </target>
  </targets>

  <!-- rules to map from logger name to target -->
  <rules>
    <!--All logs, including from Microsoft-->
    <logger name="*" minlevel="Trace" writeTo="allfile" />
    <logger name="*" minlevel="Trace" writeTo="ElasticSearch" />
  </rules>
</nlog>
```

我的 Logger 代碼可能會類似這樣:
我想調整 Logger 代碼，不要與商務邏輯混在一起。

- 一般的 Logger 我會用 AOP 的方式作成 Audit Log 記錄
- catch Exception 的 Logger 我會統一處理

```csharp
private readonly ILogger logger;

public Result MyMethod(Context ctx)
{
    this.logger.LogInformation("Hello Marsen");
    try{
      //// do some thing
    }
    catch
    {
        this.logger.LogError("What's a Wonderful World");
    }
}
```

### 20200518 補充

`nlog.config` 與 `appsettings.json` 內容調整。

使用[Elasticsearch Cloud](https://www.elastic.co/)(以下簡稱 ESC)當作服務的儲存體。  
踩到了一個雷包，當我把 `ElasticSearchServerAddress` 修改成 ESC 服務的 EndPoint  
我發現 ESC 並未接收到 Log ，更奇怪的事情是，在我本機端 docker 所建置服務仍然收到了 Log。

查詢了一下最新的[Nlog ElasticSearch Wiki](https://github.com/markmcdowell/NLog.Targets.ElasticSearch/wiki)後，  
應該改用 `uri` 屬性設定 EndPoint ，而沒有設定的情況下預設為 `localhost:9200` ，

```text
uri - Uri of a Elasticsearch node. Multiple can be passed comma separated.
Ignored if cloud id is provided. Layout Default: http://localhost:9200
```

而要使用 ESC 的 EndPoint，除了設定 `uri` 外，還需要啟用授權。
`requireAuth` 設定為 `true`(預設為`false`)，另外還要設定 `username` 與 `password`。
`appsettings.json` 的 ConnectionsString 就可以刪除了。  
`nlog.config` 修改大致如下

```xml
<!--略-->
<target xsi:type="ElasticSearch"
  name="elastic.co"
  index="dotnetcore-nlog-elk-${date:format=yyyy.MM.dd}"
  documentType="_doc"
  includeAllProperties="true"
  layout="[${date:format=yyyy-MM-dd HH\:mm\:ss}][${level}] ${logger} ${message} ${exception:format=toString}"
  uri="https://**************elastic-cloud.com:9243"
  requireAuth="true"
  username="**********"
  password="******************">
<!--略-->
```

## Dotnet Core AOP

用 AOP 的方式作成 Audit Log 記錄，  
想法是方法的 in / out 我將想知道資訊記錄下來。
比如輸入的參數或是回傳值。
題外話，因為 AOP 的特性，如果方法處理到一半想要記錄是作不到的，
這是不是意味著必須重構將方法一分為二 ?

[參考這篇](https://www.cnblogs.com/jlion/p/12394949.html)，我會使用最基本的 Filter 實作 AOP，  
![Filter 簡介](https://i.imgur.com/iZ391Rb.png)
依據 `Filter` 的特性我在這裡會實作 `IActionFilter`

```csharp
public class AuditLogAttribute : Attribute, IActionFilter
{
    private readonly ILogger logger;

    public AuditLogAttribute(ILogger<AuditLogAttribute> logger)
    {
        this.logger = logger;
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        this.logger.LogInformation("Result Filter End");
    }

    public void OnActionExecuting(ActionExecutingContext context)
    {
        this.logger.LogInformation("Result Filter Start");
    }
}
```

我的最終目標是透過掛載 Attribute 的方式來讓 Logger 與解耦，  
參考下方的程式碼。

```csharp
private readonly ILogger logger;

[AuditLog]
public Result MyMethod(Context ctx)
{
    try{
      //// do some thing
    }
    catch
    {
        this.logger.LogError("What's a Wonderful World");
    }
}
```

這裡就有一件討厭的事，因為我的 Logger 都是透過建構子注入產生，  
而自已本身也比較傾向不要使用公開 Property Injection 的方式\*。
但是使用建構子在這裡會產生另一個問題，
我將無法使用掛載 Attribute 的方式處理 Logger  
解決的方式是透過另一個 Attribute `ServiceFilterAttribute` 作間接掛載

```csharp
private readonly ILogger logger;

[ServiceFilter(typeof(AuditLogAttribute))]
public Result MyMethod(Context ctx)
{
    try{
      //// do some thing
    }
    catch
    {
        this.logger.LogError("What's a Wonderful World");
    }
}
```

這樣作還是有一些缺點，最明顯就是掛載的 Attribute 變長，  
然後視覺上又是末端文字才能表達意涵，  
另一點是原本約定成俗可以省略的`*Attribute`後綴不能省略了。

我想使用 [Castle](http://www.castleproject.org/) 與 [AutoFac](https://autofac.org/) 重作一次。
應該可以變得更加簡潔。

## 參考

- <https://www.humankode.com/asp-net-core/logging-with-elasticsearch-kibana-asp-net-core-and-docker>
- <https://kevintsengtw.blogspot.com/2018/07/aspnet-core-nlog-log-elasticsearch.html>
- <https://kevintsengtw.blogspot.com/2018/07/aspnet-core-nlog-log-elasticsearch_9.html>
- <https://www.cnblogs.com/jlion/p/12394949.html>
- <https://paper.dropbox.com/doc/202001-DIAOP--Az2Kl_lUwdJulCygth_VSyrcAg-41nJ40hCmLj7o7haWhual>
- <https://blog.johnwu.cc/article/ironman-day14-asp-net-core-filters.html>
- <https://edi.wang/post/2019/1/21/aspnet-core-dependency-injection-in-actionfilterattribute>
- 請比較 Property Injection 與其它的注入方式的優缺點。

(fin)
