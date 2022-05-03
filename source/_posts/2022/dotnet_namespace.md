---
title: "[實作筆記] dotnet core 實作隨筆"
date: 2022/05/03 15:13:12
---

## web 專案統一錯誤處理

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

(fin)
