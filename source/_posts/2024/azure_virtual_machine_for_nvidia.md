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

追加[異常記錄](https://blog.marsen.me/2024/07/30/2024/azure_vm_for_nvidia_error_records/)

## 參考

- [Sizes for virtual machines in Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes/overview)
- [Install NVIDIA GPU drivers on N-series VMs running Linux](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/n-series-driver-setup)
- [NVIDIA CUDA Installation Guide for Linux](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#prepare-ubuntu)
- [Sizes for virtual machines in Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes/overview?tabs=breakdownseries%2Cgeneralsizelist%2Ccomputesizelist%2Cmemorysizelist%2Cstoragesizelist%2Cgpu-nc-fam%2Cfpgasizelist%2Chpcsizelist#gpu-accelerated)

(fin)
