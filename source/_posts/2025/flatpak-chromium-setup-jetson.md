---
title: " [實作筆記] Jetson Orin 瀏覽器安裝"
date: 2025/07/22 12:04:31
---

## 前情提要

在 Jetson Orin 上安裝瀏覽器時，直接使用 `apt install chromium` 或 `apt install firefox` 會遇到問題。這些指令實際上會安裝 Snap 版本，但 Jetson kernel 缺少 AppArmor 或 SquashFS_xattr 支援，導致無法啟動。

## 解決方案

### Flatpak Chromium（推薦）

最穩定的方案是透過 Flatpak 安裝 Chromium：

```bash
# 安裝 Flatpak
sudo apt update && sudo apt install flatpak -y

# 新增 Flathub
sudo flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

# 安裝 Chromium
flatpak install flathub org.chromium.Chromium -y

# 執行
flatpak run org.chromium.Chromium
```

優點：
- 最接近原生 Chrome 體驗
- 完整支援硬體加速和 WebGL
- 穩定性佳

### Flatpak Firefox

```bash
# 安裝 Firefox
flatpak install flathub org.mozilla.firefox -y

# 執行
flatpak run org.mozilla.firefox
```

### 輕量化選擇：Midori

如果只需要基本瀏覽功能：

```bash
sudo apt install midori -y
```

## 問題分析

**為什麼 apt 版本無法使用？**

1. `apt install chromium` 實際安裝的是 Snap 套件
2. Jetson kernel 缺少必要的 AppArmor、SquashFS_xattr 模組
3. 即便能啟動，WebGL 和硬體加速支援也很差

**檢查硬體加速**

- Chromium: 輸入 `chrome://gpu/`
- Firefox: 輸入 `about:support`

## 小結

建議使用 Flatpak Chromium 作為主要瀏覽器，避免使用預設的 Snap 版本。如果追求輕量化且只需基本功能，Midori 是不錯的選擇。

(fin)