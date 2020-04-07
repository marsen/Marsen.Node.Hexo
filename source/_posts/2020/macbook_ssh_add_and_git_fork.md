---
title: "[實作筆記] Macbook SSH 設置與疑問 "
date: 2020/03/10 02:02:31
---

## 前情提要

最近轉換了一下跑道 ，  
剛好又了一點時間就想說順便換一下 OS 學一點新東西，
就敗了一台 Macbook ， 這幾天就忙著整理開發環境 。  
在 ssh 上卡住了點 ， 就作點記錄順便上來提問 。

## 情境

我安裝了 [git-fork](https://git-fork.com/) 作為我的 Git GUI 工具。
裡面有一個很方便的功能可以快速的設定 ssh ，  

![fork](/images/2020/3/030901_fork_setting_ssh.png)  

於是我很輕鬆娛快的設定了一組 ssh key 我命名為 `Macbook`  
也可以很正常的在 fork 裡面作一些 git 的操作 ，  
比如說 fetch push pull 等。

不過人在江湖飄， 哪有不挨刀。　　
身為一個開發者與一個 Git 的愛用者，  
有很多情境是會離開 GUI 工具操作 Git 的 。
但是很奇怪，只要離開了 fork 我的 Git 部份指令就會失效，
錯誤訊息如下

```sh
git@github.com: Permission denied (publickey).  
fatal: Could not read from remote repository.  
Please make sure you have the correct access rights and the repository exists.
```

很明顯是權限不足的原因。
但是我查找了 `~/.ssh` 資料夾 ，  
我設定的 private key `Macbook` 確實存在。  

所以我透過一個指令來取得更多資訊

`ssh -vT git@github.com`

輸出如下

```text
marsen@MarsendeMacBook-Pro ~ % ssh -vT git@github.com
OpenSSH_7.9p1， LibreSSL 2.7.3
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: /etc/ssh/ssh_config line 48: Applying options for *
debug1: Connecting to github.com [192.30.253.112] port 22.
debug1: Connection established.
debug1: identity file /Users/marsen/.ssh/id_rsa type -1
debug1: identity file /Users/marsen/.ssh/id_rsa-cert type -1
中間省略...  
debug1: Trying private key: /Users/marsen/.ssh/id_ecdsa
debug1: Trying private key: /Users/marsen/.ssh/id_ed25519
debug1: Trying private key: /Users/marsen/.ssh/id_xmss
debug1: No more authentication methods to try.
git@github.com: Permission denied (publickey).
```

幾個奇怪的地方
一個是 identity file 或是 Trying private key 的檔案我都找不到 ，
另一個奇怪的點是 `~/.ssh/Macbook` 這組我剛剛建立的 private key 明明存在 ，  
我確找不到他顯示在 identity file 或是 Trying private key 的記錄之中
總而言之，最後的訊息仍然是

```sh
git@github.com: Permission denied (publickey).
```

## 解決方法

指令 `ssh-add -K Macbook` 執行後， 就可正常運作。
小小猜測一下 ， 應該是 fork 運作的環境與一般 terminal 的環境有所差異，  
所以需要額外透過指令加上 private key 才能夠執行。

不過我仍有疑問， fork 的設定在什麼地方可以調整呢 ?
另一個問題是 log 裡面這些不存在 private key 是在哪裡設定的 ?
查找過 `/etc/ssh/ssh_config` 裡面並沒有相關的設定

```text
debug1: Will attempt key: /Users/marsen/.ssh/id_rsa
debug1: Will attempt key: /Users/marsen/.ssh/id_dsa
debug1: Will attempt key: /Users/marsen/.ssh/id_ecdsa
debug1: Will attempt key: /Users/marsen/.ssh/id_ed25519
debug1: Will attempt key: /Users/marsen/.ssh/id_xmss
```

~~希望有人能有答案或提供文件可以參考一下。  
不求甚解繼續玩 Mac 去。~~

## 後記

20200407 : 在安裝 git-fork 的時候 ， 它會另外安裝一個 git 實體 ，  
所以與 global 環境的 git 不會是同一個 ， 導致在 git-fork 的行為不一致 。  
另外之所以使用 ssh 作為連線手段 ， 是因為以前使用 SourceTree 作為 Git Gui Tool 時  
常常會 Https 的密碼過期而需要重新輸入 ， 如果信任 git-fork 的開發者 ，  
可以考慮使用 Https 連線 ， 看起來 git-fork 會記錄你的連線資訊 ，  
而不會在每次操作 git 時詢問你密碼。

(fin)
