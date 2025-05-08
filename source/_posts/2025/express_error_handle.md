---
title: " [實作筆記] Node.js Express 錯誤處理全攻略：同步、非同步與 Stream Error"
date: 2025/05/08 13:47:43
tags:
  - 實作筆記
---

## 前情提要

Node.js + Express 是開發 REST API 時最常見的組合。  
這篇文章會試著寫出三大常見錯誤類型的處理方式：

- 同步錯誤（throw）
- 非同步錯誤（async/await）
- Stream error（例如檔案下載）

如果你用的是 Express 5，其實已經支援原生 async function 錯誤攔截，連 async wrapper 都可以省下！

## Express 的錯誤處理 middleware 是怎麼運作的？

只要你呼叫 `next(err)`，Express 就會跳到錯誤處理 middleware：

```typescript
app.use((err, req, res, next) => {
  console.error(err)
  res.status(500).json({ message: err.message })
})
```

這個 middleware 需要有四個參數，Express 會自動辨識它是錯誤處理 middleware。

### 同步錯誤怎麼處理？

這種最簡單，直接 throw 就可以：

```typescript
app.get('/sync-error', (req, res) => {
  throw new Error('同步錯誤')
})
```

Express 會自動幫你捕捉，送進錯誤 middleware，不需要特別處理。

### Express 4 async/await 的錯誤要小心

Express 4 的情況：
你可能會寫這樣的 code：

```typescript
app.get('/async-error', async (req, res) => {
  throw new Error('async/await 錯誤') // ❌ 不會被 Express 捕捉
})
```

這樣寫會直接讓整個程式 crash，因為 Express 4 不會自動處理 async function 裡的錯誤。

#### 解法：自己包一層 asyncWrapper

```typescript
export const asyncWrapper = (fn) => (req, res, next) => {
  fn(req, res, next).catch(next)
}
```

使用方式：

```typescript
app.get('/async-error', asyncWrapper(async (req, res) => {
  throw new Error('async/await 錯誤')
}))
```

這樣就能安全把錯誤丟給錯誤 middleware 處理。

### Express 5：原生支援 async function

如果你用的是 Express 5，那更簡單了，直接寫 async function，Express 就會自動捕捉錯誤，完全不需要 async wrapper！

```typescript
app.get('/async-error', async (req, res) => {
  const data = await getSomeData()
  if (!data) throw new Error('找不到資料')
  res.json(data)
})
```

是不是清爽多了？

!!注意：要確認你的專案使用的是 <express@5.x> 版本。
可以用以下指令確認

```bash
npm list express
```

### Stream error：Express 捕不到

問題在哪？
stream 錯誤是透過 EventEmitter 的 'error' 事件傳遞，Express 根本不知道有這回事，例如：

```typescript
app.get('/file', asyncWrapper(async (req, res) => {
  const file = getSomeGCSFile()
  file.createReadStream().pipe(res) // ❌ 權限錯誤會 crash，Express 不會接到
}))
```

#### 正確作法：自己監聽 'error'，再丟給 next()**

```typescript
app.get('/file', asyncWrapper(async (req, res, next) => {
  const file = getSomeGCSFile()
  const stream = file.createReadStream()
  stream.on('error', next)
  stream.pipe(res)
}))
```

#### Bonus：包成一個工具函式

```typescript
// utils/streamErrorHandler.ts
import { Response, NextFunction } from 'express'
import { Readable } from 'stream'

export function pipeWithErrorHandler(stream: Readable, res: Response, next: NextFunction) {
  stream.on('error', next)
  stream.pipe(res)
}
```

使用：

```typescript
app.get('/file', asyncWrapper(async (req, res, next) => {
  const file = getSomeGCSFile()
  pipeWithErrorHandler(file.createReadStream(), res, next)
}))
```

## 整體範例程式

```typescript
import express from 'express'
import { asyncWrapper } from './utils/asyncWrapper'
import { pipeWithErrorHandler } from './utils/streamErrorHandler'

// 模擬 GCS 檔案物件
function getSomeGCSFile(shouldError: boolean) {
  const { Readable } = require('stream')
  if (shouldError) {
    // 會 emit error 的 stream
    const stream = new Readable({
      read() {
        this.emit('error', new Error('stream error 測試'))
      }
    })
    return {
      createReadStream: () => stream
    }
  } else {
    // 正常回傳資料
    return {
      createReadStream: () => Readable.from(['Hello World'])
    }
  }
}

const app = express()

// 同步錯誤
app.get('/sync-error', (req, res) => {
  throw new Error('同步錯誤')
})

// 非同步錯誤（Express 4 寫法）
app.get('/async-error-4', asyncWrapper(async (req, res) => {
  throw new Error('async/await 錯誤, Express 4.x 以下的寫法')
}))

// 非同步錯誤（Express 5 寫法）
app.get('/async-error', async (req, res) => {
  throw new Error('async/await 錯誤')
})

// stream error（正確處理）
app.get('/file', asyncWrapper(async (req, res, next) => {
  const file = getSomeGCSFile(true)
  pipeWithErrorHandler(file.createReadStream(), res, next)
}))

// stream error（沒處理會 crash）
app.get('/file-no-handle', asyncWrapper(async (req, res, next) => {
  const file = getSomeGCSFile(true)
  file.createReadStream().pipe(res)
}))

// 錯誤 middleware
app.use((err, req, res, next) => {
  console.error(err)
  res.status(500).json({ message: err.message })
})

app.listen(3000, () => {
  console.log('Server started')
})
```

如果你用的是 Express 5，可以把 asyncWrapper 都拿掉，程式碼會更簡潔！

## 小結

| 類型           | 會自動處理？         | 解法                                      |
| -------------- | -------------------- | ----------------------------------------- |
| 同步錯誤       | ✅ Express 自動處理   | 直接 throw 就好                           |
| async/await 錯誤 | ❌ Express 4 不會處理 | 用 asyncWrapper（Express 4） |
|                  | ✅ Express 5 自動處理 | 或直接寫 async function（Express 5） |
| Stream error   | ❌ 完全不會處理       | 監聽 stream.on('error', next)             |

## 參考

- <https://github.com/marsen/express-handle-err>

(fin)
