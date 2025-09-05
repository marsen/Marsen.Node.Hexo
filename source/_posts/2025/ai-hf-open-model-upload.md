---
title: " [實作筆記] PaddleOCR ONNX 模型發布到 Hugging Face Hub"
date: 2025/09/05 17:43:51
tags:
  - 實作筆記
---

## 前情提要

專案需要使用 PaddleOCR 進行文字辨識，但原始模型檔案需要從 PaddlePaddle 框架轉換成 ONNX 格式才能在不同環境中使用。

轉換完成後，面臨一個選擇：這些模型檔案要怎麼發布？放在專案內部？還是公開分享？

經過一番討論，決定將轉換後的 ONNX 模型公開發布到 Hugging Face Hub，一方面解決內部使用需求，另一方面也讓社群受益。

這篇記錄整個發布流程的實作過程與踩雷經驗。

## 目標

- 將 5 個 PaddleOCR ONNX 模型檔案 (208MB) 發布到 Hugging Face
- 建立完整的使用文檔與授權聲明  
- 提供簡單的下載方式給其他開發者
- 確保版權合規 (原始模型為 Apache 2.0 授權)

## 模型檔案清單

轉換完成的檔案包含：

- `PP-OCRv5_server_det_infer.onnx` (84MB) - 文字檢測
- `PP-OCRv5_server_rec_infer.onnx` (81MB) - 文字識別  
- `UVDoc_infer.onnx` (30MB) - 文檔矯正
- `PP-LCNet_x1_0_doc_ori_infer.onnx` (6.5MB) - 文檔方向
- `PP-LCNet_x1_0_textline_ori_infer.onnx` (6.5MB) - 文字方向
- `PP-OCRv5_server_rec_infer.yml` (145KB) - 配置檔案

## 平台選擇：為什麼是 Hugging Face？

本來考慮幾個選項：

- **GitLab**：現有平台，整合容易
- **GitHub**：開發者友好，但大檔案處理麻煩  
- **Hugging Face**：AI 模型專業平台

最終選擇 Hugging Face 的原因：

- ✅ 完全免費，50GB 額度綽綽有餘
- ✅ 原生支援 ONNX 格式
- ✅ 內建 Git LFS，大檔案處理無痛
- ✅ 全球 CDN，下載速度快
- ✅ 社群友好，可能吸引貢獻者

## 實作過程

### Phase 1: 準備階段

首先建立 Hugging Face 帳號並設定環境：

```bash
# 安裝 HF CLI
uv add huggingface_hub

# 設定 Token (需要 Write 權限)
export HF_TOKEN="your_token_here"
```

建立模型 Repository：

```python
from huggingface_hub import create_repo, whoami

# 驗證登入
user = whoami()
print(f"登入用戶: {user['name']}")

# 建立 repository  
repo_url = create_repo(
    repo_id="paddleocr-test",
    repo_type="model",
    private=False
)
print(f"Repository 建立成功: {repo_url}")
```

### Phase 2: 上傳模型檔案

```bash
  export HF_TOKEN=your_token
  hf upload marsena/paddleocr-test ./model_cache/paddleocr_onnx/ --repo-type=model
```

實際上傳時間約 3 分鐘，比預期快很多。

### Phase 3: 撰寫文檔

Hugging Face 的 README.md 比一般 GitHub 專案更重要，因為它就是模型的門面。

撰寫重點：

1. **清晰的模型說明**：每個檔案的用途
2. **具體的使用範例**：Python 程式碼示範
3. **完整的授權聲明**：避免版權爭議
4. **技術細節**：系統需求、相容性

```markdown
# PaddleOCR ONNX Models

本倉庫提供 PaddleOCR v5 的 ONNX 格式模型檔案...

## 快速開始

```python
from huggingface_hub import hf_hub_download

# 下載檢測模型
det_model = hf_hub_download(
    repo_id="marsena/paddleocr-test",
    filename="PP-OCRv5_server_det_infer.onnx"
)
```

需要注意 YAML metadata 的格式問題，HF 會檢查語法：

```yaml
---
tags:
- onnx
- ocr  
- paddleocr
- computer-vision
license: apache-2.0
---
```

### Phase 4: 測試驗證

上傳完成後務必測試下載功能：

```bash
# 測試單檔下載
hf download marsena/paddleocr-test PP-OCRv5_server_det_infer.onnx

# 測試整包下載
hf download marsena/paddleocr-test --local-dir ./test_download \
  --exclude "README.md" --exclude ".gitattributes"
```

## 一些要注意的小問題

### HF Token 權限設定

- **Read**：只能下載
- **Write**：可以上傳模型  
- **Admin**：可以刪除 repo

建立 token 時記得選擇 **Write** 權限。

### Git LFS 檔案大小限制  

Hugging Face 對單檔大小有限制：

- 一般檔案：< 10MB
- LFS 檔案：< 50GB

我們的最大檔案 84MB，自動走 LFS 沒問題。

### 背景上傳的重要性

大檔案上傳可能需要 10-30 分鐘，SSH 連線容易斷開。務必使用 `nohup` 或 `screen`。

### README 的 YAML 語法檢查

Hugging Face 會檢查 YAML metadata 語法，格式錯誤會有警告（但不影響功能）。

## 使用方式

發布完成後，其他開發者可以這樣使用：

```bash
# 下載所有模型到專案目錄
hf download marsena/paddleocr-test \
  --local-dir ./model_cache/paddleocr_onnx \
  --exclude "README.md" \
  --exclude ".gitattributes"
```

或在 Python 中：

```python
from huggingface_hub import hf_hub_download
import os

model_files = [
    "PP-OCRv5_server_det_infer.onnx",
    "PP-OCRv5_server_rec_infer.onnx", 
    "UVDoc_infer.onnx",
    "PP-LCNet_x1_0_doc_ori_infer.onnx",
    "PP-LCNet_x1_0_textline_ori_infer.onnx",
    "PP-OCRv5_server_rec_infer.yml"
]

cache_dir = "model_cache/paddleocr_onnx"
os.makedirs(cache_dir, exist_ok=True)

for file in model_files:
    hf_hub_download(
        repo_id="marsena/paddleocr-test",
        filename=file,
        local_dir=cache_dir
    )
```

## 小結

整個發布流程花費約 5-8 小時：

- **準備和上傳**：2 小時
- **文檔撰寫**：3 小時  
- **系統整合**：2 小時
- **測試驗證**：1 小時

Hugging Face Hub 確實是發布 AI 模型的好選擇，特別是：

1. **無痛 LFS**：大檔案處理完全自動化
2. **全球加速**：下載速度比自建服務快
3. **社群生態**：容易被發現和使用
4. **版本控制**：Git-based，開發者熟悉

如果你也有 ONNX 模型需要分享，不妨考慮 Hugging Face Hub。畢竟專業的事交給專業的平台來做，我們專注在模型本身就好。

## 參考連結

- [發布的模型](https://huggingface.co/marsena/paddleocr-test)
- [Hugging Face 文檔](https://huggingface.co/docs)
- [PaddleOCR 原始專案](https://github.com/PaddlePaddle/PaddleOCR)

(fin)
