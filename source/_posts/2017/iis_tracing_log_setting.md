---
title: IIS Tracing Log 設定
date: 2017/01/24 14:27:12 
tag:
  - IIS
  - 記錄
---
## 緣由
當你需要Trace送到IIS的每個Request的細節(包含在pipe line各個模組之間的input/output 與 傳入的參數、header等…), 
可以透過開啟tracing log來作追蹤

## 環境
1. Windows Server 2012 R2
2. IIS 8.5.9600.16384

## 開啟 tracing log

1. 新增 Tracing Rules 
![](https://i.imgur.com/4llAgOa.jpg)
![](https://i.imgur.com/BavVoWy.gif)


2. 開啟網站 Tracing 功能
![](https://i.imgur.com/LUgbESR.jpg)
![](https://i.imgur.com/aKtgj5m.jpg)

## 調整 IE Security 層級

1. 使用 IE 開啟 Log，因 Security 設定無法套用版型
![](https://i.imgur.com/4VsuQWp.jpg)

2. 調整ie enhanced security

![](https://i.imgur.com/Lvnygqr.jpg)

3. 開啟IE > 點選小齒輪，選擇  Internet Options > 點選 Security 頁籤 > 將 Internet 的 Security Level 調整為 Medium-high
![](https://i.imgur.com/H2F06OU.jpg)

4. 完成
![](https://i.imgur.com/biiX6w8.jpg)



(fin)