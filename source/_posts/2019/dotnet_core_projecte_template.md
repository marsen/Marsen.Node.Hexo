---
title: "[實作筆記] 利用 .Net Core Project Template 加速開發"
date: 2019/09/01 20:46:46

---

## 要知道的事

這篇是實作的紀錄，大部份資源都參考至 MSDN 線上文件  
那裡會有更新更完整的文件支援。

## 前情提要

專案上作了一個跨市場的服務，以提供主服務對不同市場金流的支援。  
首先有一個國際化的專案，用來制定 API 層級的標準化，  
而在不同的市場，例如:馬來西亞、香港、烏克蘭等…。  
在地化的項目需要建立外掛(Plugin)專案的方式進行開發…  

建立一個全新的 Plugin 專案是一件麻煩的事，  
首先要加入專案，再來要實作相對應的介面，  
這裡指的是建立一個類別，然後繼承一堆介面，  
ex:IPay、IRefund、ICheck…等，  
並不包含商務邏輯的開發與實際的 API 或 SDK 串接，  
而在公司的標準之下，我還必須引入一堆參考或 Nuget 套件。 
同時因為國際化專案採用動態載入的機制，  
有一些細節準則需要由 Plugin 專案遵循，才不會引發異常或 Runtime Error。

## 實作筆記

### 建立專案樣版 SOP

1. 建立樣版資料夾，比如說 `Blog.Plugin.Template`
2. 準備好你的專案檔與相關的必要檔案

    這裡的檔案你可以拿現成專案作修改，我個人是從頭手工打造了一個全新的Plugin。  
    並且去除所有不相關不必要的資料夾、檔案、參考與程式。  
    簡單的來說，這是一個殼、一個專案樣版(Template)而已。
    接下來會遇到的問題「參數化」，比如說專案的名稱，資料夾的名稱，或是程式本身的內容，會依據不同的 Plugin 專案而改變的部份。  
    我把它分為檔案層級與內容層級的參數化。
    - 檔案層級參數化

        表示檔案名稱或是資料夾名稱等，我有動態抽換的需求。  
        我會用 `F-*` 作為參數名稱，比如說 `F-plugin-name` 
        接下來將資料夾或檔案名稱想被抽換的部份改成參數名稱就好，參考下圖。
        ![檔案層級參數化](https://i.imgur.com/wQVnR1h.jpg)

    - 內容層級參數化

        比如說 Class Name、Namespace、建構子或方法，甚至是預設的註解，  
        簡單的說只要是檔案內容相關的文字有必要被抽換，我就會使用這個參數設定。  
        我會直接使用 `C-*` 參數名稱，比如說 `C-plugin-name`。  
        下面是個例子。
        ![內容層級參數化](https://i.imgur.com/AhlYXRt.jpg)

3. 建立 `.template.config` 資料夾

4. 在`.template.config` 建立 `template.json`
     `template.json` 可以說是整個專案樣版的靈魂部份，更多的訊息可以在 MSDN 找到，  
     我只說明 `symbols` 節點的部份，它用來方置參數的訊息，所以需要建立一個自訂義的子節點，在這裡我命名為 `pluginName`，同時指定他的 type 為 parameter，  
     透過設定 `defaultValue` 給予預設值，最後將 `fileRename` 指定為 `F-plugin-name`，  
     將 `(C-plugin-name)` 指定為我的內容層級參數。

    ```json=
    {
    "$schema": "http://json.schemastore.org/template",
    "author": "MarkLin",
    "classifications": [ "Common", "SDK", "C#8" ],
    "identity": "Marsen.Blog.Plugin.Template",
    "name": "Marsen Blog Plugin Template",
    "shortName": "mbp",
    "tags": {
        "language": "C#",
        "type": "project"
    },  
    "preferNameDirectory": true,
    "symbols":{
        "pluginName": {
        "type": "parameter",
        "defaultValue": "Demo",
        "fileRename": "F-plugin-name",
        "replaces":"(C-plugin-name)"
        }
    }
    }
    ```

5. CLI強化，在`.template.config` 建立 `dotnetcli.host.json`
    這個檔案建立的目的，是為了讓我接下來使用 CLI 建立專案時能夠指定傳入的參數。
    這裡需指定 `template.json` 中的 symbol 並設定

    ```json=
    {
      "symbolInfo": {
          "pluginName": {
              "longName": "pluginName",
              "shortName": "p"
              }
        }
    }
    ```

6. 執行 `dotnet new i` 註冊樣版
    執行命令前請確定所在位置包含`.template.config` 資料夾與相關的檔案

    ```bash
    dotnet new -i ./
    ```

7. 執行 `dotnet new -u` 查詢樣版列表資訊確認安裝已完成

8. 應用

    在想建立專案的資料夾底下執行 `dotnet new <template_name>` 語法,  
    並且可以透過 CLI 指定參數，我並沒有研究 GUI 的操作，有興趣的人請自行研究，  
    如果能分享給我就不勝感激。

    ```bash
    dotnet new mbp -p Marsen2
    ```

9. 展示  

{% youtube RA_lHoa5uuI %}

我把這個例子放在 [Github](https://github.com/marsen/dotnet.core.project.template.sample) 上，有興趣的朋友可以試試看。

## 其它

### 移除專案樣板 SOP

1. 執行 `dotnet new -u` 查詢樣版列表資訊，可以查到專案樣版的絕對路徑
2. 執行 `dotnet new -u <ABSOLUTE_FILE_SYSTEM_DIRECTORY>` 

### 更新專案樣版

直接進入專案樣版的絕對路徑

### 發佈

略…

## 參考

- [我的範例](https://github.com/marsen/dotnet.core.project.template.sample)
- [教學課程：建立項目範本](https://docs.microsoft.com/zh-tw/dotnet/core/tutorials/cli-templates-create-item-template)
- [教學課程：建立專案範本](https://docs.microsoft.com/zh-tw/dotnet/core/tutorials/cli-templates-create-project-template)
- [dotnet/dotnet-template-samples](https://github.com/dotnet/dotnet-template-samples)
- [Tips for developing templates for dotnet new](https://www.jerriepelser.com/blog/tips-for-developing-dotnet-new-templates/)
- [JSON 結構描述保存區的 template.json 結構描述](http://json.schemastore.org/template)
- [How to name (or rename) files based on a parameter? · Issue #1238 · dotnet/templating](https://github.com/dotnet/templating/issues/1238)
- [Tips for developing templates for dotnet new](https://www.jerriepelser.com/blog/tips-for-developing-dotnet-new-templates/)

(fin)
