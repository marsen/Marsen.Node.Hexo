---
title: "[實作筆記] 初體驗設定 Nvidia GPU 的 Azure VM"
date: 2024/07/03 15:59:02
---

## 前情提要

隨著深度學習和 AI 的普及，許多工作和研究需要強大的運算能力，  
而 GPU 提供了相較於傳統 CPU 更高效的計算能力。  
因此，我選擇在 Azure 上設定 Nvidia GPU 虛擬機來滿足這些需求。  
這篇文章將分享我在 Azure 上設定 Nvidia GPU 虛擬機的初體驗，並記錄實作過程中的一些重點。

## 前置作業

在開始設定 GPU 虛擬機之前，需要先完成以下準備工作：

1. **Azure 帳戶**：確保你已經擁有 Azure 的帳戶，並且帳戶中有足夠的配額來創建 GPU 虛擬機。
2. **選擇適合的虛擬機規格**：Azure 提供多種 GPU 虛擬機型號，如 NV 系列和 NC 系列，根據需求選擇合適的型號。
3. **安裝 Azure CLI**：透過 Azure CLI 可以更方便地管理和配置虛擬機，可以在本地環境中安裝並配置 Azure CLI。
4. **創建資源群組與虛擬機**：
  
  ```bash
   az group create --name <myResourceGroup> --location eastus
  ```

  ```bash
  az vm create \
    --resource-group myResourceGroup \
    --name myT4VM \
    --image UbuntuLTS \
    --size Standard_NC6s_v3 \
    --admin-username azureuser \
    --generate-ssh-keys
  ```

## 實作步驟

