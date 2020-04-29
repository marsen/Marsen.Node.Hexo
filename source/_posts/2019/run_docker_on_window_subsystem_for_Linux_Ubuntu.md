---
title: "[實作筆記]在 Windows Subsystem for Linux 執行 docker"
date: 2019/01/07 17:18:13
tag:
    - Docker
    - Container
    - 實作筆記
---

## 前情提要

最近開始學 [Docker](https://www.docker.com/)， 這個神器大概在 C 社時期就有聽過了，  
不過一直沒有機會在實務上接觸，雖然有自已摸一點，但就只有皮毛而已，
在 A 社有機會由同事開課，並提供 EC2 作實驗，就趁這個機會作一點深入的學習。

一開始有一個很天真的想法，**我想在 Windows 內建的 Linux 子系統 Ubantu 安裝 Docker**，  
沒想到最後是脫褲子放屁，不過過程蠻有趣的，稍微記錄一下。

## 操作步驟

1. Windows 10 安裝 Ubantu
2. 在 Ubantu 安裝 Docker

    更新 apt-get

    ```bash
    sudo apt-get update
    ```

    允許 apt-get 透過 https

    ```bash
    sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        software-properties-common
    ```

    加入 Docker 官方 GPG KEY

    ```shell
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```

    然後可以驗証一下

    ```shell
    sudo apt-key fingerprint 0EBFCD88
    ```

    加入 Docker 的 apt-repository

    ```shell
    $ sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
    ```

    再次更新 apt-get

    ```shell
    sudo apt-get update
    ```

    安裝 Docker CE

    ```shell
    sudo apt-get install docker-ce
    ```

3. Run Docker

```shell
sudo docker container run hello-world
```

我都會失敗如下

```shell
docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?.
See 'docker run --help'.
```

## 如何解決

1. Windows 上要安裝 Docker for Windows
2. 勾選 Docker for Windows → Setting → Expose deamon on tcp://localhost:2375 Without TLS
![Docker](/images/2019/1/docker.jpg)

3. 回到 Ubantu , 執行以下命令

指定 Docker Host 在路徑

```bash
docker -H localhost:2375 images
```

如果不想每次指定的話…請參考以下命令

```bash
export DOCKER_HOST=localhost:2375
```

![執行結果](/images/2019/1/ubantu_docker.jpg)

## 學到的事

- Docker 有 Host(Deamon) 與 Client 之分
- 在 Windows 上的 VM(Ubantu) 再安裝 Docker Deamon 是行不通的(細節我並不清楚，求補充…)
- 我還不夠了解 Windows Container 與 Linux Container 的差異
  - [Windows Container FAQ - 官網沒有說的事](https://columns.chicken-house.net/2016/09/05/windows-container-faq/)
  - [LCOW Labs: Linux Container On Windows](https://columns.chicken-house.net/2017/10/04/lcow/)
  - more ...
- ~~繞了一圈，一事無成~~

## 參考

- [Installing the Docker client on Windows Subsystem for Linux (Ubuntu)](https://medium.com/@sebagomez/installing-the-docker-client-on-ubuntus-windows-subsystem-for-linux-612b392a44c4)
- [Get Docker CE for Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
- [取得 Ubuntu - Microsoft Store zh-TW](https://www.microsoft.com/zh-tw/p/ubuntu/9nblggh4msv6?activetab=pivot%3Aoverviewtab)
- [Docker CE Desktop Windows](https://hub.docker.com/editions/community/docker-ce-desktop-windows)
- [如何安裝 Docker for Windows ?](https://oomusou.io/docker/docker-for-windows/)
- [Windows 上的 Linux 容器](https://docs.microsoft.com/zh-tw/virtualization/windowscontainers/deploy-containers/linux-containers)

(fin)
