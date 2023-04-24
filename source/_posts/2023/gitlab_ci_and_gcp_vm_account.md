---
title: "[實作筆記] Gitlab CI/CD 與 GCP - User Account"
date: 2023/04/24 20:01:33
tag:
  - CI/CD
---

## 前言

請參考[前篇](https://blog.marsen.me/2023/04/13/2023/gitlab_ci_and_gcp_vm/)，我們將建立兩台 VM，  
一台作為 CI/CD 用的 Gitlab Runner，另一台作為 Web Server，  
本篇將介紹 Gitlab Runner 的相關設定，  
架構如下
![GCP 與 Gitlab](/images/2023/gitlab-gcp.jpg)

參考我們的架構圖，

## Linux 系統的使用者管理

使用 Linux 作業系統管理使用者帳號如新增、刪除或修改使用者帳號資訊等等。
以下是幾個常用的命令。

### useradd

使用者帳號可以讓你進入系統，執行操作和管理文件等。  
這是一個基本的命令，通常需要以系統管理員的權限執行。  
範例如下：  

```shell
useradd john
```

這個命令將會新增一個名為 john 的使用者帳號。  

### userdel

刪除一個使用者帳號，你可以使用 userdel。  
刪除使用者帳號會刪除該使用者的所有文件，因此請確保你已經備份了重要的文件。  
範例如下：  

```shell
userdel john
```

這個命令將會刪除名為 john 的使用者帳號。

### usermod

修改使用者帳號的資訊，例如姓名或者家目錄，你可以使用 usermod 命令。  
範例如下：

```shell
usermod -c "John Smith" john
```

這個命令將會修改名為 john 的使用者帳號的姓名為 John Smith。

## 資料夾權限

可以用以下語法列出資料夾內的檔案與資料夾權限

```shell
ls -al
```

在 Linux 系統中，檔案的類型和權限使用特殊的符號來表示。
第一個字元表示檔案的類型，例如目錄、檔案或連結檔等等。  
`d` 是目錄，`-`是檔案，`l` 是 link file;  
`b` 是可儲存的周邊設備、`c` 是鍵盤、滑鼠等設備。

接下來的三個字元代表三種權限，分別是讀取(`r`)、寫入(`w`)和執行(`x`)。  
每三個一組，分別對應檔案的擁有者、群組帳號和其他帳號。如果沒有權限，則會用減號來表示。  
這些權限可以用來控制不同使用者對於檔案的存取權限。  

在 macOS 中最後一個字元，有可能是 `@`、`+`、`.` 或沒有字元  
`@`：代表此檔案有擴展屬性（Extended Attributes）。  
`+`：代表此檔案有 ACL（Access Control List）權限限制。  
`.`：代表此檔案有 SELinux 安全屬性（SELinux Security Attributes）。  
沒有字元：代表此檔案沒有以上特性  

舉例來說

```shell
drwx------ 87 mark.lin  rd      2784  4 12 13:21 Library
```

這是一個名為 Library 的資料夾，擁有者是使用者 mark.lin，群組是 rd。  
權限設定為只有擁有者有讀、寫、執行權限，而群組和其他使用者都沒有任何權限。  

## 建立 VM 的 Account

在GCP的實作上我們不需要直接建立帳戶與群組，記得架構圖中的 SSH Key 嗎?  
當我們在 Compute Engine Metadata 建立 SSH KEYS 時，就會依照這把 KEY 的　Comments　建立一個 Linux VM 帳號　　
因此我們的 VM 就會有對應的帳號資料

## 參考

- Linux 指令
  - [useradd](https://man7.org/linux/man-pages/man8/useradd.8.html)
  - [userdel](https://man7.org/linux/man-pages/man8/userdel.8.html)
  - [usermod](https://man7.org/linux/man-pages/man8/usermod.8.html)
  - [Linux 的檔案權限與目錄配置](https://linux.vbird.org/linux_basic/centos7/0210filepermission.php)
- [實作筆記] Gitlab CI/CD 與 GCP 相關文章
  - [架構全貌](https://blog.marsen.me/2023/04/13/2023/gitlab_ci_and_gcp_vm/)
  - [建立 Web Server VM](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_create_server/)
  - [建立 Gitlab Runner VM](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_cretae_runner/)
  - [Firewall](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_firewall/)

(fin)
