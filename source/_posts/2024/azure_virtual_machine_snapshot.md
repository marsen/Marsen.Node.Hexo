---
title: " [實作筆記] 建立 Azure VM 的快照(Snapshot)"
date: 2024/07/01 13:02:14
---

## 前情提要

這次記錄實作 Azure VM 的快照(Snapshot)與還原，與 VM images 不同，  
快照是針對磁碟的即時捕捉，而不是整個虛擬機器的範本。  
VM images 用於創建新的虛擬機器，是在固定的狀態下保存整個 VM，  
包括操作系統、應用程式和所有配置。  

相比之下，快照則僅保存特定磁碟的狀態，適合於快速還原或備份目前正在運行的 VM。  
選擇快照主要因為它的快速性和簡便性，可以在短時間內恢復 VM 到指定狀態，  
以我的例子來說，我因為 AI 需求建了昂貴的 Azure VM ，使用 Image 會會佔用我們的 Quotas。  
所以 Snapshot 會是較好的還擇

## 前置作業

在開始建立快照之前，請確保以下條件已經準備妥當：

1. 已經擁有一個可操作的 Azure 帳戶並能夠登入 Azure 入口網站。
2. 目標 VM 已經啟動並運行正常，且確定需要為其建立快照。
3. 已經分配足夠的儲存空間來保存快照數據。

## 實作步驟

### 建立 Snapshot

1. 登入 [Azure 入口網站](https://portal.azure.com) 並進入虛擬機器(Virtual Machines)頁面。
2. 從虛擬機器列表中，選擇您想要建立快照的 VM。
3. 在左側導航欄中，找到並點擊 "磁碟(Disks)"。
4. 在磁碟頁面中，選擇想要建立快照的磁碟(通常是 OS Disk)。
5. 點擊上方的 "快照(Snapshot)" 按鈕，進入快照配置頁面。
6. 在快照配置頁面中，輸入快照名稱，並選擇儲存帳戶和資源群組。
7. 檢查所有設定無誤後，點擊 "建立(Create)" 按鈕，開始建立快照。這個過程可能需要幾分鐘。
8. 快照建立完成後，就可以在資源群組或儲存帳戶中找到該快照，並隨時使用它來還原 VM 的狀態。

### 從 Snapshot 到 OS Disk 到 VM

1. 在 Azure 入口網站中，進入 "快照(Snapshots)" 頁面，選擇您之前建立的快照。
2. 點擊快照名稱進入詳細資訊頁面，然後選擇 "建立磁碟(Create Disk)"。
3. 在磁碟配置頁面中，輸入磁碟名稱，並選擇合適的儲存帳戶和資源群組。
4. 檢查所有設定無誤後，點擊 "建立(Create)" 按鈕，開始將快照轉換為磁碟。
5. 磁碟建立完成後，進入 "磁碟(Disks)" 頁面，選擇您剛建立的磁碟。　　
6. 在磁碟詳細資訊頁面中，點擊上方的 "建立 VM(Create VM)" 按鈕。
7. 在虛擬機器配置頁面中，完成其他配置，如名稱、大小、網路設定等，然後點擊 "檢閱 + 建立(Review + create)" 按鈕。　　
8. 檢查所有設定無誤後，點擊 "建立(Create)" 按鈕，開始建立新的虛擬機器。　　
9. 新的虛擬機器建立完成後，登入並驗證其狀態是否與快照拍攝時一致。　　

## 問題

我發現使用 Security Type 為 Trusted Launch 的 VM 所建立的 Snapshot；　　
與其建立的 Disk 與 VM 其 Security Type 也會是 Trusted Launch。　　
但是當我嚐試透過　SSH using Azure CLI 連線 VM，  
會遇到無法連線的問題，同樣的步驟，Security Type 為 Standard 就不會有問題。　　
待確認原因…　　

## 參考

1. [Microsoft 官方文件: 建立和管理虛擬機器磁碟的快照](https://learn.microsoft.com/zh-tw/azure/virtual-machines/windows/snapshot-copy-managed-disk)
2. [Azure 入口網站指南](https://docs.microsoft.com/zh-tw/azure/azure-portal/)
3. [Azure 虛擬機器文檔](https://docs.microsoft.com/zh-tw/azure/virtual-machines/)

(fin)
