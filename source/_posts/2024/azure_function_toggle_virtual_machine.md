---
title: "[踩雷筆記] Azure Function 開關虛據機與錯誤排除"
date: 2024/08/12 14:09:10
---

## 前情提要

在 Azure Functions 和 Queue 的架構下，我們試圖通過自動化開關虛擬機器來節省成本。　　
這是因為 GPU 設備的價格相當高昂，而我們的 AI 服務又離不開這些昂貴的機器。

實作記錄

## 實作記錄

最初的構想非常簡單：通過 Queue 接收特定的任務，然後由 Azure Function 根據需要決定資源的使用。  
如果需要進行 AI 計算，就必須啟動虛擬機器，而在任務完成後，再將機器關閉。  
這樣的方式不僅能夠節省資源，還能有效控制成本。

以下是 Python 程式碼的大致實作，主要為虛擬機器的開關控制邏輯：

```python
import azure.functions as func
from azure.identity import DefaultAzureCredential
from azure.mgmt.compute import ComputeManagementClient
import logging, os

app = func.FunctionApp()

@app.queue_trigger(arg_name="args", queue_name="vm-queue", connection="CONNECTION_STRING") 
def main(args: func.QueueMessage):
    logging.info('Queue Msg: %s', args.get_body().decode('utf-8'))
    
    try:
        credential = DefaultAzureCredential()
        subscription_id = os.getenv("SUBSCRIPTION")
        resource_group = os.getenv("RG")
        vm_name = os.getenv("VM_NAME")

        compute_client = ComputeManagementClient(credential, subscription_id)
        vm = compute_client.virtual_machines.get(resource_group, vm_name, expand='instanceView')
        vm_status = vm.instance_view.statuses[1].display_status
        logging.info('VM 狀態: %s', vm_status)
        
        if vm_status == "VM running":
            logging.info('正在關閉VM: %s', vm_name)
            operation = compute_client.virtual_machines.begin_deallocate(resource_group, vm_name)
            operation.result()
            logging.info('%s 已關閉', vm_name)
        elif vm_status in ["VM deallocated", "VM stopped"]:
            logging.info('正在啟動VM: %s', vm_name)
            operation = compute_client.virtual_machines.begin_start(resource_group, vm_name)
            operation.result()
            logging.info('VM %s 已啟動', vm_name)
        else:
            logging.info('非預期的 VM 狀態: %s', vm_status)

    except Exception as e:
        logging.error('異常錯誤: %s', str(e))

```

### 異常問題:EnvironmentCredential

```shell
DefaultAzureCredential failed to retrieve a token from the included credentials.
Attempted credentials:
 EnvironmentCredential: EnvironmentCredential authentication unavailable. Environment variables are not fully configured.
Visit https://aka.ms/azsdk/python/identity/environmentcredential/troubleshoot to troubleshoot this issue.
 ManagedIdentityCredential: ManagedIdentityCredential authentication unavailable, no response from the IMDS endpoint.
 SharedTokenCacheCredential: SharedTokenCacheCredential authentication unavailable. No accounts were found in the cache.
 AzureCliCredential: Azure CLI not found on path
 AzurePowerShellCredential: PowerShell is not installed
 AzureDeveloperCliCredential: Azure Developer CLI could not be found. Please visit https://aka.ms/azure-dev for installation instructions and then,once installed, authenticate to your Azure account using 'azd auth login'.
To mitigate this issue, please refer to the troubleshooting guidelines here at https://aka.ms/azsdk/python/identity/defaultazurecredential/troubleshoot.
```

**這個問題表面上看似由於權限不足引起的，但實際上根本原因是缺乏正確的環境變數配置。**  
要使 Azure Function 能夠正常運作並獲得所需的權限，我們需要在 Function 的環境設置中正確配置相應的環境變數。  
這些環境變數包括關鍵的憑證和授權信息，它們使得 Azure Function 能夠在執行過程中獲取必要的存取權限，從而能夠正常與 Azure 資源進行交互。  
如果環境變數配置不當或缺失，Azure Function 將無法獲得所需的授權，從而導致權限不足的錯誤。  
確保這些環境變數被正確設置和管理，是解決此類問題的關鍵步驟

配置的方式有三種

#### 如果要使用 Service Principal 的 Client Secret，配置  

- AZURE_CLIENT_ID
- AZURE_TENANT_ID
- AZURE_CLIENT_SECRET

#### 如果要使用 Service Principal 的　Certificate 驗證，配置  

- AZURE_CLIENT_ID
- AZURE_TENANT_ID
- AZURE_CLIENT_CERTIFICATE_PATH
- AZURE_CLIENT_CERTIFICATE_PASSWORD (Optional)

#### 若要使用密碼進行用戶身份驗證，配置

- AZURE_USERNAME
- AZURE_PASSWORD

我選擇第一種配置，需要先加上 App registrations
具體流程如下：

- 在 Azure Portal 上建立一個新的 Azure Registration。  
- 選擇 Certificates & secrets，建立一組 Certificates & secrets。  
- 到 Azure Function　Apps 設定環境變數 AZURE_CLIENT_ID 與　AZURE_CLIENT_SECRET,  
- Tenant_ID 可以在透過 Azure Portal 找 Tenant Properties 查詢，一樣設定到環境變數。  
- 接下來將 Registration 設定為 Virtual Machine 的 Contributor  
  (待確認開關機是不是有更小的權限？ex:Virtual Machine Contributor、Virtual Machine Operator：)  

如此一來在開關機時就能有足夠的權限。

## 參考

- <https://aka.ms/azsdk/python/identity/environmentcredential/troubleshoot>
- <https://github.com/Azure/azure-sdk-for-python/blob/main/sdk/identity/azure-identity/TROUBLESHOOTING.md#troubleshoot-environmentcredential-authentication-issues>
- <https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview?pivots=programming-language-csharp>

(fin)
