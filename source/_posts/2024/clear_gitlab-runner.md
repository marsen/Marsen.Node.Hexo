---
title: " [實作筆記] 清理 CI/CD (Gitlab-runner) "
date: 2024/04/17 10:25:21
tags:
  - 實作筆記
  - CI/CD
---

## 寫在前面

NM;DR
沒有什麼意義，不需要讀這篇

## 緣由

- CI/CD 執行失敗，檢查錯誤發現是 Gitlab-runner[*1] 主機空間不足。  

## 處理歷程

- 連線 Gitlab-runner 主機
- 查詢空間使用狀況，找到佔磁區的資料
  > marsen@gr00:~$  `sudo du -h --max-depth=1 /`  
  > 24K /tmp  
  > 0 /sys  
  74M /boot  
  ... skip ...
  24K /root  
  94G /var  
  97G /
- 用指令作深層的搜尋　`sudo du -h --max-depth=3 /var`
- 發現 docker 的問題最大
  >  252K /var/lib/docker/containers
  > 4.0K /var/lib/docker/trust
  > 4.0K /var/lib/docker/runtimes
  > 91G /var/lib/docker
- 不選擇清理磁碟，而用 docker 指令檢查 `docker images`
- 批次刪除指令　`docker images -f "dangling=true" -q | xargs docker rmi`
- 再次檢查
  > marsen@gr00:~$  `sudo du -h --max-depth=1 /`  
  > 24K /tmp  
  > 0 /sys  
  74M /boot  
  ... skip ...
  24K /root  
  16G /var  
  20G /

## 後續

- 可能要建立一個排程定期去清理 CI/CD 無用的 image
  - 可以的話想整入 CI/CD 作業中，但在 DinD[*2] 的環境不確定怎麼實作
- log 與 cache 也有看到較高的資料成長曲線，未來也要納入評估

### 20240829 更新設置定期清理腳本

建立 cleanup.sh，並將其放置在 root 根目錄下：

```bash
sudo cp /path/to/cleanup.sh /root/
sudo chmod +x /root/cleanup.sh
```

cleanup.sh 的內容：

```bash
#!/bin/bash

# 清理未使用的 Docker 資源
sudo docker system prune -a --force --volumes

# 清理未使用的 Docker 卷
sudo docker volume prune --force

# 縮減系統日誌大小
sudo journalctl --vacuum-time=30d
```

#### 設置排程

以 root 用戶創建定期排程以每月執行清理腳本：

打開 cron 編輯器：

```bash
sudo crontab -e
```

在 cron 編輯器中添加以下行：

```bash
0 0 1 * * /root/cleanup.sh > /root/cleanup.log 2>&1
```

- 0 0 1 * *：每月的第一天午夜執行。
- /root/cleanup.sh：執行清理腳本。
- > /root/cleanup.log 2>&1：將輸出和錯誤重定向到 /root/cleanup.log。

#### 驗證和檢查

手動運行清理腳本以驗證它是否按預期工作：

```bash
sudo /root/cleanup.sh
```

#### 查看 cron 日誌

檢查 cron 日誌以確保定期任務已成功運行：

在 Ubuntu/Debian 系統上：

```bash
sudo grep CRON /var/log/syslog
```

使用 journalctl（如果使用 systemd）：

```bash
sudo journalctl -u cron
```

## 註解

1. Gitlab-Runner 是　Gitlab Solution 中執行工作的實體，可以是多台，此案只有單台
2. Dind: Docker In Docker,如同字面意思，在 Docker 中跑 Docker，是 Gitlab-runner 的實作方法之一

(fin)
