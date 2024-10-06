---
title: " [學習筆記] Omit 與 Pick 的 Distributive 版本：解決 TypeScript Utility Types 的陷阱"
date: 2024/10/05 15:58:30
tags:
  - 學習筆記
---

## 前情提要

在 TypeScript 中，Omit 和 Pick 是廣受喜愛的 Utility Types，  
它們允許你從現有型別中排除或選擇特定的屬性來創建新型別。  
參考我之前的[文章](https://blog.marsen.me/2022/09/12/2022/TypeScript_Omit/)

## 本文

### 基礎示範：Omit 的使用

我們先來看個簡單的例子，假設我們有一個 Game 型別，其中包含 id、name 和 price 三個屬性。

```typescript
type Game = {
  id: string;    // 遊戲的唯一識別碼
  name: string;  // 遊戲名稱
  price: number; // 遊戲價格
};

type GameWithoutIdentity = Omit<Game, "id">;

const game: GameWithoutIdentity = {
  //id: "1", // ❗ 編譯錯誤：'price' 不存在於 'GameWithoutIdentity' 型別中
  name: "The Legend of Zelda", 
  price: 59.99, 
};

```

Omit 可以幫助我們排除 id 屬性並創建新型別 GameWithoutIdentity  
這在單一型別中運行良好，但當我們引入 Union Types 時，就有一些細節值得討論。

## 問題：Union Types 中的 Omit 行為

假設我們有三個型別：Game、VideoGame 和 PCGame。
它們的 id 和 name 屬性相同，但每個型別都有其獨特的屬性。

```typescript

type Game = {
  id: string;        // 遊戲的唯一識別碼
  name: string;      // 遊戲名稱
};

type VideoGame = {
  id: string;              // 遊戲的唯一識別碼
  name: string;            // 遊戲名稱
  platform: string;        // 遊戲平台
  genre: string;           // 遊戲類型
};

type PCGame = {
  id: string;                    // 遊戲的唯一識別碼
  name: string;                  // 遊戲名稱
  systemRequirements: string;    // 系統需求
  hasDLC: boolean;              // 是否有 DLC
};

```

當我們將這三個型別聯合起來形成 GameProduct 並嘗試使用 Omit 排除 id 時，結果卻不是我們預期的。

```typescript
type GameProduct = Game | VideoGame | PCGame;
type GameProduct = Omit<GameProduct, "id">;
```

你可能期望 GameProductWithoutId 是三個型別排除 id 屬性的 Union Type，但實際上，我們只得到了這樣的結構：

```typescript
type GameProduct = {
  name: string;
}
```

這表示 `Omit` 在處理 `Union Types` 時，並沒有對每個聯合成員單獨操作，而是將它們合併成了一個結構。

### 原因分析

這種行為的根源在於，`Omit` 和 `Pick` 不是 `Distributive` 的 `Utility Types`。  
它們不會針對每個 `Union Type` 成員進行個別操作，而是將 `Union Type` 視為一個整體來操作。  
因此，當我們排除 id 屬性時，它無法處理每個成員型別中的不同屬性。

這與其他工具型別如 `Partial` 和 `Required` 不同，這些工具型別可以正確地處理 Union Types，並在每個成員上應用。

```typescript
type PartialGameProduct = Partial<GameProduct>;
// 正確地給出了聯合型別的部分化版本
type PartialGameProduct = Partial<Game> | Partial<VideoGame> | Partial<PCGame>;
```

### 解決方案：Distributive Omit 與 Distributive Pick

要解決這個問題，我們可以定義一個 Distributive 的 Omit，這個版本會針對 Union Type 的每個成員進行操作。

```typescript
type DistributiveOmit<T, K extends PropertyKey> = T extends any
  ? Omit<T, K>
  : never;
```

使用 DistributiveOmit 後，我們可以正確地得到想要的結果：

```typescript
type GameProductWithoutId = DistributiveOmit<GameProduct, "id">;
// Hover 正確地給出了聯合型別的必填化版本
//type GameProductWithoutId = Omit<Game, "id"> | Omit<VideoGame, "id"> | Omit<PCGame, "id">
```

這將生成以下結構：

```typescript
// 所以
type GameProductWithoutId = 
  { name: string } | 
  { name: string; platform: string; genre: string } | 
  { name: string; systemRequirements: string; hasDLC: boolean };
```

現在，GameProductWithoutId 正確地成為了每個型別的 Union Type，並且成功地排除了 id 屬性。

### Distributive Pick

類似的，我們也可以定義一個 Distributive 的 Pick：

```typescript
type DistributivePick<T, K extends keyof T> = T extends any
  ? Pick<T, K>
  : never;
```

這個 `Distributive` 版本的 `Pick` 確保了你所選擇的屬性實際存在於你正在操作的型別中，與內建的 `Pick` 行為一致。

## 總結

`Omit` 和 `Pick` 雖然在單一型別中表現良好，但在 `Union Types` 中，  
它們並不是 `Distributive` 的，這可能導致意想不到的結果。
我們可以創建 `DistributiveOmit` 和 `DistributivePick`，  
使它們能夠針對每個 `Union Type` 成員單獨進行操作，從而獲得更為預期的結果。
如果你在專案中遇到這類問題，記得可以考慮使用自定義的 `Distributive` 版本來處理 `Union Types`，這樣可以避免踩雷！

(fin)
