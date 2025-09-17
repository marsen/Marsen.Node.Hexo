---
title: " [實作筆記] 命令式程式碼重構到函數式 Pipe 流水線"
date: 2025/09/17 16:53:43
tags:
  - 實作筆記
---
## 前言

最近小朋友在進行中文轉換規則的開發時，遇到了一個典型的程式碼異味。

你可以先挑戰看一下有沒有辦法識別。

```typescript
// 重構前的程式碼
apply(text: string): string {
  const protectedChars = ['台']
  
  // 1. 標記階段
  let protectedText = text
  protectedChars.forEach((char) => {
    protectedText = protectedText.replaceAll(char, `<<<${char}>>>`)
  })

  // 2. 轉換階段
  let result = this.baseConverter(protectedText)

  // 3. 還原階段
  protectedChars.forEach((char) => {
    const convertedChar = this.baseConverter(char)
    result = result.replaceAll(`<<<${convertedChar}>>>`, char)
  })

  // 4. 自訂轉換
  result = this.customConverter(result)
  
  return result
}
```

這篇文章記錄了我如何將一個命令式的方法重構為函數式的 pipe 流水線，

讓程式碼變得更加簡潔和易讀。

## 壞味道

原始的 `ChineseConversionRule` 中的 `apply` 方法存在以下問題：

1. **過多中間變數**：`text` → `protectedText` → `result`
2. **命令式寫法**：透過變數重新賦值來處理資料流
3. **邏輯分散**：標記保護字符和還原的邏輯內嵌在主方法中

過多的中間變數和命令式的寫法讓程式碼顯得冗長且不夠優雅。

protectedText、result、text　都是。

這裡的挑戰是要對原始的字串加工

在不破壞原本邏輯的情況下，保留擴充的彈性，並保持程式結構清晰、可維護。

可能有很多模式可以解決(Decorator / Template Method / Pipeline)

小朋友寫也得也不差了，我們試著讓它更好

## 重構過程

### 階段一：消除中間變數

第一步是消除不必要的中間變數，直接在同一個變數上操作：

可以看到修改後，只有 text 一個變數，還是傳入了的

越少變數，越不用花心思思考命名，減少認知負擔，而本質上這些冗餘的變數的確是同質可以刪除的

這是一種隱性的重複壞味道。

```typescript
apply(text: string): string {
  const protectedChars = ['台']
  
  // 直接操作 text 變數，消除 protectedText 和 result
  protectedChars.forEach((char) => {
    text = text.replaceAll(char, `<<<${char}>>>`)
  })

  text = this.baseConverter(text)

  protectedChars.forEach((char) => {
    const convertedChar = this.baseConverter(char)
    text = text.replaceAll(`<<<${convertedChar}>>>`, char)
  })

  return this.customConverter(text)
}
```

### 階段二：引入 Functional Programming

接下來導入函數式程式設計的概念，使用 pipe 模式來處理資料流：

這裡要先看懂 `pipe`，簡單理解它把一個初始值依序丟進多個函數，前一個輸出就是下一個的輸入。

可以看到一些明顯的壞味道，**重複的 protectedChars**，1 跟 3 本身是匿名函數，讀起來也沒那麼好理解，

這是重構必經之路，我們再往下走

```typescript
apply(text: string): string {
  const protectedChars = ['台']
  
  return this.pipe(
    text,
    // 1. 標記階段：將需要保護的字符標記為不轉換
    (input) => protectedChars.reduce((acc, char) =>
      acc.replaceAll(char, `<<<${char}>>>`), input),
    
    // 2. 轉換階段：使用 OpenCC 進行轉換
    this.baseConverter,
    
    // 3. 還原階段：將標記的字符還原為原始字符
    (input) => protectedChars.reduce((acc, char) => {
      const convertedChar = this.baseConverter(char)
      return acc.replaceAll(`<<<${convertedChar}>>>`, char)
    }, input),
    
    // 4. 使用自訂轉換器進行模糊字詞的修正
    this.customConverter
  )
}

// 自製的 pipe 函數
private pipe<T>(value: T, ...fns: Array<(arg: T) => T>): T {
  return fns.reduce((acc, fn) => fn(acc), value)
}
```

