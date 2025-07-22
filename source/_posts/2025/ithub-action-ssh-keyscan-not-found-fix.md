---
title: "[踩雷筆記] GitHub Action ssh-keyscan not found 問題修復"
date: 2025/07/22 13:42:30
tags:
  - 踩雷筆記
---


## 前情提要

GitHub Action `marsen/hexo-action@v1.0.11` 之前運行正常，但最近開始失敗，錯誤訊息：

```text
/usr/app/entrypoint.sh: 9: /usr/app/entrypoint.sh: ssh-keyscan: not found
```

## 問題分析

### 初步檢查

檢查 entrypoint.sh 第9行：

```bash
ssh-keyscan -t rsa github.com >> /root/.ssh/known_hosts
```

檢查 Dockerfile 套件安裝：

```dockerfile
FROM node:20-buster-slim
RUN apt-get install -y git openssh-client
```

明明有安裝 `openssh-client`，為什麼找不到 `ssh-keyscan`？

### 根本原因

**Debian Buster 生命週期結束**

- `node:20-buster-slim` 基於 Debian 10 (Buster)
- Debian 10 於 2024年8月達到 End of Life
- 套件庫不再維護，套件結構可能變化

**OpenSSH 套件重組**

- 2024年 OpenSSH 多個版本發布 (9.7, 9.9)
- `ssh-keyscan` 可能從 `openssh-client` 移到其他套件

**Docker 映像自動更新**

- Docker Hub 會自動重建映像
- 新版本移除不必要工具以減少攻擊面

## 解決方案

### 快速修復（不推薦）

```dockerfile
RUN apt-get install -y git openssh-client openssh-server
```

雖然可行，但安裝 SSH 伺服器會增加攻擊面。

### 正確解法

升級基底映像到支援版本：

```dockerfile
# 從過期版本
FROM node:20-buster-slim

# 升級到安全版本  
FROM node:20-bookworm-slim
```

完整的 Dockerfile 修改：

```dockerfile
FROM node:20-bookworm-slim

LABEL version="1.0.12"
LABEL repository="https://github.com/marsen/hexo-action"
LABEL homepage="https://blog.marsen.me"
LABEL maintainer="marsen.lin <admin@marsen.me>"

WORKDIR /usr/app

COPY entrypoint.sh /usr/app/entrypoint.sh
COPY sync_deploy_history.js /usr/app/sync_deploy_history.js

# 安全且最佳化的套件安裝
RUN apt-get update > /dev/null && \
    apt-get install -y git openssh-client > /dev/null && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    chmod +x /usr/app/entrypoint.sh

ENTRYPOINT ["/usr/app/entrypoint.sh"]
```

## 關鍵改進

1. **基底映像升級**：buster-slim → bookworm-slim
   - Debian 12 取代已 EOL 的 Debian 10
   - 更好的安全性和長期支援

2. **最小權限原則**：只安裝 openssh-client
   - 包含所需的 ssh-keyscan 工具
   - 不安裝 SSH 伺服器，減少攻擊面

3. **Docker 最佳實踐**：
   - 清理 apt 快取減少映像大小
   - 使用 && 合併 RUN 指令減少層數

## 預防措施

- 使用具體版本標籤而非 latest
- 定期檢查基底映像的生命週期
- 建立自動化測試驗證依賴可用性

## 小結

這次問題的核心是基底映像過期導致的連鎖反應。在容器化開發中，外部依賴的變化往往會影響既有系統。解決方案不是快速修復，而是從根本上升級到安全的長期支援版本。

除錯關鍵思路：分析「為什麼之前可以，現在不行」，往往能找到外部環境變化的線索。

(fin)
