---
title: " [實作筆記] 在 MacBook 上安裝與使用 Ollama"
date: 2025/02/14 17:43:21
tags:
  - 實作筆記
---


## 前情提要

AI 時代到來，就像 Web、Mobile 等帶來一波波的浪潮與機會，趁機來學習點新技術了。
比如說像是 **Ollama** 這款簡單好用的工具，可以快速在本機啟用大型語言模型 (LLM)，如 LLaMA、Mistral 等。以下記錄在 macOS 上的安裝與初體驗。

## 實作記錄

### 1. 安裝 Ollama

- **使用 Homebrew 安裝**：

   ```bash
   brew install ollama
   ```

- **確認安裝成功**：

   ```bash
   ollama --version
   ```

### 2. 執行第一個模型

- 拉取並執行 LLaMA 2 模型：

   ```bash
   ollama run llama2
   ```

- 若要使用其他模型（如 Mistral）：

   ```bash
   ollama run mistral
   ```

### 3. 互動範例

- 問答互動示例：

  ```bash
  $ ollama run llama2
  > 你好！請問我可以幫你什麼？
  ```

### 4. 常見的 ollama 指令

- **列出可用模型**：

   ```bash
   ollama list
   ```

- **查看模型詳情**：

   ```bash
   ollama info <model_name>
   ```

- **停止運行中的模型**：

   ```bash
   ollama stop <model_name>
   ```

- **更新 Ollama**：

   ```bash
   brew upgrade ollama
   ```

## 簡單的 web 介面

使用 [Chrome 外掛](https://chromewebstore.google.com/detail/page-assist-a-web-ui-for/jfgfiigpkhlkbnfnbobbkinehhfdhndo)
安裝後選擇模型就可以透過 Web UI 與模型互動。
也有一些開源專案有提供 Web UI

- <https://github.com/open-webui/open-webui>
- <https://github.com/Bin-Huang/chatbox>

我沒有實作作，僅記錄一下查找到的結果

## 參考

- [Ollama 官方文件](https://ollama.ai/)
- [Ollama Github](https://github.com/ollama/ollama)
- [Homebrew 官方網站](https://brew.sh)
- [DeepSeek-R1最佳本地用法！免費開源，無痛運行高級 AI 大模型，秒建私人知識庫！ | 零度](https://www.youtube.com/watch?v=tWJvSy7dL1w)
- [DeepSeek-r1:7b](https://ollama.com/library/deepseek-r1:7b)
- [DeepSeek](https://www.deepseek.com/)
- [DeepSeek Chat](https://chat.deepseek.com/)

(fin)
