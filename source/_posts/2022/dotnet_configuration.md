---
title: " [實作筆記] ASP.Net Core Config 的隨筆"
date: 2022/05/02 02:05:12
tags:
  - 實作筆記
---

## 提要

- DI Configuration
- 組態的層次
- Options Pattern

### DI Configuration

dotnet webapi 專案預設會注入 IConfiguration ，
在專案中的程式建構子放入 `IConfiguration` 就可以直接取得組態設定

```csharp=
private readonly IConfiguration _config;

public WeatherForecastController(IConfiguration config)
{
    _config = config;
}

[HttpGet(Name = "GetWeatherForecast")]
public string Get()
{
    return _config["ASPNETCORE_ENVIRONMENT"]!;
}
```

你可以將組態設定在 `appsettings.json` 之中
或是透過環境變數，使用環境變數有幾種作法

- 設定在機器上
- 設定在 Cloud Service 所提供的組態管理工具之中
- 使用 `launchSettings.json`

在開發環境我會使用 `launchSettings` 好處是可以不需要真的在開發機的環境中設定環境變數，
對同時擁有多個專案的開發者而言，可以有效隔離不同的專案變數，避免彼此影響污染

### 組態的層次

為了方便管理，我們可以透過組態的層次來加以分門別類。
比如說:

```json
  "Level": {
    "First": {
      "Key": "1st Value"
    },
    "Second": {
      "Key": "2nd Value"
    }
  },
```

一般的 JSON 或 XML 要作到這樣的層次結構都相當的簡單，
而要取用組態的方式如下:

```csharp=
var key = _config["Level:First:Key"];
```

不過我們要注意環境變數並無法使用`:`字符，也沒有 JSON 或是 XML 所能提供的結構，  
這裡要注意可以用 \_\_(雙底線(Double underscore))來建立層次。

```text
$Level__First__Key
$Level__Second__Key
```

### [Options Pattern](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/configuration/options?view=aspnetcore-6.0)

Options Pattern 可以提供一種更物件導向的組態配置解決方案，
你只需要兩個步驟，即可實作 Options Pattern

1. 建立 Option 物件
   我在這裡提供二種不同的方法，後面會再說明

   ```csharp=
   //方法一 Dictionary
   public class LevelOptions
   {
       public const string Level = "Level";

       public Dictionary<string,string> First { get; set; }
   ```

   ```csharp=
   //方法二 Object
   public class LevelOption
   {
       public const string Level = "Level";

       public FirstOptions First { get; set; }
   }

   public class FirstOptions
   {
       public string Key { get; set; } = string.Empty;
   }
   ```

2. 註冊 Option 物件

```csharp
  builder.Services.Configure<LevelOptions>(LevelOption.Level)
```

最後是讀取組態，在方法一(使用 Dictionary)的方法讀取

```csharp=
  var key = _options.Value.First["Key"];
```

或是方法二(使用 Object)的方法讀取

```csharp=
  var key = _options.Value.First.Key;
```

兩種方法都可以，物件化的方法需要建立多個物件，
但是在取用組態的過程就不會有 hard coding string;  
反之 Dictionary 的方法可以少寫很多物件。

對我而言我比較偏向使用 Object 的方法去處理，只要適當的分類，  
物件多並不是問題，而且通常一次不會增加太多的組態，可以迭代增量。

順代一提，如果組態檔層次太多，會是另一個困擾 ? 我的建議是，如果組態的深度到達 3 時，  
需要考慮這是不是一個壞味道，是不是過度優化了 ?  
實務上來說，我會避免建立超過 3 層，也很難會有一定要超過 3 層的情境，  
如果你有什麼情境上需要超過 3 層，可以留言給我，我們可以一起討論。

### 在 Program.cs 使用

如果在 `Program.cs` 的注入階段，如果要取得組態，  
可以用
`(new ConfigurationManager()).GetSection("First")["Key"]`  
或是 `Environment.GetEnvironmentVariable("Level__First__Key");` 等方法取得組態。  
但是我更推薦 ServiceProvider 的 GetRequiredService 取得 Options

```csharp=
//注入在前
builder.Services.Configure<LevelOption>(builder.Configuration.GetSection(LevelOption.Level));
builder.Services.AddHttpClient("OtherService", (provider, client) =>
{
    //透過 provider 取得 config
    var o = provider.GetRequiredService<IOptions<LevelOptions>>();
    client.BaseAddress = new Uri($"{o.Value.Url}/");
    client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", o.Value.Token);
});
```

這樣就可以用一致的方法取得組態了。

## 參考

- [ASP.NET Core 的設定](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/configuration/?view=aspnetcore-6.0)
- [ASP.NET Core 中的選項模式](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/configuration/options?view=aspnetcore-6.0)
- [.NET 中的選項模式](https://docs.microsoft.com/zh-tw/dotnet/core/extensions/options)
- [Looking inside ConfigurationManager in .NET 6](https://andrewlock.net/exploring-dotnet-6-part-1-looking-inside-configurationmanager-in-dotnet-6/)
- [你可能不知道的.Net Core Configuration(Evernote 備份)](https://www.evernote.com/shard/s36/sh/20abd4ce-f7e2-47ef-9bc6-36971709348a/d0d0525f9b4b7e5dcf1493eca0b8c232)

(fin)
