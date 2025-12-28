---
title: " [實作筆記] EAI 機器瀏覽器安裝問題與解決方案"
date: 2025/07/29 13:51:36
---

## 前情提要

最近拿到一個地端資源 [EAI-I233](https://www.lannerinc.com/products/edge-ai-appliance/deep-learning-inference-appliances/eai-i233) 開發地端 AI 應用。  
過程中遇到了安裝瀏覽器(firefox)無法正常運作的問題。  
經過查找資料，發現這是 Jetson Orin Nano 上的已知問題，不是個案。
記錄一下如何透過 Flatpak 安裝 Chromium 在 EAI 機器上，以解決瀏覽器問題。

## 問題描述

在 Jetson Orin Nano 等 EAI 設備上，透過 snap 安裝的 Chromium 瀏覽器會出現無法正常啟動或運行不穩定的狀況。  
這個問題並非個案，在 NVIDIA 開發者論壇上有許多用戶回報類似問題。  

## 解決方案

### 移除現有的 snap 版本 Chromium

首先移除透過 snap 安裝的 Chromium：

```bash
sudo snap remove chromium
```

### 安裝與設定 Flatpak

更新套件管理器並安裝 Flatpak：

```bash
sudo apt update
sudo apt install flatpak
```

添加 Flathub 儲存庫：

```bash
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

### 重開機並安裝 Chromium

重新啟動機器後，透過 Flatpak 安裝 Chromium：

```bash
# 重開機後執行
flatpak install flathub org.chromium.Chromium
```

## 為什麼要用 Flatpak？

相比於 snap，Flatpak 在 ARM 架構的設備上有更好的相容性。  
特別是在 Jetson 系列開發板上，Flatpak 能夠提供更穩定的套件執行環境，避免因為 snap 沙盒機制與硬體驅動互動時產生的問題。  

## 一些要注意的小問題

**重開機是必要的**  
在安裝完 Flatpak 後，務必重開機再安裝 Chromium，確保所有相依服務正確啟動。

**權限問題**  
如果遇到權限問題，確認當前使用者已加入相關群組：

```bash
sudo usermod -a -G sudo $USER
```

## 參考資料

- [Jetson Orin Nano Browser Issue - NVIDIA Developer Forum](https://forums.developer.nvidia.com/t/jetson-orin-nano-browser-issue/338580/44)
- [JetsonHacks: Why Chromium Suddenly Broke on Jetson Orin](https://jetsonhacks.com/2025/07/12/why-chromium-suddenly-broke-on-jetson-orin-and-how-to-bring-it-back/)

## 小結

透過 Flatpak 安裝 Chromium 是目前在 EAI 設備上最穩定的解決方案。  
雖然需要額外的設定步驟，但能夠有效解決 snap 版本在 ARM 架構上的相容性問題。  
這個解決方案不僅適用於 Jetson Orin Nano，也可以應用到其他類似的 ARM 架構開發板上。  
有機會拿到別的機器再來試試。

(fin)
