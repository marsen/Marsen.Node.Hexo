---
title: "[實作筆記] 用 Github Registry 作自已的 Nuget Server"
date: 2019/10/11 11:55:41
tag:
 - 實作筆記 
---

## 目標

使用 GitHub Package Registry 建立一個自已的 Nuget Server。

## 步驟

### 前置作業

#### 申請

GitHub Package Registry 還在 Beta 階段，需要申請才能取得試用。
過程並不複雜，在此不贅述，請再自行查找網路。

#### 建立 Repository

實測的結果，`nuget push` 並無法建立 Repository ，  
並且會導致發佈失敗，故必需 **優先建立 Packages 的 Repository** 。
Repository 可以包含多個專案，並獨立發佈(發佈指令詳見下文)。  

#### 建立 .nuget 文檔

請以專案名稱命名檔案，ex:`Marsen.Utility.nuget`  
請依實際情況調整 **id**、**version** 與 **projectUrl**  
特別是 **version** 代表的是 Packages 的版本
更多訊息請[參考](https://docs.microsoft.com/zh-tw/nuget/reference/nuspec)

```xml
<?xml version="1.0" encoding="utf-8"?>
<package xmlns="http://schemas.microsoft.com/packaging/2010/07/nuspec.xsd">
    <metadata>
    <id>Marsen.Utility</id>
    <version>1.0.0</version>
    <authors>Marsen Lin</authors>
    <description>Marsen Utility</description>
    <language>en-US</language>
    <projectUrl>https://github.com/marsen/Marsen.Nuget.Packages</projectUrl>
    </metadata>
</package>
```

### 取得授權 token

請參考[Configuring NuGet for use with GitHub Package Registry](https://help.github.com/en/articles/configuring-nuget-for-use-with-github-package-registry#authenticating-to-github-package-registry) 或是 [Creating a personal access token for the command line](https://help.github.com/en/articles/creating-a-personal-access-token-for-the-command-line)
**至少需要以下權限**

- [ ] repo Full control of private repositories
  - [ ] repo:status Access commit status
  - [ ] repo_deployment Access deployment status
  - [ ] public_repo Access public repositories
  - [ ] repo:invite Access repository invitations
- [ ] write:packages Upload packages to github package registry
- [ ] read:packages Download packages from github package registry

### 下載/安裝/設定 Nuget

1. [下載 Latest Nuget](https://www.nuget.org/downloads)，取得 nuget.exe
2. 設定 Path (以 Windows 10 為例)
    - 控制台
    - 系統設定>進階設定
    - 環境變數 > Path
3. 設定完 path 記得重啟 Terminal Session

### 新增 Nuget Source

#### 新增

Name 、 Source 與 UserName 網址請依實際調整參數
Password 請代入 token

```bash
nuget sources Add -Name "Marsen.Nuget sources" /
-Source "https://nuget.pkg.github.com/marsen/index.json" /
-UserName marsen -Password 3*******************************1
```

#### 設定 Key

```bash
nuget setapikey 3*******************************1 -Source "Marsen Nuget Sources"
```

#### 刪除

```bash
nuget sources Remove -Name "Marsen.Nuget sources"
```

#### 查詢

```bash
nuget sources
```

#### 打包

```bash
nuget pack Marsen.Package.csproj -OutputDirectory c:\local_nugets
```

#### 發佈

```bash
nuget push c:\local_nugets\Marsen.Utility.1.0.0.nupkg -Source "Marsen Nuget Sources"
```

### 結果

![GitHub Package Registry](https://i.imgur.com/Qz5Rv4c.jpg)

## 下一步

1. 每次都要打版號好麻煩，能不能自動化 ?
2. 透過 CI 建立，並傳入參數作為版號 ?
3. 每次 commit 只要通過 UT 測試就發佈.beta版 ? CI 發佈正式版 ?

more...

## 參考

- [Configuring NuGet for use with GitHub Package Registry](https://help.github.com/en/articles/configuring-nuget-for-use-with-github-package-registry#authenticating-to-github-package-registry)
- [Creating a personal access token for the command line](https://help.github.com/en/articles/creating-a-personal-access-token-for-the-command-line)
- <https://www.nuget.org/>
- <https://docs.microsoft.com/zh-tw/nuget/install-nuget-client-tools>
- <https://docs.microsoft.com/zh-tw/nuget/reference/nuspec>

(fin)
