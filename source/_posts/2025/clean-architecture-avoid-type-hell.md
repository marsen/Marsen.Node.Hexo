---
title: " [實作筆記] Clean Architecture 思考：避免過度設計"
date: 2025/09/11 15:23:42
tags:
  - 實作筆記
---

## 前情提要

在實作強型別語言，經常會遇到一些僅次於命名的問題：

到底要建立多少層級的 DTO/型別？

什麼時候該抽象，什麼時候該保持簡單？

這篇文章記錄一個前端要求帶來的反思，如何在「架構純粹性」與「實用主義」之間找到平衡點？

## 問題場景

系統架構為 TypeScript 並實作 Clean Architecture。

我們有一個比對系統，Domain Entity 中的 `statusCode` 定義為 `string | null`，

但前端要求 API 必須回傳 `string` 型別，允許空字串而不處理 null。

```typescript
// Domain Entity
export class Comparison {
  statusCode: string | null  // 業務邏輯：可能為空
}

// 前端期望的 API 回應
{
  "statusCode": ""  // 必須是 string，不能是 null
}
```

## 解決方案演進

### 第一版：UseCase 層建立完整 DTO

```typescript
// 建立回應 DTO
export interface ComparisonResponseDTO {
  id: string
  statusCode: string  // 轉換為 string
  // ... 其他欄位
}

// 建立 UseCase 回應型別
export type GetComparisonListResDTO = IDataAndCount<ComparisonResponseDTO>

// UseCase 中處理轉換
private toResponseDTO(comparison: Comparison): ComparisonResponseDTO {
  return {
    // ...
    statusCode: comparison.statusCode || '',
    // ...
  }
}
```

看起來很「Clean Architecture」，但真的有必要嗎？

關鍵觀點，有沒有可能過度設計了？

### 最終方案：Controller 邊界處理

```typescript
// Controller 直接處理轉換
async getComparisonList(req: Request, res: Response): Promise<void> {
  const { data, count } = await this.getComparisonListUseCase.execute(input)
  
  res.status(200).json({
    data: data.map(comparison => ({
      ...comparison,
      statusCode: comparison.statusCode || ''
    })),
    pagination
  })
}
```

## 避免型別地獄的原則

### 1. YAGNI 原則 (You Aren't Gonna Need It)

不要為了「完整性」而建立無意義的型別包裝

或是建立過多的 Mapper 類別

```typescript
// ❌ 過度抽象
export type GetComparisonListResDTO = IDataAndCount<ComparisonResponseDTO>

// ✅ 直接使用泛型
IUseCase<GetComparisonListReqDTO, IDataAndCount<ComparisonResponseDTO>>
```

### 2. 什麼時候需要抽象化？

我的判斷，重複的時候，例如，當 2 個以上的 API 需要相同的型別時，才考慮抽象

也不排除可以能轉換極度複雜，這時有一個 Mapper 的導入反而可以減輕負擔時才導入。

架構／Design Pattern 應該服務 RD 而不是折摩 RD

```typescript
// 如果只有一個 API 使用，直接 inline
data.map(item => ({ ...item, statusCode: item.statusCode || '' }))

// 如果多個 API 都需要，才建立共用函數或型別
const transformComparison = (item: Comparison) => ({
  ...item,
  statusCode: item.statusCode || ''
})
```

### 3. 複雜度評估

簡單的轉換邏輯不需要額外抽象：

```typescript
// ✅ 簡單轉換，直接處理
statusCode: comparison.statusCode || ''

// ❌ 為簡單邏輯建立複雜抽象
private transformStatusCode(statusCode: string | null): string {
  return statusCode || ''
}
```

## 實用主義 vs 理論完美

### 現代 TypeScript 最佳實踐

1. **直接使用泛型** - 避免不必要的 type alias
2. **減少型別層級** - 除非有明確的業務意義
3. **保持簡潔** - 不為了「完整性」而建立無意義的包裝

### 架構決策的平衡點

| 考量因素 | 過度設計 | 適度設計 | 設計不足 |
|---------|---------|---------|---------|
| 型別數量 | 為每個 UseCase 建專屬 DTO | 共用 + 泛型 | 沒有型別安全 |
| 轉換位置 | 每層都轉換 | 邊界處理 | 隨意放置 |
| 複雜度 | 型別地獄 | 恰到好處 | 難以維護 |

## 進階判斷：何時需要抽象，何時保持簡單？

| 判斷角度       | 適合抽象化的情境 | 適合保持簡單的情境 |
|----------------|------------------|--------------------|
| **使用頻率**   | 多個 UseCase 或 API 重複出現 → 建立共用型別/函數 | 僅在單一 Controller 出現一次 → inline 即可 |
| **業務語意**   | 欄位轉換具有業務意義（例：狀態機的一環） → 抽象進 Domain/UseCase | 純展示需求（例：`null → ""`） → 邊界層處理 |
| **團隊規模與可讀性** | 大團隊、專案壽命長 → 適度抽象，降低未來重構風險 | 小團隊、短期專案 → 保持簡單，降低溝通成本 |
| **錯誤影響範圍** | 錯誤會影響商業邏輯（例：金額精度） → 上升到 UseCase/Domain | 只影響 API 輸出表現（例：空字串 vs. null） → Controller 處理 |
| **演進空間**   | 需求可能持續變化（欄位規則增多/不同前端需求） → 抽象留彈性 | 需求相對穩定 → inline 保持簡潔 |


## 小結

Clean Architecture 的精神是**將商業邏輯集中在核心，邊界負責格式適配**。

不要被「層級完整性」綁架，重點是：

1. **Domain 保持純粹** - 反映真實的業務狀態
2. **UseCase 專注邏輯** - 編排業務流程，不處理格式
3. **Controller 處理邊界** - HTTP 格式轉換在此進行
4. **實用主義優先** - 簡單的需求用簡單的方法

記住：**改動越少越好，沒有必要就不要建一大堆型別**。

架構是為了解決問題，不是為了展示理論知識。當實用主義與理論衝突時，選擇能讓團隊更有生產力的方案。

(fin)
