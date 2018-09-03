---
title: CI/CD 環境建置筆記 - 使用Openshift 的 Jenkins (失敗)
date: 2017/01/15 22:16:42
tag:
  - CI/CD
  - Jenkins
  - Openshift
---

## 前情提要

- [CI/CD 環境建置筆記 - 前言<目的>](/2017/01/15/ci_use_jenkins/)

## 操作記錄(失敗)

### 目標: 

- 在 Openshift 新增一個 Jenkins Server

### 失敗原因: 

- 我找不到方法在 Openshift 上建置的 Jenkins Server 

### 記錄:

1. 登入OpenShift的[web console](https://openshift.redhat.com/app/console/applications)
2. Add Application 選擇 `Jenkins Server`，使用預設設定Create Application.  
Public URL 設定為 http://jenkins-youraccount.rhcloud.com
3. 連線進 http://jenkins-youraccount.rhcloud.com 會發現需要帳密登入
4. 取得帳號密碼
	1. 使用 SSH 連線 Openshift 的 Jenkins Server (可以在web console 查到連結)
		`> ssh 4263*****************@jenkins-youraccount.rhcloud.com`
	2. 查看以下兩個檔案可以取得帳號密碼
	`JENKINS_USERNAME`
	`JENKINS_PASSWORD` 
	
	 
5. 登入後，管理 Jenkins > 設定系統
	1. 將執行程式數量設定為 1 
6. 選擇新增作業
	1. 第一步要將原始碼自Github pull下來;在原始碼管理選擇`Git` , 設定好`Repositories`、`Branches to build`
	
7. **執行一次建置**，這個步驟是為了產生work space 。 
8.  SSH 連線 Openshift，切換目錄到你的專案的work space 
	`> ssh 4263*****************@jenkins-youraccount.rhcloud.com`
	`> cd app-root/data/workspace/your_project_name`
9.  檢查一下目前Git的遠端Repo有哪些
	`> git remote -v`
10. 將Openshift上的nodejs應用程式repo設為remote
	`>git remote add openshift ssh://5**********************@nodejs-youraccount.rhcloud.com/~/git/nodejs.git/`
11. 推送到`openshift remote`
失敗!!原因是權限問題，Openshift remote 是透過 SSH 連線，
在認証公鑰的過程中，需要寫入 `~/.ssh/known_hosts` 檔案。
但是 Openshift 建立的 Jenkins Server 登入帳號，權限並不足以寫入而造成失敗

