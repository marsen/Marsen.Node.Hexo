---
title: CI/CD 環境建置筆記(二) - 在 windows 安裝 Jenkins 之旁門左道私有雲
date: 2017/04/29 11:55:16
tag:
  - CI
  - Jenkins
---

## 應該要知道的事

- Dropbox
- Windows Service
- [CI/CD 環境建置筆記(一) - 在 windows 安裝 Jenkins ](/2017/01/15/ci_use_jenkins/)

## 問題

在 windows 上安裝了Jenkins , 也建立了一個新的作業 ,
讓程式可以透過 Jenkins 執行自動化部署。
不過實務上仍有缺點，那就是只有這台機器可以執行部署 ,
最理想的狀況是擁有一個在雲端的 CI Server ,
隨時隨地想部署只需要能連上網 , 登入執行就好
這部份可以試試看 `Travis-CI`、`CircleCI`之類的服務 ;
不過對於只熟悉 Jenkins 的我來說 ,
我想了另一套解決辦法(旁門左道) ,
不過至少解決了我目前的需求 ,
在公司與家中隨時都能透過本機Jenkins Server 進行部署 , 
並且不用花時間同步設定值. 
在花時間研究`Travis-CI`、`CircleCI`的部署方式之前 ,
算是一個折衷的方式 .

## 方案
1. 首先你要有 Dropbox 
2. 下載 Windows 版的 Jenkins
3. 安裝在 Dropbox 資料夾內
4. 在另外一台電腦 , 進行相同的安裝

想法很簡單 , 透過 Dropbox 與 Jenkins Service
在兩台電腦安裝 Jenkins
只要將 Jenkins Service 的啟動路徑
設定在 Dropbox 中 , 
就可以達成我們的目標 .
![](https://i.imgur.com/DyrNMJD.jpg)

## 問題 
1. Windows 要如何修改 Jenkins Service 的啟動路徑 ?

`HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Jenkins`

修改路徑以符合你本機的檔案位置即可。

![](https://i.imgur.com/KGemCSe.jpg)

(fin)