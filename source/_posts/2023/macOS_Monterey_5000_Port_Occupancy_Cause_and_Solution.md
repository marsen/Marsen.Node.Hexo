---
title: "[AI 共筆] macOS Monterey 5000 Port 佔用原因與解方"
date: 2023/03/15 15:38:40
---

## 前言

在 macOS Monterey 上如何處理開發伺服器使用端口被佔用的問題，  
若使用其他 macOS 版本，請查看相關頁面。

## 問題

更新到最新的 macOS 作業系統時，發現我的前端網站，  
顯示類似於「Port 5000 already in use」的訊息。

透過執行  

```terminal
lsof -i :5000
```

發現正在佔用該端口的進程名稱為 ControlCenter，這是一個原生的 macOS 應用程式。  
即使你使用強制終止它，它還是會自動重新啟動。  

## 處理方式

原來使用該端口的進程是 AirPlay 伺服器，  
你可以在「系統偏好設定」 >「共享」中取消勾選「AirPlay Receiver」以釋放 5000 端口。

## 小結

網路大多的中文解法是關閉"隔空播放接收器"，在新版的 OS 找不到，可能是語言差異。  
所幸找到英文的解法，順利停止佔用 port 的服務。

## 參考

- [[Solved] Port 5000 Used by Control Center in macOS Monterey](https://nono.ma/port-5000-used-by-control-center-in-macos-controlce)

(fin)
