---
title: " [實作筆記] Ollama 容器化調研筆記"
date: 2025/06/22 23:36:35
---

## 前情提要

最近在搞 AI 模型的本地部署，並試著將 Ollama 容器化。
經過一段時間的調研和實測，想記錄一下目前的進度和對幾個主流方案的看法。

### 目標

- 找到穩定的 Ollama 容器化方案
- 評估各種新興工具的可用性
- 為未來的產品開發做技術預研

### Ollama 簡介

Ollama 是一個開源工具，讓你在本地電腦上輕鬆運行大型語言模型（如 Llama、CodeLlama、Mistral 等）。
安裝簡單，一行指令就能下載和運行各種 AI 模型，無需複雜設定。支援 macOS、Linux 和 Windows。

```bash
#  安裝後直接用
ollama run llama3.2
```

類似 Docker 的概念，但專門為 AI 模型設計。
還提供 REST API：

```bash
# 啟動後自動開啟 API server (預設 port 11434)
curl http://localhost:11434/api/generate \
  -d '{"model": "llama3.2", "prompt": "Hello"}'
```

沒有 Ollama 的話，你需要：

- 手動下載模型檔案（通常好幾 GB）
- 處理不同模型格式（GGUF、GGML 等）
- 設定 GPU 加速環境
- 寫代碼載入模型到記憶體
- 處理 tokenization 和 inference
- 自己包 API server

Ollama 把這些都幫你搞定了。

## 實作

### 目前採用：Ollama Container + 指令初始化

經過評估，暫時保留傳統的 ollama container 方案，透過指令的 init 方法來取模型。

**為什麼選這個？**
- 穩定，bug 少
- 好 debug，每個步驟都可控
- 彈性高，要調整什麼都很容易

**痛點**
- 需要手動操作（未來會優化成全自動）

基本用法：
```bash
# 啟動 ollama container
docker run -d --name ollama -p 11434:11434 ollama/ollama

# 下載模型
docker exec -it ollama ollama pull llama3.2
```

### Docker Model Runner：值得關注但暫不採用

DMR 在 2025 年 4月推出 Beta 版，看起來很有潛力，但目前不打算用在產品上。

**問題點**
1. **平台差異**：Linux 上的啟用方法跟 Mac 不一樣，需要額外適配
2. **實驗階段**：API 和功能還在快速變化，對產品開發風險太高
3. **跨平台支援**：雖然最新的 Docker Desktop 4.41 已經支援 Windows，但整體成熟度還需觀察

會持續觀察，如果穩定性和跨平台支援改善，未來可能會是主流選擇。

### Podman AI Lab：理想與現實的差距

概念上跟 DMR 類似，但有個致命問題：**模型清單限制**。

**問題**
- 只能用官方清單中的模型
- 清單不夠齊全（連 Gemma3 都沒有）
- 比 DMR 的模型支援少很多

雖然最近跟 RamaLama 整合，但模型支援的局限性讓我直接 pass。

## 一些要注意的小問題

### Ollama Container 常見坑

1. **GPU 支援**：記得加 `--gpus all` 或用 docker-compose 設定
2. **模型路徑**：預設在 `/root/.ollama`，要持久化記得 mount volume
3. **網路設定**：如果在 k8s 環境，注意 service 的 port 設定

### Docker Model Runner 評估心得

- 目前主要在 macOS Apple Silicon 上比較穩定
- Linux 支援還在完善中
- Windows 支援是最近才加的，需要更多實測

## 技術選型思考

這次調研讓我想到幾個點：

1. **穩定性 > 新功能**：對產品開發來說，穩定性永遠是第一考量
2. **生態完整性**：單一工具再好，生態不完善就是硬傷
3. **維護成本**：新技術通常需要更多維護工作

## 下一步

短期計畫：
1. 把 Ollama container 的初始化流程自動化
2. 持續追蹤 DMR 發展
3. 建立技術方案評估標準

中期目標：
- 等 DMR 穩定後再評估導入
- 研究其他容器化方案
- 整理最佳實踐文件

## 小結

AI 基礎設施變化很快，容器化技術也在演進。目前沒有完美方案，但這個調研過程很有價值。

技術選擇是動態平衡的過程，要在穩定性、功能性、維護成本間找平衡點。

會持續關注這領域的發展，有新發現再分享。

(fin)