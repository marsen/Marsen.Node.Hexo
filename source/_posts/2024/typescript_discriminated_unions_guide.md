---
title: " [學習筆記] 使用 Discriminated Unions 解決多狀態問題"
date: 2024/10/05 00:30:00:
tags:
  - 學習筆記
---

## 前情提要

在日常開發中，我們常需要定義不同狀態下的數據結構，這類需求通常涉及到多個狀態及其對應的屬性。  
在 TypeScript 中，如果不加以控制，這些狀態容易變成一個充滿可選屬性的複雜物件，導致程式碼難以管理。  
我們有一種更優雅的解決方案 —— `Discriminated Unions`，  
它在 TypeScript 中幫助我們避免常見的「bag of optionals」的問題。  

## 主文

假設我們正在開發一個付款流程，並需要一個 PaymentState 類型來描述付款狀態：

```typescript
type PaymentState = {
  status: "processing" | "success" | "failed";
};
```

此時，我們還需要根據不同狀態保存資料或錯誤訊息，  
因此我們為 PaymentState 類型新增了可選的 receiptUrl 和 error 屬性：  

```typescript
type PaymentState = {
  status: "processing" | "success" | "failed";
  errorMessage?: string;
  receiptUrl?: string;
};
```

這個定義表面上看起來可行，但在實際使用時容易出現問題。  
假設我們定義了一個渲染 UI 的函式 renderUI：

```typescript
const renderUI = (state: PaymentState) => {
  if (state.status === "processing") {
    return "Payment Processing...";
  }

  if (state.status === "failed") {
    return `Error: ${state.errorMessage.toUpperCase()}`;
  }

  if (state.status === "success") {
    return `Receipt: ${state.receiptUrl}`;
  }
};
```

TypeScript 提示 state.errorMessage 可能是 undefined，  
這意味著我們在某些狀態下無法安全地操作 errorMessage 或 receiptUrl 屬性。  
這是因為這兩個屬性是可選的，並且沒有和 status 明確地綁定。  
這就造成了**類型過於鬆散**。

## 解決方案：Discriminated Unions

為了解決這個問題，我們可以使用 Discriminated Unions。  
這種模式可以將多個狀態拆分為具體的物件，並通過共同的 status 屬性來區分每個狀態。

首先，我們可以將每個狀態單獨建模：

```typescript
type State =
  | { status: "processing" }
  | { status: "error"; errorMessage: string }
  | { status: "success"; receiptUrl: string };
```

這樣一來，我們就能明確地將 errorMessage 屬性與 error 狀態綁定，  
並且 receiptUrl 屬性只會出現在 success 狀態下。  
當我們在 renderUI 函式中使用這些屬性時，TypeScript 會自動根據 status 來縮小類型範圍：

```typescript
const renderUI = (state: State) => {
  if (state.status === "processing") {
    return "Processing...";
  }

  if (state.status === "error") {
    return `Error: ${state.errorMessage.toUpperCase()}`;
  }

  if (state.status === "success") {
    return `Receipt: ${state.receiptUrl}`;
  }
};
```

TypeScript 現在能夠根據 status 確保 errorMessage 在 error 狀態下是一個字串，  
並且 receiptUrl 只會在 success 狀態出現。  
這大大提高了我們程式碼的安全性和可讀性。  

### 進一步優化：提取類型別名

為了讓程式碼更加清晰，我們可以將每個狀態定義為單獨的類型別名：

```typescript
type LoadingState = { status: "processing" };
type ErrorState = { status: "error"; errorMessage: string };
type SuccessState = { status: "success"; receiptUrl: string };

type State = LoadingState | ErrorState | SuccessState;
```

這樣的結構不僅清晰易讀，也方便日後的擴展。  
如果我們需要增加其他狀態，比如 idle 或 cancelled，只需要新增對應的類型別名即可，保持程式碼的擴展性和一致性。  

## 小結

Discriminated Unions 是 TypeScript 中處理多狀態情境的強大工具，特別適合用來解決「可選屬性的大集合」問題。它能夠：

確保每個屬性和狀態間的強關聯，減少錯誤和不一致性。  
提升程式碼的可讀性和維護性，並且有助於擴展性。  
讓 TypeScript 自動推斷正確的屬性類型，避免不必要的空值檢查。  
當你發現程式碼中有過多的可選屬性且沒有強烈關聯時，考慮使用 discriminated unions 來重構並簡化你的類型定義。

## 參考

- [TypeScript Discriminated Unions for Frontend Developers](https://www.totaltypescript.com/discriminated-unions-are-a-devs-best-friend)
- [TypeScript Handbook - Unions and Intersection Types](https://www.typescriptlang.org/docs/handbook/unions-and-intersections.html)

(fin)
