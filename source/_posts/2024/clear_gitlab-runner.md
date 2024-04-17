---
title: " [實作筆記] 清理 CI/CD(Gitlab-runner) "
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

## 註解

1. Gitlab-Runner 是　Gitlab Solution 中執行工作的實體，可以是多台，此案只有單台
2. Dind: Docker In Docker,如同字面意思，在 Docker 中跑 Docker，是 Gitlab-runner 的實作方法之一

(fin)
