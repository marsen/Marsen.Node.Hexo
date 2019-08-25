---
title: "[學習筆記]AWS EC2 學習筆記 AWS CLI 與 Login"
date: 2018/03/25 23:01:24
tag:
  - AWS
  - Docker
  - Container 
---

## [安裝 AWS CLI](https://docs.aws.amazon.com/zh_cn/cli/latest/userguide/installing.html)

## [配置 AWS CLI](https://docs.aws.amazon.com/zh_cn/cli/latest/userguide/cli-chap-getting-started.html#cli-quick-configuration)

## EC2 開機

![EC2 開機](https://i.imgur.com/dNGygaT.jpg)

- 直接開機跳過網路設定(也還沒有辦法設)
- 第5步驟設定 TAG ,對找尋 ec2 的 instance 很有幫助

## 設定 [Putty](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/putty.html?icmpid=docs_ec2_console) 

## 連線機器

![連線機器](https://i.imgur.com/xIQsEac.jpg)

ex:  

```bash
ssh -i /path/my-key-pair.pem ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com
```

### 預設連線帳戶

> For Amazon Linux, the user name is `ec2-user`.
> For Centos, the user name is `centos`. 
> For Debian, the user name is `admin` or `root`. 
> For Fedora, the user name is `ec2-user`. 
> For RHEL, the user name is `ec2-user` or `root`. 
> For SUSE, the user name is `ec2-user` or `root`. 
> For Ubuntu, the user name is `ubuntu` or `root`. 
> Otherwise, if `ec2-user` and `root` don't work, check with your AMI provider.

windows 好像是 Administrator ? 求補充

## Docker

### 安裝 Docker

```bash
sudo yum install docker
```

### 啟動 Docker 服務，並讓它隨系統啟動自動載入

```bash
sudo service docker start
sudo chkconfig docker on
```

### 雷包

- 重啟機器的話 public dns 會改變.(意味連線的命令參數會變)
- 注意使用的AIM, 不同的 Linux OS 會有不同的套件執行命令
- ubuntu `apt-get`
- CentOS `yum`

### 參考

- [Docker —— 從入門到實踐](https://philipzheng.gitbooks.io/docker_practice)
- [全面易懂的Docker指令大全](https://www.gitbook.com/book/joshhu/dockercommands/details)

(fin)
