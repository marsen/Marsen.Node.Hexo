---
title: " [學習筆記] TypeScript 的 Immutable Objects 與 Record<T1,T2>
date: 2025/01/31 15:41:28
tags:
  - 學習筆記
---

## 前情提要

最近在看 TypeScript Wizard 的課程，稍作一下紀錄  

```typescript
const user = { name: "John" };
user.age = 24; // ❌ Property 'age' does not exist on type '{ name: string; }'.
```

**這裡 IDE 會在 user.age 上報錯，因為 TypeScript 預設只允許存取「已知」的屬性。**  
**由於 user 只有 name 屬性，TypeScript 會將其推斷為 { name: string } 類型，因此無法直接新增 age。**

## 解法

這時候我們有幾種解法，但要挑選 最符合 TypeScript 精神的做法。

### 明確定義 User 類型

這是最推薦的作法

```typescript
type User = { name: string; age?: number };
const user: User = { name: "John" };
```

### 使用 Record 定義動態物件

如果我們需要 允許物件擴展但仍保持類型安全，可以使用 Record<K, V>：

```typescript
const user: Record<string, string | number> = { name: "John" };
user.age = 24; // ✅ 這次不會報錯！
user.isAdmin = true; // ❌ Type 'boolean' is not assignable to 'string | number'.
```

### 另一種動態的作法，明確定義索引簽名

```typescript

type User = { name: string; [key: string]: string | number };
```

這樣 user 可以擁有 任何鍵（string 類型），但值必須是 string 或 number，避免亂塞其他類型。

## Immutable Objects

反之，如何保證物件的結構不會意外被修改？
我們可以使用 readonly 保持 Immutable Objects

```typescript
type User = Readonly<{ name: string; age: number }>;
const user: User = { name: "John", age: 24 };

user.age = 25; // ❌ Cannot assign to 'age' because it is a read-only property.
```

這種方式確保 user 一旦被賦值，就不能再改變，完全符合 Immutable Objects 的概念。  
另一種方式是使用 Object.freeze() 來讓物件變成真正的 Immutable：  

```typescript
const user = Object.freeze({ name: "John", age: 24 });

user.age = 25; // ❌ TypeError: Cannot assign to read only property 'age'
```

不過要注意，Object.freeze() 只會影響淺層屬性，深層的物件仍然可以變更。

進一步思考，如何選擇？  

- 如果你的物件是固定結構但不可變更，readonly 是最佳選擇。
- 如果需要允許擴展但要控制類型範圍，使用 Record<K, V>。
- 如果想確保物件完全不可變，Object.freeze() 是最安全的選項。

## 參考

- [Techniques for Solving Property Does Not Exist on Type Error](https://www.totaltypescript.com/tutorials/solving-typescript-errors/errors/property-does-not-exist-on-type-error/solution)
- [Record 型別解釋：Record in TypeScript](https://www.typescriptlang.org/docs/handbook/utility-types.html#recordkeys-type)
- [Object.freeze()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)

(fin)
