---
title: " [實作筆記] 一些關於 Azure Resource Group 的冷知識"
date: 2024/01/25 16:22:17
---

## 前情提要

Resource Group 是 Azure 上比較特別的一個設計，  
這裡拿來記錄一些知道就知道，不知道就不知道的小事務。

## 本文

### NetworkWatcherRG

- <https://learn.microsoft.com/zh-tw/azure/network-watcher/network-watcher-create?tabs=portal#delete-a-network-watcher-in-the-portal>

> 注意
>  
> 當您使用 Azure 入口網站 建立網路監看員實例時：
>  
> 網路監看員實例的名稱會自動設定為NetworkWatcher_region，其中region會對應至 網路監看員 實例的 Azure 區域。  
> 例如，在美國東部區域中啟用的網路監看員名為NetworkWatcher_eastus。  
> 網路監看員實例會建立在名為NetworkWatcherRG的資源群組中。 若尚無該資源群組，將會加以建立。

### Azure DevOps

- <https://learn.microsoft.com/en-us/azure/devops/organizations/billing/set-up-billing-for-your-organization-vs?view=azure-devops>

當我們將 Azure DevOps 的 Billing 綁定之時，  
會建立一組 `VisualStudioOnline-XXXX` 的 Resource Group

(fin)
