---
title: "[實作筆記] 初體驗設定 Nvidia GPU 的 Azure VM -- 錯誤排除"
date: 2024/07/30 17:10:59
---


## 前情提要

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

## 實作記錄

1. Azure 開機，指定作業系統為 Ubuntu 22.04，並且需注意 A100 系列機器只在特定的 Zone 才有資源開機，本次使用東日本 zone2 的資源
2. 安裝 CUDA,　[參考官方文件 **3.9. Ubuntu**](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#ubuntu)
3. 安裝 ubuntu-drivers-common
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

4. 更新套件
   1. 加上drivers的 repo 路徑
      > `sudo add-apt-repository ppa:graphics-drivers/ppa`
   2. 更新
      > `sudo apt update`
   3. 安裝趨動, 讓系統自已判斷安裝什麼版本, 參考 [NVIDIA drivers installation | Ubuntu](https://ubuntu.com/server/docs/nvidia-drivers-installation)
      > `sudo ubuntu-drivers install`
   4. 重開機
      >`sudo reboot`
   5. 查一下，他幫你裝哪個版本
      > `dpkg -l | grep nvidia-driver`
   6. 再安裝指定版本的 Ubuntu Azure Package，也可以至 [Ubuntu 網站](https://packages.ubuntu.com/focal/kernel/) 搜尋確認
      > `sudo apt install -y linux-modules-nvidia-<version>-azure`

### 確認是否成功

  ```bash
  nvidia-smi
  ```

　看到下面的畫面就是成功
  
  ```sh
$ nvidia-smi
Tue Jul 30 08:07:18 2024
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.90.07              Driver Version: 550.90.07      CUDA Version: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA A100 80GB PCIe          Off |   00000001:00:00.0 Off |                    0 |
| N/A   32C    P0             43W /  300W |       1MiB /  81920MiB |      0%      Default |
|                                         |                        |             Disabled |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
  ```

## 參考

- [Ubuntu 網站](https://packages.ubuntu.com/focal/kernel/)
- [NVIDIA drivers installation | Ubuntu](https://ubuntu.com/server/docs/nvidia-drivers-installation)
- [參考官方文件 **3.9. Ubuntu**](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#ubuntu)

(fin)
