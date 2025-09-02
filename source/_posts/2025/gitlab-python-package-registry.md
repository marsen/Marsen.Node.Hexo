---
title: " [實作筆記] 建立私有 Python Package Registry - 以 ONNX Runtime 為例"
date: 2025/09/02 14:42:42
tags:
  - 實作筆記
---

## 前情提要

在開發 AI 應用時，我們遇到一些特殊的依賴管理需求，需要私有 Registry

在這個專案中，我們遇到了幾個挑戰：

1. **平台特化需求**：Jetson 平台需要特製的 `onnxruntime-gpu` 版本，PyPI 上沒有現成的
2. **版本一致性**：確保所有環境（開發、測試、生產）使用完全相同的依賴版本
3. **安全性考量**：避免依賴外部不穩定的來源，降低供應鏈攻擊風險
4. **內部套件分發**：團隊開發的內部工具需要有管道分發

## 架構設計

整體架構分為三個層次：

```text
┌───────────────────────┐
│  GitLab CI/CD         │ → 自動化構建與發布
├───────────────────────┤
│  Package Registry     │ → 私有套件儲存
├───────────────────────┤
│  Project Dependencies │ → 專案依賴管理
└───────────────────────┘
```

## 實作步驟

### 設定 GitLab Package Registry

首先，在 GitLab 專案中啟用 Package Registry，並建立 Deploy Token：

在 GitLab 專案設定中建立 Deploy Token, Settings → Repository → Deploy Tokens

權限：read_package_registry, write_package_registry

記下 token ID 和 token 值，格式如下：

- Token ID: deploy-token-{ID}

- Token: {TOKEN}

### 取得 Project ID

有三種方式可以找到 GitLab Project ID：

從專案首頁：

進入你的 GitLab 專案首頁
Project ID 會顯示在專案名稱下方
例如：Project ID: 12345678

從專案設定頁面：

進入 Settings → General
在最上方的 "General project settings" 區塊
可以看到 Project ID

從 GitLab API：

如果你在 CI/CD pipeline 中，可以直接使用環境變數 $CI_PROJECT_ID
這個變數會自動帶入當前專案的 ID

### 配置專案的依賴管理

在 `pyproject.toml` 中設定私有 registry：

```toml
# 定義私有 index
[[tool.uv.index]]
name = "onnx"
url = "https://gitlab+deploy-token-{TOKEN_ID}:{TOKEN}@gitlab.com/api/v4/projects/{PROJECT_ID}/packages/pypi/simple"
default = false

# 指定套件來源
[tool.uv.sources]
onnxruntime-gpu = { index = "onnx" }

# 多平台依賴策略
dependencies = [
    # macOS ARM64 使用 CPU 版本
    "onnxruntime==1.19.0; sys_platform == 'darwin' and platform_machine == 'arm64'",
    # Jetson 使用私有倉庫的 GPU 版本
    "onnxruntime-gpu; sys_platform == 'linux' and platform_machine == 'aarch64'",
]
```

這裡的關鍵是使用條件依賴，根據不同平台安裝不同版本的套件。

### 3. 開發者使用私有 Registry

根據專案的安全策略，RD 有幾種方式存取私有 registry：

#### 選項 1：Token 內嵌在 pyproject.toml（簡單但不安全）

如果 `pyproject.toml` 中已經包含完整的認證 URL：

```bash
# 直接安裝依賴
uv sync

# 或使用 pip
pip install -e .
```

#### 選項 2：使用環境變數（推薦）

在 `pyproject.toml` 中使用佔位符：

```toml
url = "https://deploy-token-{TOKEN_ID}:${GITLAB_TOKEN}@gitlab.com/api/v4/projects/{PROJECT_ID}/packages/pypi/simple"
```

RD 需要設定環境變數：

```bash
export GITLAB_TOKEN=your_deploy_token
uv sync
```

#### 選項 3：使用認證檔案

設定 pip 或 uv 的認證檔案：

```bash
# 建立 pip 配置
cat > ~/.pip/pip.conf <<EOF
[global]
extra-index-url = https://deploy-token-{TOKEN_ID}:{TOKEN}@gitlab.com/api/v4/projects/{PROJECT_ID}/packages/pypi/simple
EOF
```

#### 選項 4：企業內網存取

如果使用企業內網或 VPN：

```bash
# 連接 VPN 後直接使用
uv sync
```

### 4. CI/CD 自動化發布

設定 GitLab CI 自動構建並推送套件到 registry：

```yaml
# .gitlab-ci.yml
variables:
  REGISTRY_PATH: registry.gitlab.com/$GROUP_PATH/$PROJECT_NAME

stages:
  - build
  - publish

build-package:
  stage: build
  script:
    # 構建 Python 套件
    - pip install build
    - python -m build
  artifacts:
    paths:
      - dist/

publish-to-registry:
  stage: publish
  dependencies:
    - build-package
  script:
    # 設定認證
    - pip install twine
    - |
      cat > ~/.pypirc <<EOF
      [distutils]
      index-servers = gitlab
      
      [gitlab]
      repository = https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/packages/pypi
      username = gitlab-ci-token
      password = ${CI_JOB_TOKEN}
      EOF
    
    # 上傳到 GitLab Package Registry
    - python -m twine upload --repository gitlab dist/*
  only:
    - main
```

### 6. Docker 映像支援多架構

為了支援不同的硬體平台，使用 buildx 構建多架構映像：

```yaml
build-multi-arch:
  stage: build
  script:
    # 設定 buildx
    - docker buildx create --use
    
    # 構建並推送多架構映像
    - |
      docker buildx build \
        --platform linux/amd64,linux/arm64 \
        -t $REGISTRY_PATH:$NEW_TAG \
        --push .
```

## 常見問題與解決方案

### Deploy Token 權限不足？

確保 Deploy Token 有 `read_package_registry` 和 `write_package_registry` 權限。

### 套件版本衝突？

使用 `uv` 的 resolution markers 功能，明確指定版本解析策略。

### 多架構構建失敗？

檢查 Docker buildx 是否正確安裝，並確認基礎映像支援目標架構。

### Registry URL 格式錯誤？

GitLab PyPI registry URL 格式為：

```
https://gitlab.com/api/v4/projects/{PROJECT_ID}/packages/pypi/simple
```

注意將 `{PROJECT_ID}` 替換為實際的專案 ID。

## 小結

透過建立私有 Package Registry，我們成功解決了：

- **特製版本管理**：Jetson 平台的特殊 ONNX Runtime 版本需求
- **依賴一致性**：所有環境使用相同版本的套件
- **安全性提升**：減少對外部來源的依賴
- **自動化流程**：CI/CD 自動構建並發布套件

這個架構不僅適用於 ONNX Runtime，也可以擴展到其他需要特殊管理的依賴套件。

(fin)