可以參考[官方手冊](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#local-repo-installation-for-ubuntu))，  
本文大量引用 3. Package Manager Installation 中 3.9 的篇幅

### 登入到虛擬機後，安裝 Nvidia 驅動程式

  ```bash
  sudo apt-get update
  sudo apt-get install -y nvidia-driver-470
  sudo reboot
  ```

### 安裝當前運行的內核版本所需的 Linux 標頭文件

```bash
sudo apt-get install linux-headers-$(uname -r)
```

### 刪除過時的金鑰(實作上，跟本沒有這個金鑰，所以跳過也沒關係)

```bash
sudo apt-key del 7fa2af80
```

### 查詢作業系統與晶片架構

```bash
> lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 22.04.4 LTS
Release:	22.04
Codename:	jammy
> uname -m
x86_64
```

### 安裝 CUDA-Keyring

```sh
wget https://developer.download.nvidia.com/compute/cuda/repos/$distro/$arch/cuda-keyring_1.1-1_all.deb
```

參考前一步驟將 `$distro` 換成 `ubuntu2204`，`$arch` 換成 `x86-64`,  
也可以直接到此 <https://developer.download.nvidia.com/compute/cuda/repos> 找到你合適的 deb 檔
下載:  

```sh
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86-64/cuda-keyring_1.1-1_all.deb
```

安裝:

```sh
sudo dpkg -i cuda-keyring_1.1-1_all.deb
```

### 安裝 CUDA SDK

```sh
sudo apt-get install cuda-toolkit
```

### 安裝 Nvidia GDS 驅動，提升 GPU 和存儲間的高效數據傳輸性能

```sh
sudo apt-get install nvidia-gds
```

### 重啟主機

```sh
sudo reboot
```

### 確認是否安裝成功

  ```bash
  nvidia-smi
  ```

　看到下面的畫面就是成功
  
  ```sh
  > nvidia-smi
  Mon Jul  1 09:21:24 2024
  +---------------------------------------------------------------------------------------+
  | NVIDIA-SMI 535.183.01             Driver Version: 535.183.01   CUDA Version: 12.2     |
  |-----------------------------------------+----------------------+----------------------+
  | GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
  | Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
  |                                         |                      |               MIG M. |
  |=========================================+======================+======================|
  |   0  Tesla T4                       Off | 00000001:00:00.0 Off |                  Off |
  | N/A   31C    P8               9W /  70W |    140MiB / 16384MiB |      0%      Default |
  |                                         |                      |                  N/A |
  +-----------------------------------------+----------------------+----------------------+

  +---------------------------------------------------------------------------------------+
  | Processes:                                                                            |
  |  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
  |        ID   ID                                                             Usage      |
  |=======================================================================================|
  |    0   N/A  N/A      1095      G   /usr/lib/xorg/Xorg                          130MiB |
  |    0   N/A  N/A      1356      G   /usr/bin/gnome-shell                          7MiB |
  +---------------------------------------------------------------------------------------+
  ```

## 20240730 更新

### 原由

RD 們反應因為 Driver 更新導致服務異常，停止運作。  
重新安裝時又遇到一些奇怪的問題，例如：  
錯誤：

```shell
sudo apt install -y linux-modules-nvidia-550-azure nvidia-driver-550
Reading package lists... Done
Building dependency tree       
Reading state information... Done
E: Unable to locate package linux-modules-nvidia-550-azure
```

或是

```shell
Failed to initialize NVML: Driver/library version mismatch
NVML library version: 535.183
```

依實際狀況與 RD 討論，發現他們沒有一個標準的作業程序，調研時也沒有任何記錄，  
東作一塊，西作一塊，沒有辦法很清楚了交互的作用關係。  
故由我重新實作一遍，記錄並排除相關錯誤。  

- Azure 開機，指定作業系統為 Ubuntu 22.04，並且需注意 A100 系列機器只在特定的 Zone 才有資源開機
- 安裝 CUDA,　[參考官方文件 **3.9. Ubuntu**](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#ubuntu)
- 安裝 ubuntu-drivers-common
  - `sudo apt install ubuntu-drivers-common`
  - `sudo apt install alsa-utils` #音效相關，不一定要裝，但是可以減少指令時的錯誤訊息
  - `ubuntu-drivers device`
  
  ```bash
    $ ubuntu-drivers devices
    == /sys/devices/LNXSYSTM:00/LNXSYBUS:00/ACPI0004:00/VMBUS:00/00000041-0001-0000-3130-444532304235/pci0001:00/0001:00:00.0 ==
    modalias : pci:v000010DEd000020B5sv000010DEsd00001533bc03sc02i00
    vendor   : NVIDIA Corporation
    driver   : nvidia-driver-555-open - third-party non-free
    driver   : nvidia-driver-470-server - distro non-free
    driver   : nvidia-driver-535-server-open - distro non-free
    driver   : nvidia-driver-555 - third-party non-free recommended
    driver   : nvidia-driver-535-server - distro non-free
    driver   : nvidia-driver-470 - distro non-free
    driver   : nvidia-driver-550 - third-party non-free
    driver   : nvidia-driver-550-open - third-party non-free
    driver   : nvidia-driver-535-open - distro non-free
    driver   : nvidia-driver-545-open - third-party non-free
    driver   : xserver-xorg-video-nouveau - distro free builtin
  ```

- 更新套件資訊
  - sudo add-apt-repository ppa:graphics-drivers/ppa
  - sudo apt update
- 安裝趨動, 讓系統自已判斷安裝什麼版本, 參考 [NVIDIA drivers installation | Ubuntu](https://ubuntu.com/server/docs/nvidia-drivers-installation)
  > `sudo ubuntu-drivers install`
- 重開機
  >`sudo reboot`
- 查一下，他幫你裝哪個版本
  > `dpkg -l | grep nvidia-driver`
- 再安裝指定版本的 Ubuntu Azure Package，也可以至 [Ubuntu 網站](https://packages.ubuntu.com/focal/kernel/) 搜尋確認
  > `sudo apt install -y linux-modules-nvidia-<version>-azure`

## 參考

- [Sizes for virtual machines in Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes/overview)
- [Install NVIDIA GPU drivers on N-series VMs running Linux](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/n-series-driver-setup)
- [NVIDIA CUDA Installation Guide for Linux](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#prepare-ubuntu)
- [Sizes for virtual machines in Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes/overview?tabs=breakdownseries%2Cgeneralsizelist%2Ccomputesizelist%2Cmemorysizelist%2Cstoragesizelist%2Cgpu-nc-fam%2Cfpgasizelist%2Chpcsizelist#gpu-accelerated)

(fin)
