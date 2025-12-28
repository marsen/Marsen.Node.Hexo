---
title: " [學習筆記] Node.js 檔案操作 mkdir 的正確姿勢"
date: 2025/08/07 13:15:41
tags:
  - 學習筆記
---

## 前情提要

在 code review 中 RD 寫了以下的程式

```typescript
if (!await this.exists(fullPath)) {
  await fs.promises.mkdir(fullPath, { recursive: true })
}
```

看似合理的「檢查然後執行」（Check-Then-Act）模式可能導致的併發問題。

假設兩個請求同時上傳檔案到同一個不存在的目錄：

```text
時間線：
T1: 請求A 執行 this.exists(fullPath) → 返回 false (目錄不存在)
T2: 請求B 執行 this.exists(fullPath) → 返回 false (目錄不存在)
T3: 請求A 執行 mkdir(fullPath) → 成功建立目錄
T4: 請求B 執行 mkdir(fullPath) → 可能拋出 EEXIST 錯誤(fs.promises.mkdir 已在底層排除這個問題)
```

雖然使用了 `recursive: true`，那前面的檢查很可能是不必要的行為。

### 為什麼會這樣？

問題的根源在於**時間窗口**。兩個步驟之間存在時間差，而這個時間差就是競態條件的溫床：

```typescript
// ❌ 有時間窗口的寫法
if (!await this.exists(fullPath)) {  // 步驟1: 檢查
  // 👆 這裡到下面之間就是危險的時間窗口
  await fs.promises.mkdir(fullPath, { recursive: true })  // 步驟2: 執行
}
```

## 解決方案

### 方法1: 直接使用 mkdir（推薦）

```typescript
async uploadFile(file: UploadedFile, pathToStore?: `/${string}`): Promise<string> {
  const fullPath = `${this.uploadDir}${pathToStore ?? ''}`
  // 直接建立目錄，recursive: true 會自動處理已存在的情況
  await fs.promises.mkdir(fullPath, { recursive: true })
  const uploadPath = path.join(fullPath, file.originalName)
  await fs.promises.writeFile(uploadPath, file.buffer)
  return uploadPath
}
```

## 為什麼 recursive: true 已經足夠

根據 Node.js 官方文件，`fs.promises.mkdir(path, { recursive: true })` 具有以下特性：

1. **自動建立父目錄**：如果父目錄不存在會自動建立
2. **處理已存在目錄**：當 `recursive` 為 `true` 時，如果目錄已存在不會拋出錯誤
3. **簡化錯誤處理**：避免了手動檢查目錄是否存在的需要

官方範例：

```javascript
import { mkdir } from 'node:fs';

// Create ./tmp/a/apple, regardless of whether ./tmp and ./tmp/a exist.
mkdir('./tmp/a/apple', { recursive: true }, (err) => {
  if (err) throw err;
});

這就像餐廳點餐的差別：

```typescript
// ❌ 競態條件版本（不好的做法）
if (餐廳沒有準備我要的餐) {  // 檢查
  請廚師準備這道餐        // 執行
}
// 問題：兩個客人可能同時檢查到「沒有」，然後都要求準備

// ✅ 直接執行版本（好的做法）
請廚師準備這道餐，如果已經有了就不用重複準備
// 廚師會自己判斷是否需要準備，避免重複工作
```

## 效能優勢

修復後還有意外的效能提升：

```typescript
// 修復前：2次系統調用
await this.exists(fullPath)  // 系統調用1: stat()
await fs.promises.mkdir()    // 系統調用2: mkdir()

// 修復後：1次系統調用  
await fs.promises.mkdir(fullPath, { recursive: true })  // 系統調用1: mkdir()
```

## 經驗教訓

1. **避免 Check-Then-Act 模式**：這是併發程式設計的經典陷阱，不過這次案例，執行的底層實作已處理好，所以不會有問題
2. **信任系統調用**：現代 API 通常已經考慮了併發場景
3. **簡單就是美**：移除不必要的檢查邏輯，程式碼更簡潔也更安全

## 參考

- [Nodejs - fsPromises.mkdir(path[, options])](https://nodejs.org/api/fs.html#fsmkdirpath-options-callback)

(fin)
