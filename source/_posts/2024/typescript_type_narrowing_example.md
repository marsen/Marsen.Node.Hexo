---
title: " [學習筆記] TypeScript 的 Narrowing Types 與 Boolean 的例外"
date: 2024/10/05 01:48:30
tags:
  - 學習筆記
---


## 前情提要

在使用 TypeScript 時，我們常常會利用條件語句來進行 Narrowing Types，確保程式邏輯的正確性。
例如:
Narrowing Types的例子：使用 typeof

```typescript
const processValue = (value: string | number) => {
  if (typeof value === "string") {
    // 在這裡 TypeScript 已經收窄為 string
    console.log(`String value: ${value.toUpperCase()}`);
  } else {
    // 在這裡 TypeScript 已經收窄為 number
    console.log(`Number value: ${value.toFixed(2)}`);
  }
};
```

### 說明

在這個例子中，我們定義了一個 processValue 函數，它接受一個 string 或 number 類型的參數。  
使用 typeof 來檢查 value 的類型後，TypeScript 會自動進行 Narrowing Types：
當 `typeof value === "string"` 時，TypeScript 將 value 的類型自動收窄為 string，  
所以我們可以安全地使用字串方法，如 `toUpperCase()`。
當 else 條件觸發時，value 類型已經收窄為 number，因此我們可以使用 number 專有的方法，如 toFixed()。
這樣的收窄機制能夠在多態的情況下提高程式的安全性和可讀性，讓開發者更清楚在不同條件下該如何操作不同類型的值。

## 主文

然而，在一些特定情況下，TypeScript 的 Narrowing Types 並不如我們預期的那麼靈活。  
比如說 Boolean() 函數進行條件檢查時，TypeScript 不會像其他檢查方法那樣進行 Narrowing Types。

```typescript
const NarrowFun = (input: string | null) => {
  if (!!input) {
    input.toUpperCase();  // 這裡 TypeScript 收窄為 string
  }
};

const NotNarrowFun = (input: string | null) => {
  if (Boolean(input)) {
    input.toUpperCase();  // 這裡 TypeScript 沒有收窄，仍然是 string | null
  }
};
```

在這段程式碼中，我們希望透過條件語句來檢查 input 是否為 null。  
使用 `!!input` 時，TypeScript 會正確地將 input 的 Narrowing Types為 string，  
但當我們改用 Boolean(input) 時，TypeScript 並沒有收窄，仍然認為 input 可能是 string | null。  

### 為什麼會這樣？  

#### Boolean 函數的行為

Boolean 是 JavaScript 的一個內建函數，它接受任何值並將其轉換為 true 或 false，  
基於 JavaScript 的「true」或「false」邏輯。  
然而，TypeScript 並不視 Boolean(input) 作為一個能收窄類型的操作。  
它只將 Boolean(input) 視為一個普通的布林值返回，不會對 input 進行更深入的類型推斷。  

在 TypeScript 中，Boolean 函數的定義是這樣的：

```typescript
interface BooleanConstructor {
  new(value?: any): Boolean;
  <T>(value?: T): boolean;
}

declare var Boolean: BooleanConstructor;
```

如你所見，Boolean 是一個接受任何類型 (any) 的值作為輸入，並返回 boolean 類型的結果。  
它會將輸入轉換為布林值 (true 或 false)，但並不提供關於輸入值的具體類型資訊。  
因此，TypeScript 只知道返回的是一個 boolean，而無法推斷 input 的原始類型是否已經被過濾（例如從 string | null 到 string）。  

#### 缺乏類型推斷

在 Boolean(input) 中，TypeScript 僅知道 input 被轉換為 true 或 false，  
但它不會從中推斷 input 的實際類型。  
因此，TypeScript 並沒有收窄 input 為 string 或 null。  
這就是為什麼即使你在 if(Boolean(input)) 內部，input 依然是 `string | null`，而不是單純的 `string`。

```typescript
if (Boolean(input)) {
  console.log(input);  // TypeScript 仍視 input 為 string | null
}
```

#### 類型收窄的條件

TypeScript 進行類型收窄的條件是根據語法或邏輯條件來進行推斷，  
例如 `typeof`、`instanceof`、比較操作 (`==`, `===)` 等。  
這些條件能幫助 TypeScript 推斷出更具體的類型。  
當你使用 typeof input === "string" 時，TypeScript 可以自動將 input 收窄為 string，  
因為這是一個明確的類型檢查：

```typescript
if (typeof input === "string") {
  console.log(input);  // 現在 input 是 string 類型
}
```

#### 為什麼 !!input 能收窄

與 Boolean(input) 不同，`!!input` 能夠收窄類型。  
因為這個雙重否定操作 (!!) 是 JavaScript 的一個常見模式，它會將任意值轉換為布林值。  
由於 TypeScript 能識別這個模式，當你寫 if(!!input) 時，  
TypeScript 可以推斷 input 是 truthy，並將 null 或 undefined 排除在外。  
因此，input 被收窄為一個具體的類型（在這裡是 string）。  

```typescript
if (!!input) {
  console.log(input);  // input 被收窄為 string
}
```

因此，雖然 Boolean(input) 返回的是一個布林值，但它並沒有改變 input 的實際類型，這就導致了 Narrowing Types失敗。

## 替代方案：自定義判斷函數

如果我們需要更加靈活的 Narrowing Types，可以考慮使用自定義的判斷函數。  
例如，我們可以創建一個 isString 函數來明確檢查某個值是否為字串，這樣就可以正確進行 Narrowing Types。

```typescript
const isString = (value: unknown): value is string => {
  return typeof value === "string";
};

const myFunc = (input: string | null) => {
  if (isString(input)) {
    console.log(input);  // 這裡 TypeScript 收窄為 string
  }
};
```

在這段程式碼中，我們定義了一個 isString 函數，它不僅進行類型判斷，還告訴 TypeScript 當返回 true 時，value 確實是 string。  
這樣的函數可以幫助我們更好地進行 Narrowing Types。  

### 另一個常見的例子：filter(Boolean)

filter(Boolean) 是 JavaScript 中常用的一個語法糖，用來過濾掉 falsy 值（如 null、undefined、0 等）。  
但在 TypeScript 中，這個模式無法進行 Narrowing Types。

```typescript
const arr = [1, 2, 3, null, undefined];

// 結果類型仍然包含 null 和 undefined
const result = arr.filter(Boolean);
```

在這裡，TypeScript 沒有收窄 result 的類型，它仍然認為結果可能包含 null 和 undefined。  
要解決這個問題，我們可以自定義一個過濾函數，來正確處理這些類型。

```typescript
const filterNullOrUndefined = <T>(value: T | null | undefined): value is T => {
  return value !== null && value !== undefined;
};

const arr = [1, 2, 3, null, undefined];

const result = arr.filter(isNotNullOrUndefined);  // 現在類型是 number[]
```

這樣，我們就能確保結果陣列只包含 number，而不再包含 null 和 undefined。

## 小結

雖然 Boolean() 是 JavaScript 中常見的布林判斷工具，但在 TypeScript 中，它無法像其他判斷語句一樣進行 Narrowing Types。  
我們可以透過自定義類型判斷函數，來精確地告知 TypeScript 在特定條件下的類型，從而實現更強大和靈活的類型檢查。

參考

- [Narrowing Types in TypeScript](https://commerce.nearform.com/blog/2022/narrowing-types)
- [Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)

(fin)
