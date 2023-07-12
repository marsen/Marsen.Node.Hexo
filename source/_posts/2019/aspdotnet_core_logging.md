---
title: " [實作筆記] ASP.Net Core Logger"
date: 2019/04/06 12:21:17
tags:
  - .Net Framework
  - 實作筆記
---

## 要知道的事

- 這是個人的學習記錄
- 可能對你沒幫助
- 網路上資訊很多
- 希望對你有幫助

## 概念

Asp.Net Core 的 Life Cycle 由 `Program.cs` 的 `main` 方法開始(是的，就如同其它一般的程式)，  
在 `WebHostBuilder` 中的 `ConfigureLogging` 可以提供彈性讓你設定屬於你的 LoggerProvider，
不論是微軟提供、知名的第三方套件或是你手工自已刻一個，大致你的程式碼會如下

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .ConfigureLogging(logging=>
        {
            logging.ClearProviders();
            logging.AddEventLog();
            logging.AddFile("D:\\Temp\\Log.txt");
            logging.AddConsole();
        })
        .UseStartup<Startup>()
```

而在 Controller 或其它 Module 間，你只要透過建構子注入 logger 實體就可以實現 log 的功能

```csharp
    public HomeController(ILogger<HomeController> logger)
    {
        this._logger = logger;
    }
```

## 預設的行為

如果你沒有呼叫 `ConfigureLogging` 預設的行為如下述.

The default project template calls `CreateDefaultBuilder`， which adds the following logging providers:

- Console
- Debug
- EventSource (starting in ASP.NET Core 2.2)

```csharp
public static void Main(string[] args)
{
    CreateWebHostBuilder(args).Build().Run();
}

public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>();
```

### Find the Logs

#### Console

範例說明:在建構子中注入 `ILogger` 實體，運行網站後連到 Home\Index 頁面，並觀察 Console

```csharp
public class HomeController : Controller
{
    private readonly ILogger _logger;

    /// <summary>
    /// Initializes a new instance of the <see cref="HomeController" /> class.
    /// </summary>
    public HomeController(ILogger<HomeController> logger)
    {
        this._logger = logger;
    }

    public IActionResult Index()
    {
        this._logger.Log(LogLevel.Information，"HomeController Information");
        this._logger.Log(LogLevel.Critical，"HomeController Critical");
        this._logger.Log(LogLevel.Debug，"HomeController Debug");
        this._logger.Log(LogLevel.Error，"HomeController Error");
        this._logger.Log(LogLevel.None，"HomeController None");
        this._logger.Log(LogLevel.Trace，"HomeController Trace");
        this._logger.Log(LogLevel.Warning，"HomeController Warning");
        return View();
    }
}
```

結果如下，可以發現 `LogLevel.None`、`LogLevel.Trace` 與 `LogLevel.Warning` 並未出現在 Console 資訊當中

```text
info: Marsen.NetCore.Site.Controllers.HomeController[0]
      HomeController Information
crit: Marsen.NetCore.Site.Controllers.HomeController[0]
      HomeController Critical
dbug: Marsen.NetCore.Site.Controllers.HomeController[0]
      HomeController Debug
fail: Marsen.NetCore.Site.Controllers.HomeController[0]
      HomeController Error
