---
title: " [學習筆記] TypeScript 字串建議的小技巧"
date: 2024/09/30 14:00:14
tags:
  - 學習筆記
---

## 前情提要

最近在寫一個 Hero component，需求是讓使用者能指定英雄的種族。  
我們的設計有一些既定的種族，例如 human 和 demon，同時也希望讓使用者能輸入任何自定義的種族名稱。  
最初的想法是用以下的定義：  

```typescript
type Race = 'human' | 'demon' | string;
```

並在 Hero 的 props 中使用這個型別：  

```ts
export type HeroProps = {
  name: string;
  race: Race;
}

// component 大概如下
export const Hero = ({ race,name }: HeroProps) => {
  return (
    <h1>Hero: {name} is {race}</h1>
  );
};
```

這樣一來，使用者可以像這樣使用：  

```ts
// src/components/HeroDisplay.tsx

import React from 'react';
import { Hero } from '../components/hero'; // 引入 Hero 型別

const HeroDisplay = () => {
  return (
    <div>
      <Hero name="alice" race="human" />
      <Hero name="mark" race="demon" />
      <!--more heros -->
    </div>
  );
};

export default HeroDisplay;
```

一切看似沒問題，但問題是——在使用 Hero component 時，TypeScript 並不會自動給出 human 或 demon 這樣的建議。  

**既然我們希望能提供建議，該怎麼解決這個問題呢？**  

## 實作記錄

解決方法看起來有些奇怪，我們可以透過將字串類型與一個空的物件相交來達成目標：  

```ts
type Race = 'human' | 'demon' | (string & {});
```

這樣一來，在使用 Hero component 時，TypeScript 就會正確地給出 primary 和 secondary 的建議。  
為什麼這會起作用？這其實是 TypeScript 編譯器的一個小「怪癖」。  
當你把字串常值類型（例如 "human"）與字串類型（string）進行聯集時，  
TypeScript 會急切地將其轉換為單純的 string，因此在 Hover 時會看到類似這樣的結果：  

```ts
type Race = 'human' | 'demon' | string ;
// Hover 時會顯示：
type Race = string
```

換句話說，TypeScript 在使用前就忘記了 human 和 demon。  
而透過與空物件 & {} 進行相交，我們能「欺騙」 TypeScript，讓它在更長時間內保留這些字串常值類型。  

```ts
type Race = 'human' | 'demon' | (string & {});
// Hover 時會顯示：
type Race = 'human' | 'demon' | (string & {});
```

這樣，我們在使用 Race 型別時，TypeScript 就能記得 human 和 demon，並給出對應的建議。

值得注意的是，string & {} 實際上和單純的 string 是相同的類型，因此不會影響我們傳入的任何字串：

```ts
  <Hero name="alice" race="human" />
  <Hero name="mark" race="demon" />
```

這感覺像是在利用 TypeScript 的漏洞。  
不過，TypeScript 團隊其實是知道這個「技巧」的，他們甚至針對這種情況進行測試。  
或許將來，TypeScript 會原生支援這樣的功能，但在那之前，這仍是一個實用的小技巧。  

[範例 Code](https://gist.github.com/marsen/9b6f041177d736f36c42a372ff684f66)

## 小結

總結來說，當你想允許使用者輸入任意字串但又想提供已知字串常值的自動補全建議時，可以考慮使用 `string & {}` 這個技巧：  

它防止 TypeScript 過早將 string | "literal" 合併成單純的 string。  
實際使用時行為與 string 一樣，但會多提供自動補全功能。  
這或許不是最正式的解法，但目前仍是一個可以信賴的方式。  
也許未來 TypeScript 能夠原生解決這個問題，但在那之前，這個小技巧可以為開發帶來便利。  

(fin)
