---
title: "[踩雷筆記] Azure Function 開關虛據機與錯誤排除"
date: 2024/08/12 14:09:10
---

## 前情提要

在 Azure Functions 與 Queue 架構，來開關虛擬機器節省費用。因為這些有 GPU 的機器很貴，而建立 AI 服務又不能沒有它們。

## 實作記錄

一開始的構想很簡單：透過 Queue 接收特定的工作項，並由 Azure Function 來判斷所需要的資源，如果需要 AI 服務，就需要將虛擬機器的打開，
並由另一個程式進行後續工作。當工作完成後，再將機器關機。

程式(python)大致如下,內含開關機的實作

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

### 雷:EnvironmentCredential

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

表面的原因是權限不足，底層的原因是需要為 Azure Function 配置環境變數，才能取到權限。

如果要使用 Service Principal 的 Client Secret，配置  

- AZURE_CLIENT_ID
- AZURE_TENANT_ID
- AZURE_CLIENT_SECRET

如果要使用 Service Principal 的　Certificate 驗證，配置  

- AZURE_CLIENT_ID
- AZURE_TENANT_ID
- AZURE_CLIENT_CERTIFICATE_PATH
- AZURE_CLIENT_CERTIFICATE_PASSWORD (Optional)

若要使用密碼進行用戶身份驗證，配置

- AZURE_USERNAME
- AZURE_PASSWORD

我選擇第一種配置，需要先加上 App registrations
具體流程如下：

在 Azure Portal 上建立一個新的 Azure Registration。  
選擇 Certificates & secrets，建立一組 Certificates & secrets。  
接下來到 Azure Function　Apps 設定環境變數 AZURE_CLIENT_ID 與　AZURE_CLIENT_SECRET,  
Tenant_ID 可以在透過 Azure Portal 找 Tenant Properties 查詢，一樣設定到環境變數。  
接下來將 Registration 設定為 Virtual Machine 的 Contributor  
(待確認開關機是不是有更小的權限？ex:Virtual Machine Contributor、Virtual Machine Operator：)  
如此一來在開關機時就能有足夠的權限。


## 參考

- <https://aka.ms/azsdk/python/identity/environmentcredential/troubleshoot>
- <https://github.com/Azure/azure-sdk-for-python/blob/main/sdk/identity/azure-identity/TROUBLESHOOTING.md#troubleshoot-environmentcredential-authentication-issues>
- <https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview?pivots=programming-language-csharp>

(fin)
