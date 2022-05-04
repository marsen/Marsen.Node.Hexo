---
title: "[實作筆記] dotnet core 實作隨筆"
date: 2022/05/03 15:13:12
---

## Web/Api 專案統一錯誤處理

首先建立一個新的類別 `ErrorHandlingFilter` 繼承 `ExceptionFilterAttribute` 並複寫 `OnException` 函數

```csharp
public class ErrorHandlingFilter : ExceptionFilterAttribute
{
    public override void OnException(ExceptionContext context)
    {
        //do your logical here
    }
}
```

在 Program.cs 中注入

```csharp
builder.Services.AddControllers(options => { options.Filters.Add(new ErrorHandlingFilter()); });
```

## 專案更改 Root Namespace

預設會使用專案名稱作為整個專案的 Root Namespace，
但是我們可以在 csproj 檔中設定 `RootNamespace` 讓它與專案名稱有所差異

```csharp
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
    <RootNamespace>MyNameSpace</RootNamespace>
  </PropertyGroup>
</Project>
```

## Nuget

### add nuget registry

```bash
dotnet nuget add source <registry url> -n SourceName -u UserName -p Password --store-password-in-clear-text
```

### pack

```bash
dotnet pack -o <output folder> -c NUGET --no-dependencies --force  -p:PackageVersion=<version>
```

### push

```bash
dotnet nuget push -s SourceName <Packed Version>
```

## 建立共用的 HttpClient

在 Program.cs 中註冊

```csharp
builder.Services.AddHttpClient("ThirdService", (provider, client) =>
{
    var o = provider.GetRequiredService<IOptions<ThirdServiceOptions>>();
    client.BaseAddress = new Uri($"{o.Value.Url}/");
    client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", o.Value.Token);
});
```

再透過 HttpClientFactory 具名取用

```csharp
public MyService(IHttpClientFactory httpClientFactory)
{
  HttpClient _httpClient = httpClientFactory.CreateClient("ThirdService");
}
```

## 最小化 docker image

這一種被稱為 Multiple Stage Builds 的 Docker 技巧，
主要的概念是將建置(Build)與部署(Deploy)的步驟分開，
最終只部署出營運所需的映象檔(Image)。

比如說下面的 `Docker File` 就是將 dotnet 專案作了兩步驟的處理

此外，如果我們更換適當的 Image，  
比如說，在 Create Runtime Image Step，
將 `mcr.microsoft.com/dotnet/aspnet:6.0` 改為 `mcr.microsoft.com/dotnet/aspnet:6.0-alpine`
映象檔的 Size 將減少 46.6% (216.11 MB → 115.4 MB)

```dockerfile
# Build Step
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /source
# copy everything else and build app
COPY /src/WebAPI/. ./WebApi/
WORKDIR /source/WebApi
RUN dotnet restore 'WebApi.csproj'
RUN dotnet build 'WebApi.csproj' -c release -o /app --no-restore
RUN dotnet publish 'WebApi.csproj' -c release -o /app --no-restore

# Create Runtime Image Step
FROM mcr.microsoft.com/dotnet/aspnet:6.0-alpine
ENV ASPNETCORE_URLS=http://+:5110
WORKDIR /app
COPY --from=build /app ./
EXPOSE 5110
ENTRYPOINT ["dotnet", "WebApi.dll"]
```

## 參考

- [在 ASP.NET Core 中使用 IHttpClientFactory 發出 HTTP 要求](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/http-requests?view=aspnetcore-6.0)
- [透過 Multiple Stage Builds 編譯出最小的 Docker Image](https://jiepeng.me/2018/06/09/use-docker-multiple-stage-builds)

(fin)
