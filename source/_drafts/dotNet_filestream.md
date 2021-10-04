---
title: FileStream正確的使用FileAccess與FileShare參數
tag:
  - .Net Framework
  - IO
  - 記錄
---
## 前情提要

使用FileStream存取檔案是基本功,但是沒有了解底層細節,
在多執行緒同時存取與編輯檔案時,將會引爆咬檔地雷。

## 加碼 HttpContent 與 Stream

## 測試工具 `LinQPad`

## 參考

- <https://stackoverflow.com/questions/25097773/system-io-filestream-fileaccess-vs-fileshare>
- <https://referencesource.microsoft.com/#q=FileStream>