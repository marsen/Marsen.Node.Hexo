---
title: CI/CD 環境建置筆記(一) - 在 windows 安裝 Jenkins 
date: 2017/01/15 22:16:42
tag:
  - CI
  - Jenkins
  - Openshift
---

## 前情提要

- [CI/CD 環境建置筆記 - 前言<目的>](/2017/01/15/ci_use_jenkins/)

## 操作記錄

1. 下載並安裝Jenkins（記錄版本為2.32.1）
2. 連線localhost:8080,會要求輸入`Administrator password`
![](https://i.imgur.com/ik5l0sq.jpg)

3. 安裝Plugins(這裡選擇預設)
![](https://i.imgur.com/dOad35P.jpg)


4. 安裝畫面
![](https://i.imgur.com/6FyGHPm.jpg)

5. 建立管理者帳號密碼
![](https://i.imgur.com/AzHrJdu.jpg)
![](https://i.imgur.com/2ct8GiO.jpg)

6. 登入後,管理 Jenkins > 設定系統
	1. 將執行程式數量設定為 1  (安裝完預設為2,其實可以不用修改)
	2. `Shell` 設定為 `C:\Windows\system32\cmd.exe` 

7. 建立第一個作業,選擇新增作業 > 輸入作業名稱 ,選擇「建立多重設定專案」
![](https://i.imgur.com/0xkci16.jpg)
8.  **執行一次建置**,這個步驟是為了產生work space 。
work space 路徑大致如下 `.\Jenkins\workspace\Project name`

9. 執行 console 並切換路徑至 work space 
10. 將 Openshift 上的 nodejs 應用程式 repo 設為 remote
	`>git remote add prod ssh://5**********************@nodejs-youraccount.rhcloud.com/~/git/nodejs.git/`
	
11. 回到Jenkins,作業 > 組態 > 建置 > 新增「執行Windows批次指令」
	
	``` bat
	REM 測試
	whoami
	git push prod HEAD^:master
	```

	1. 額外處理事項:
	直接在 Jenkins server 發 pr 給 openshift 時,發生 503 錯誤。
	使用 ssh 登入 openshift 看 log ,發現 
	`Node Sass does not yet support your current environment` 錯誤
	必須登入執行以下語法修正模組問題 `npm rebuild node-sass` 
	
12. 建置作業,會得到錯誤訊息
```
上略...
19:44:46 Host key verification failed.
19:44:46 fatal: Could not read from remote repository.
下略...
```
	
原因:主機密鑰驗證失敗,這個錯誤的意思是我的 Jenkins Server 主機並不認得遠端的 Openshift Server 的 host key,主要的原因是 Jenkins Service 在執行的身份是 `NT AUTHORITY\SYSTEM`,由於我已經有合適權限的帳號,所以只需要切換執行 Jenkins Server 的身份即可。
![](https://i.imgur.com/HSJoXJp.jpg)

如果你尚未建立遠端ssh連線的存取權限,或是對ssh連線不熟悉,可以參考文末的參考聯結。

## 參考

- [SSH原理与运用（一）：远程登录](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)
- [SSH原理与运用（二）：远程操作与端口转发](http://www.ruanyifeng.com/blog/2011/12/ssh_port_forwarding.html)
- [[Tip] Windows使用ssh對Github進行操作](https://dotblogs.com.tw/kirkchen/2013/04/23/use_ssh_to_interact_with_github_in_windows)
- [在 Windows 使用「非對稱金鑰」來遠端登入 SSH 的方法](http://www.vixual.net/blog/archives/190)

(fin)