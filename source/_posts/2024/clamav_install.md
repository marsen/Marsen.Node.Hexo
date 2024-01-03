---
title: " [實作筆記] ClamAV 安裝"
date: 2024/01/03 17:14:54
---

## 提要

因 ISO 需要在 GCP VM(Linux 系統)上安裝防毒。
ClamAV 是一款強大的開源防毒引擎，專為檢測和清除惡意程式而設計。  
這篇用來記錄終端機中安裝和配置 ClamAV，包括系統更新、病毒庫更新、自動運行設定等步驟，  
以確保系統有效抵禦各種威脅。  

## 實作

首先，在終端機中輸入以下指令，展開系統更新的步驟：

```bash
sudo apt update && sudo apt upgrade
```

出現警示請按 Y

```terminal
After this operation, 12.1 MB of additional disk space will be used.
Do you want to continue? [Y/n] Y
```

接著，透過以下指令安裝 ClamAV，一強大的防護工具，以確保系統免受惡意程式的侵害：

```bash
sudo apt install clamav clamav-daemon -y 
```

當安裝完成後

確保沒有其他 freshclam 進程在運行，您可以使用以下指令查看：

```bash
ps aux | grep freshclam
```

如果有其他 freshclam 進程在運行，請終止(kill)它們。

執行以下指令可更新 ClamAV 的病毒定義庫：

```bash
sudo freshclam
```

隨後，您可以進行系統掃描，以查找並清除潛在的威脅，請使用以下指令：

```bash
sudo clamscan -r /path/to/folder
```

若欲使 ClamAV 在系統啟動時自動運行，請執行以下指令：

```bash
sudo systemctl enable clamav-daemon
```

這將設定 ClamAV 在每次系統啟動時主動保護您的系統。

啟動 ClamAV 服務（如果它沒有在系統啟動時自動啟動）：

```bash
sudo systemctl start clamav-daemon
檢查 ClamAV 服務的運行狀態：

```bash
sudo systemctl status clamav-daemon
```

這樣，您可以確保 ClamAV 已經更新、服務已經啟動，並檢查服務的當前運行狀態。

## 參考

- <https://blog.yslifes.com/archives/3129>

(fin)
