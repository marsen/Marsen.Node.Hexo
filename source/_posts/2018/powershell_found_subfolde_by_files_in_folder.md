---
title: "[實作筆記]用 Command Line 取得資料夾內包含特定檔案的子資料夾"
date: 2018/11/23 13:43:00
tag:
    - powershell
---

## 情境
- OS : Windows 10

在一個巨型的 Git Repo 當中，底下依專案分了許多專案資料夾，  
參考下圖。  
![擁有多個專案的Repo](https://i.imgur.com/IabNBFa.jpg)

而今天發生了一件事情，我需要更新某幾個子專案的 POCO 的 .tt 檔，  
這裡不說明 POCO 是什麼；簡單的說，有的專案會有 .tt 檔，  
有的專案會沒有，而每個專案的資料夾結構又不一定相同，  
所以要找出這些 .tt 是有點麻煩的，另外我的目標並不是 .tt 檔，  
而是所在的專案，再用 IDE 開啟進行修改，  
為此我需要列出**專案資料夾**  

## 目標 

用 Command Line 取得Repo資料夾內包含.tt檔案的專案資料夾名稱  

## 作法

```shell
D:\Repo\Taiwan\******.******.Repofolder
λ Get-ChildItem -Path .\ -Filter *.tt -Recurse -File -Name | ForEach-Object { $_.Split('\')[0] } | Group {$_} | select name
```
```
Name  
----  
FacebookShop  
LineOrderFinish  
LineOrderNotify  
Mail  
NMQMonitor  
OrderMonitor  
SyncImageToOthers
```

## 小結
總覺得寫得有點又臭又長，希望有更好的作法可以提供給我，
不限於 `powershell` 就算是 Linux 的語法也可以讓我參考一下。

(fin)