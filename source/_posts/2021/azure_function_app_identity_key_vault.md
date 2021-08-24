---
title: "[實作筆記] Azure Functions 存取 Key Vault"
date: 2021/08/24 08:51:43
tag:
    - 實作筆記
---

## 前情提要

新的工作有機會接觸到微軟的 Azure Cloud 服務,  
包含 AWS 、 GCP 的話，世界的三大雲服務都碰過一輪了,  
不過也只是皮毛而已, 紀錄下來免得遺忘 .

說明一下需求，在實作授權的

在這裡我會使用到 Azure Cloud 的三個服務,  

1. [Azure Functions](https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview)  
2. [Managed Identity](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview)  
3. [Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/general/overview)

### 使用的服務簡介 TL;DR  

>說明一下我理解的三個服務:
>  
Azure Functions 是微軟無伺服器(Serverless),  
Serverless 的概念是讓開發人員專注開發, 減少對伺服器維運的成本,  
並且提供極度彈性的可擴展性，並且可以使用更少的資源(更低的成本),  
最大的限制是[運算時間(timeout)](https://docs.microsoft.com/en-us/azure/azure-functions/functions-host-json#functiontimeout),  
但就我個人而言的 Best Practice 在 300 秒內完成不了的運算, 應考量整體架構的瓶頸在哪裡,  
而不是採取無限制的付費方案(Plan Type)。
稍微作個 ORK 評量，應該在 0.2 | 10 | 300 秒內完成計算。
>
>Managed Identity 是微軟用來作身份認証與授權的服務,  
與其它雲服務最大的差別應該是它是基於 Azure Active Directory (Azure AD) 之上,  
一般來說會建議使用遵循(Role-based Access Control),  
下面的圖片很好的說明了其概念。
>
>![如何使用適用於 Azure 資源的受控識別？](https://docs.microsoft.com/zh-tw/azure/active-directory/managed-identities-azure-resources/media/overview/when-use-managed-identities.png)
>
> Key Vault 是個相對簡單的概念, 在開發中我們會接觸到需多的密鑰、証書、連線字串等等資訊...  
這些資訊有敏感性, 但對開發過程又不是最重要的東西,  
這些資訊在單體架構(Monolithic)時常常面臨開發與資安的兩難,  
而 Key Vault 可以解決這個問題(當然不同的 Cloud 也有類似的解決方案)

### 案例說明

這次的案例，使用者會在瀏覽器提供給我們一組 Token ,
而我們會拿這組 Token 與第三方互動換回我們需要的 Key,  
最後我們會把這組 Key 存進 Key Vault 給其它服務使用.  
![流程簡介](../../images/2021/function_app_key_vault_flow.png)

你可能會好奇, 在上圖中 Managed Identity 扮演的角色為何 ?
就是在 Function Apps 存取 Key Vault 這一段,  
我們將提供一組 Identity 給 Function Apps 讓它有權限存取 Key Vault .

## 實作

我將步驟大致分為以下幾步

1. 建立 Function App 與 function
2. 啟用 Function App 的 Managed Identity
3. 建立 Key Vault
4. 設定 Key Vault Access policies  
5. 設定 Function App Configuration
6. 修改 Function

讓我們開始吧.

### 建立 Function Apps 與 function

開始前先作名詞解釋, 因為微軟的命名會讓剛接觸的人十分混淆  

- Azure Functions : 微軟的 Serverless 產品名稱, 我們可以在這個服務下建立許多 Function Apps 的實體
- Functions Apps : 主要運作的實體, 也是我們要設定的地方，每個 App 裡面可以有多個 function
- Functions : 主要邏輯所在的地方, 以開發的角度來說, 我們應該只關心這塊

[建立 Function Apps](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-function-app-portal#create-a-function-app) 的步驟較為簡單，請依照微軟文件即可，  
接下來選擇 Function Apps > 選擇剛剛建立的 Functions App > Create > Http Trigger C#,  
選擇這個範本我們將以 C# 語言實作，授權等級我選擇 Anonymous .  

建立好後，可以在 Code + Test 裡面查看預設的程式.  
這裡建立的是 C# 指令碼, 開發上的細節請參考[官方文件](https://docs.microsoft.com/zh-tw/azure/azure-functions/functions-reference-csharp#reusing-csx-code)

```csharp
using System.Net;

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
{
    log.Info("C# HTTP trigger function processed a request.");

    // parse query parameter
    string name = req.GetQueryNameValuePairs()
        .FirstOrDefault(q => string.Compare(q.Key, "name", true) == 0)
        .Value;

    if (name == null)
    {
        // Get request body
        dynamic data = await req.Content.ReadAsAsync<object>();
        name = data?.name;
    }

    return name == null
        ? req.CreateResponse(HttpStatusCode.BadRequest, "Please pass a name on the query string or in the request body")
        : req.CreateResponse(HttpStatusCode.OK, "Hello " + name);
}
```

### 啟用 Function App 的 Managed Identity

移動到 Functions Apps 的 Settings > Identity,
可以看 System assigned > Status 將它切換為 `On`.
這一步的設定是為了讓服務之間可以受到 AAD 控管而不需要你自行處理.  

### 建立 Key Vault

在建立之前也是需要一些名詞解釋，

- Key Vault: 微軟的服務名稱
- key vault section: Key Vault 的實體區塊(可以想像成一個實體檔案，裡面可以存很多 Key, 並且我們可以對這個檔案進一步設定,也可以參考[官方的文件快速建立](https://docs.microsoft.com/en-us/azure/key-vault/general/quick-create-portal))
- Keys : 在 key vault section 的 settings 中有三種類型的資料，Keys、Secrets 與 Certificates,

找到 Key Vault > key vault section > Settings > Secrets , 建立一組 Secret

![建立 Key Vault](../../images/2021/function_app_key_vault_create_key_vault.png)

點選剛剛建立的 Secret 並且選擇 CURRENT VERSION 取得 Secret Identifier(複製下來, 待會會用到)

### 設定 Key Vault Access policies  

一樣在 Key Vault > Settings > Access policies > Add access policy ,
在 Select principal 找到剛剛建立的 Function App,
在　Configure from template (optional) 我們直接套用 Secret Management,  
這允許我們的 Function App 會取得管理 Key Vault Secret 的權限.  

### 設定 Function App Configuration

移動到 Function App > Settings > Configuration > Application settings
我們要在這裡新加一個 Application Setting

Name 自已取但是請記得，我們稍後就會用到，這裡我先命名為 `TestKV`,  
Value 請參加下面的範例, 將前面步驟取得的 `Secret Identifier` 填到 SecretUri 之後. 　

```text
@Microsoft.KeyVault(SecretUri=https://mykv.vault.azure.net/secrets/RefreshToken/xxxx)
```

這裡有小朋友問我, 那為什麼不把資料設定在 Function App Configuration 就好,  
這樣子開發人員也碰不到資料, 要修改也是很有彈性的. 　
我的回答是, 這樣子其它的服務也需要這個資料怎麼辦呢？
Copy-Paste 會造成維護上很大的困難, 我們會希望維持一組就好.
放在 Key Vault 可以讓我們不同的服務共用. 　
![Overview](../../images/2021/function_app_key_vault_overview.png)

> 反思 Best Practice 應該怎麼作???
> 設定檔散落在不同的層級, 還是應該集中管理, 再用 Config 橋接???

### 修改 Function 的代碼

下列我們用 `TestKV` 這組設定與 C# 作為範例:

```csharp
// Get
var testKV =
  Environment.GetEnvironmentVariable("TestKV", EnvironmentVariableTarget.Process);
// do something update TestKV  
// Set
Environment.SetEnvironmentVariable("TestKV", testKV);
```

其它語言可以[參考](https://docs.microsoft.com/zh-tw/azure/azure-functions/functions-how-to-use-azure-function-app-settings?tabs=portal#use-application-settings)

## 參考

- [Azure Functions](https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview)  
- [Integrate Key Vault Secrets With Azure Functions](https://daniel-krzyczkowski.github.io/Integrate-Key-Vault-Secrets-With-Azure-Functions/)
- [淺析 serverless 架構與實作](https://denny.qollie.com/2016/05/22/serverless-simple-crud/)
- [Azure Function Tutorial](https://adamtheautomator.com/azure-functions-tutorial/)
- [Azure Functions: Extend Execution Timeout Past 5 Minutes](https://build5nines.com/azure-functions-extend-execution-timeout-past-5-minutes/)

(fin)
