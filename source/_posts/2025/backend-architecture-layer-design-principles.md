---
title: " [架構筆記] Clean Architecture 分層職責：一道值得深思的面試題"
date: 2025/08/12 13:29:15
tags:
  - 學習筆記
---

## 前情提要

最近在 Code Review 時遇到一個有趣的討論：檔案上傳功能中，檔案編碼修復應該放在 Middleware 還是 UseCase？

這個問題引發了我對 Clean Architecture 分層職責的重新思考，也讓我意識到這是一個很好的面試題目。畢竟，分層架構不只是把程式碼分資料夾這麼簡單，背後有著更深層的設計哲學。

## 問題背景

在一個採用 Clean Architecture 的專案中，我們有兩個檔案上傳功能：

- **知識庫檔案上傳** - 支援 PDF、Word、圖片等多種格式
- **合約檔案上傳** - 原始合約支援 PDF/Word，簽署後合約只支援 PDF

當前系統架構包含：

- **Middleware** - Express 中介軟體層
- **Controller** - HTTP 請求處理層  
- **UseCase** - 業務邏輯層
- **Service** - 基礎設施服務層

## 核心問題：分層職責如何劃分？

以下邏輯應該放在哪一層？

1. **檔案編碼修復** - 解決中文檔名亂碼問題 (latin1 → utf8)
2. **檔案類型驗證** - 檢查 MIME type 是否符合業務需求  
3. **檔案大小限制** - 根據不同業務場景設定不同大小限制
4. **JWT Token 驗證** - 檢查使用者身份
5. **檔案內容解析** - 提取 PDF/Word 文件內容

## 分析思路

### Middleware 的職責：技術基礎設施

```typescript
// ✅ 適合放 Middleware
// authMiddleware.ts - HTTP 認證技術處理
- JWT token 解析和驗證  
- HTTP 權限檢查
- 將認證結果附加到 req.user

// errorHandler.ts - HTTP 錯誤處理  
- 將系統錯誤轉換為 HTTP 狀態碼
- 統一錯誤格式回應
- 錯誤日誌記錄

// uploadFiles.ts - 檔案上傳技術處理
- 檔案編碼修復 (HTTP 上傳技術問題) ✅
- Multer 設定和記憶體存儲
```

### UseCase 的職責：業務邏輯驗證

```typescript
// ✅ 適合放 UseCase
// UploadKnowledgeFiles.ts - 知識庫檔案業務邏輯
class UploadKnowledgeFiles {
  private readonly validMimeTypes = {
    'application/pdf': 'pdf',
    'text/plain': 'txt',
    'application/msword': 'word',
    'image/jpeg': 'image',
    'image/png': 'image',
    // ... 更多類型
  } as const
  
  // 檔案類型白名單驗證 (業務規則)
  // 檔案大小限制檢查 (業務需求)  
  // 檔案數量限制驗證 (業務邏輯)
}

// UploadContractFile.ts - 合約檔案業務邏輯
class UploadContractFile {
  // 不同類型合約的檔案限制 (業務場景)
  private readonly validMimeTypesForOrigin = {
    'application/pdf': 'pdf',
    'application/vnd.openxmlformats-officedocument.wordprocessingml.document': 'word',
  } as const

  private readonly validMimeTypesForNonOrigin = {
    'application/pdf': 'pdf',  // 簽署後只能 PDF
  } as const
}
```

## 設計原則

### Middleware 處理的是...

- **跨領域技術問題** (認證、編碼、錯誤處理)
- **HTTP 協議相關** (請求解析、回應格式)  
- **基礎設施關注點** (日誌、監控、安全)
- **與業務無關的技術細節**

### UseCase 處理的是...

- **特定業務場景的規則** (不同業務有不同檔案限制)
- **領域知識驗證** (合約標題、內容檢查)
- **業務流程邏輯** (檔案處理、資料儲存)
- **動態配置的業務參數** (從環境變數讀取的業務限制)

## 實際案例分析

### 檔案編碼問題：技術問題 → Middleware ✅

```typescript
// uploadFiles.ts
const fixFilenameEncoding = (filename: string): string => {
  return Buffer.from(filename, 'latin1').toString('utf8')
}

const upload = multer({
  storage: multer.memoryStorage(),
  fileFilter: (_req, file, cb) => {
    file.originalname = fixFilenameEncoding(file.originalname)
    cb(null, true)
  },
})
```

**為什麼放 Middleware？**

- 這是 HTTP 上傳過程中的技術問題，不是業務邏輯
- 所有檔案上傳都需要這個處理，跨多個 UseCase
- 屬於請求處理層面的責任

### 檔案類型限制：業務邏輯 → UseCase ✅

```typescript
// UploadContractFile.ts  
class UploadContractFile {
  execute(files: Express.Multer.File[], isOrigin: boolean) {
    const validTypes = isOrigin 
      ? this.validMimeTypesForOrigin 
      : this.validMimeTypesForNonOrigin
      
    // 業務邏輯：根據合約類型決定允許的檔案格式
    this.validateFileTypes(files, validTypes)
  }
}
```

**為什麼放 UseCase？**

- 不同業務場景有不同的檔案類型限制
- 這是領域知識，需要業務邏輯判斷
- 可能隨著業務需求變化而調整

## 進階思考

### 如果檔案大小限制要動態調整呢？

```typescript
// UseCase 層處理動態業務配置
class UploadKnowledgeFiles {
  constructor(
    private configService: ConfigService,
    private userService: UserService
  ) {}
  
  async execute(files: Express.Multer.File[], userId: string) {
    const user = await this.userService.findById(userId)
    const maxSize = user.isVip 
      ? this.configService.get('VIP_MAX_FILE_SIZE')
      : this.configService.get('NORMAL_MAX_FILE_SIZE')
      
    this.validateFileSize(files, maxSize)
  }
}
```

這樣的設計對測試也更友好：

```typescript
// 可以輕鬆 mock 依賴，測試不同的業務場景
describe('UploadKnowledgeFiles', () => {
  it('should allow larger files for VIP users', async () => {
    // Arrange
    mockUserService.findById.mockResolvedValue({ isVip: true })
    mockConfigService.get.mockReturnValue(100 * 1024 * 1024) // 100MB
    
    // Act & Assert
    await expect(uploadUseCase.execute(largeFiles, 'vip-user-id'))
      .resolves.not.toThrow()
  })
})
```

## 總結

好的架構設計不是憑感覺，而是要有明確的原則：

- **Middleware** = 技術基礎設施，處理 HTTP 層面的問題
- **UseCase** = 業務邏輯驗證，處理領域相關的規則

這種分層不只讓程式碼更好維護，也讓測試更容易撰寫，更符合單一職責原則。

下次在設計架構時，不妨問問自己：

- 這個邏輯是技術問題還是業務問題？
- 這個規則會因為業務需求變化嗎？
- 這個處理邏輯需要在多個地方重複嗎？

Clean Architecture 的精神就在於：**讓業務邏輯獨立於技術細節**。

面試時遇到類似問題，記得從職責分離的角度思考，相信你會有不錯的表現！

(fin)