```

[LogLevel](https://docs.microsoft.com/zh-tw/dotnet/api/microsoft.extensions.logging.loglevel?view=aspnetcore-2.2)說明了 `None` 的意義就是不記錄任何訊息，

| Enum        | Level | Description                                                                                                                                                                                          |
| ----------- | ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Trace       | 0     | Logs that contain the most detailed messages. These messages may contain sensitive application data. These messages are disabled by default and should never be enabled in a production environment. |
| Debug       | 1     | Logs that are used for interactive investigation during development. These logs should primarily contain information useful for debugging and have no long-term value.                               |
| Information | 2     | Logs that track the general flow of the application. These logs should have long-term value.                                                                                                         |
| Warning     | 3     | Logs that highlight an abnormal or unexpected event in the application flow， but do not otherwise cause the application execution to stop.                                                          |
| Error       | 4     | Logs that highlight when the current flow of execution is stopped due to a failure. These should indicate a failure in the current activity， not an application-wide failure.                       |
| Critical    | 5     | Logs that describe an unrecoverable application or system crash， or a catastrophic failure that requires immediate attention.                                                                       |
| None        | 6     | Not used for writing log messages. Specifies that a logging category should not write any messages.                                                                                                  |

Log 的作用範圍會受 `appsettings.json` 影響，  
另外要注意 appsettings.json 的載入順序.

```json
  "Logging": {
    "LogLevel": {
      "Default": "Trace"，
      "System": "Information"，
      "Microsoft": "Information"
    }
```

#### Debug

如同 `Console` 的行為一般，可以在 Visual Studio 的輸出(Output)>偵錯(Debug)視窗中，查詢到記錄。

![Console](https://i.imgur.com/274witm.jpg)

#### EventSource

如同[官方文件](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?view=aspnetcore-2.2#eventsource-provider)所說，我下載了 `PerfView` ，
如下圖作了設定，  
![PerfView](/images/2019/4/perfview.jpg)  
不過我並沒有取得記錄，  
![PerfView Log](/images/2019/4/perfview.jpg)

錯誤訊息如下  
 `EventSource Microsoft-Extensions-Logging: Object reference not set to an instance of an object`  
暫時不打算深追查，
ETW 可以記錄的 Memory 、Disc IO 、CPU 等資訊，  
其實與我想要的應用程式記錄有所差異，稍稍記錄一下以後也許用得到。  
如果有人能留言給我一些方向，也是非常歡迎。

## 自訂 Filelog 與 EventLog

調整一下程式

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .ConfigureLogging(logging=>
        {
            logging.AddEventLog();
            logging.AddFile("D:\\Temp\\Log.txt");
        })
        .UseStartup<Startup>()
```

這裡我使用 `Microsoft.Extensions.Logging.EventLog` 處理 EventLog 可以在 Event View 中看見記錄;  
而 file log 我使用 `Serilog.Extensions.Logging.File` ， 特別要注意以下兩點

- Nuget 使用的版本為 2.0.0 以上版本，目前仍然不是穩定版本
- AddFile 傳入的是記錄檔的完整 Path 而非目錄

## 自訂 Elmah

Elmah 在 Net 算是一個蠻方便的工具，有提供簡易介面、可以選擇用 File 或是 Database 方式作 Logging，  
更重要是小弟我用了 4 年，順手就研究一下。

設定相當簡單， 在 `Startup.cs` 的 `ConfigureServices` 加入

```csharp
services.AddElmah<XmlFileErrorLog>(options =>
{
    //options.CheckPermissionAction = context => context.User.Identity.IsAuthenticated;
    //options.Path = @"elmah";
    options.LogPath = "D:\\Temp\\elmah";
})
```

在 `Configure` 加入

```csharp
app.UseElmah();
```

要注意是使用 `XmlFileErrorLog` 時，要設定的 options 是 `LogPath` 而非 `Path`，  
其實使用 File 只能說是開發環境的暫時處置，真正的 Prodction 應該將 Log 放到專門的 Database 或是 Cloud Service 之中，  
在這裡可以看見 Elmah 的行為與 Net Core 的行為並不一致，Log 與錯誤記錄本來就不該混為一談。  
我想我要調整一下我的想法了，不過關於 Log 暫時就到此為止。

## 參考

- [ASP.NET Core Logging](https://codingblast.com/asp-net-core-logging/)
- [.NET Core Logging With LoggerFactory: Best Practices and Tips](https://stackify.com/net-core-loggerfactory-use-correctly/)
- [Logging in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?view=aspnetcore-2.2#log-scopes)
- [[鐵人賽 Day16] ASP.NET Core 2 系列 - 多重環境組態管理 (Multiple Environments)](https://blog.johnwu.cc/article/ironman-day16-asp-net-core-multiple-environments.html)
- [ASP.NET Core EventLog provider](https://stackoverflow.com/questions/47773058/asp-net-core-eventlog-provider)
- [ASP.NET Core: The MVC Request Life Cycle](http://www.techbloginterview.com/asp-net-core-the-mvc-request-life-cycle/)
- [Monitoring and Observability in the .NET Runtime](https://mattwarren.org/2018/08/21/Monitoring-and-Observability-in-the-.NET-Runtime/)
- [ElmahCore/ElmahCore](https://github.com/ElmahCore/ElmahCore)

(fin)