### 階段三：職責分離

將標記和還原邏輯抽取成獨立的私有方法：

將重複出現的 `protectedChars` 提升為類別屬性：

也不需要把匿名函數寫一坨在 pipe 裡面，這時要煩惱的只有方法的名字要怎麼才達意

但至少我們還有註解。

更進一步可以簡化方法內的參數名，因為 scope 很小，不會有認知負擔

```typescript
export class ChineseConversionRule implements IRule {
  private baseConverter: ConvertText
  private customConverter: ConvertText
  private readonly protectedChars = ['台']  // 統一管理

  // ... constructor

apply(text: string): string {
  return this.pipe(
    text,
    // 1. 標記階段：將需要保護的字符標記為不轉換
    this.markProtectedChars,
    
    // 2. 轉換階段：使用 OpenCC 進行轉換
    this.baseConverter,
    
    // 3. 還原階段：將標記的字符還原為原始字符
    this.restoreProtectedChars,
    
    // 4. 使用自訂轉換器進行模糊字詞的修正
    this.customConverter
  )
}

  /**
   * 標記需要保護的字符
   */
  private markProtectedChars = (input: string): string => {
    return this.protectedChars.reduce((acc, c) => acc.replaceAll(c, `<<<${c}>>>`), input)
  }

  /**
   * 還原被標記的保護字符
   */
  private restoreProtectedChars = (input: string): string => {
    return this.protectedChars.reduce((acc, c) => {
      const convertedChar = this.baseConverter(c) // 例如：'台' -> '臺'
      return acc.replaceAll(`<<<${convertedChar}>>>`, c)
    }, input)
  }
```

## 重構成果

1. **簡潔性**：主方法從 20 行縮減到 8 行
2. **可讀性**：資料流向清晰，從上到下一目了然
3. **可測試性**：每個步驟都是純函數，可以獨立測試
4. **可維護性**：職責分離，邏輯集中管理
5. **函數式**：無副作用，符合 FP 原則

### 效能考量

- 測試結果顯示功能完全正常，262 個測試案例全數通過，這是個大前提，沒有測試沒有重構
- 重構過程中沒有改變演算法複雜度
- Pipe 函數本身的開銷微乎其微

## 關於 Pipe 的選擇

在重構過程中考慮過使用現成的函式庫：

- **Ramda**：功能最完整的 FP 函式庫
- **Lodash/fp**：輕量級選擇
- **fp-ts**：型別安全的 FP 函式庫

最終選擇自製 pipe 函數的理由：

兩行程式碼，沒有多餘依賴，寫法完全貼合專案需求。

團隊看了就能用，不用再去學新的函式庫。

邏輯也很單純，後續維護起來相對輕鬆。

除非更大範圍的重複發生，不然不需要額外引用套件突增學習成本

```typescript
private pipe<T>(value: T, ...fns: Array<(arg: T) => T>): T {
  return fns.reduce((acc, fn) => fn(acc), value)
}
```

## RD 反饋

這次重構讓我深刻體會到函數式程式設計的優雅之處：

1. **資料即流水線**：透過 pipe 讓資料在各個函數間流動
2. **純函數的威力**：每個步驟都可預測、可測試
3. **組合勝過繼承**：透過函數組合建構複雜邏輯
4. **漸進式重構**：一步步改善，降低風險

從命令式到函數式的重構不僅讓程式碼變得更優雅，也提升了整體的可維護性。

雖然函數式程式設計有一定的學習曲線，但一旦掌握了基本概念，就能寫出更簡潔、更易懂的程式碼。

重構的關鍵在於：**小步快跑，持續改善**。每一次小的改進都讓程式碼朝著更好的方向發展，這正是軟體工藝精神的體現。

## 小結

嗯，小朋友很會用 AI 寫作文呢。

(fin)
