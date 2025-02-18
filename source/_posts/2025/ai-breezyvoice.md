---
title: " [實作筆記] 試用 Breezy Voice"
date: 2025/02/17 17:10:43
tags:
  - 實作筆記
---

## 前情提要

聯發科開源 MR Breeze 2 多模態模型，包含 Llama-Breeze2 繁中語言模型、BreezyVoice 台灣口音語音合成模型，以及 Android APP。Llama-Breeze2 提升了繁體中文知識和視覺能力，BreezyVoice 則能生成擬真人聲

覺得有趣，來試玩看看

## 網頁操作

<https://huggingface.co/spaces/Splend1dchan/BreezyVoice-Playground>

比較簡單的作法可以 Duplicate Huggingface 的 UI 操作介面，  
上傳一個 5 分鐘的聲音檔案，就可以生成結果，等待時間 1~10 分鐘不等。

### 也可以使用 [Kaggle](https://www.kaggle.com/code/marsenlin/breezyvoice-playground/edit)

但是遇到一些問題

1. 沒找到可用 GPU ,可能是帳號權限的關係
   - 進行帳號驗証
   - 可以在右側[Session Option]區塊調整 Accelerator
   - 調整後可以選用 GPU 但是有 30 小時的時間限制

2. 無法 Clone Github Repo

    ```terminal
    Cloning into 'BreezyVoice'...
    fatal: unable to access 'https://github.com/mtkresearch/BreezyVoice.git/': Could not resolve host: github.com
    ```

   - 因為 Kaggle 無法連網(x)
   - 可以在右側[Session Option]區塊調整 Internet on

File > Open in Colab 可以

### 改用 [Google Colab]  

執行到 `!pip install -r requirements.txt` 會有以下錯誤  

```terminal
Collecting ttsfrd-dependency==0.1 (from -r requirements.txt (line 10))
  Downloading https://www.modelscope.cn/models/speech_tts/speech_kantts_ttsfrd/resolve/master/ttsfrd_dependency-0.1-py3-none-any.whl (1.1 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.1/1.1 MB 1.4 MB/s eta 0:00:00
ERROR: ttsfrd-0.3.9-cp310-cp310-linux_x86_64.whl is not a supported wheel on this platform.
```

主因是 Colab 的 python 版本過新，降級 python 就可以了（3.11→３.10）

### 改用 [本地(macbook)]  

作業系統不符，直接沒得玩

```terminal
❯ python3.10 -m pip install -r requirements-mac.txt

Looking in indexes: https://pypi.org/simple, https://download.pytorch.org/whl/cu118
Collecting ttsfrd-dependency==0.1 (from -r requirements-mac.txt (line 35))
  Downloading https://www.modelscope.cn/models/speech_tts/speech_kantts_ttsfrd/resolve/master/ttsfrd_dependency-0.1-py3-none-any.whl (1.1 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.1/1.1 MB 1.6 MB/s eta 0:00:00
ERROR: ttsfrd-0.3.9-cp310-cp310-linux_x86_64.whl is not a supported wheel on this platform.
```

## 心得

效果蠻好的，聲音非常像難以辦認，但是在長文朗讀或是歌唱發聲表現不知道如何？  
學習了 Kaggle Colab 兩個工具，了解一下網路與資源(GPU)設定相關的題目
地端開發的話，可能 Linux 才是最佳解，會比 Macbook 更好 ??
但是 GPU 仍是重要資源。

## 參考

- <https://www.mediatek.tw/blog/%E8%81%AF%E7%99%BC%E5%89%B5%E6%96%B0%E5%9F%BA%E5%9C%B0%E5%85%A8%E9%9D%A2%E9%96%8B%E6%BA%90-breeze2>
- <https://huggingface.co/spaces/Splend1dchan/BreezyVoice-Playground>
- [How to downgrade python version in Colab](https://www.geeksforgeeks.org/how-to-downgrade-python-version-in-colab/)
- <https://www.modelscope.cn/models/speech_tts/speech_kantts_ttsfrd>

(fin)
