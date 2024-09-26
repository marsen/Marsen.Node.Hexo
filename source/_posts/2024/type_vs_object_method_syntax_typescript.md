---
title: " [學習筆記] 淺談 TypeScript 方法簡寫與物件屬性語法的差異"
date: 2024/09/26 11:14:58
tags:
  - 學習筆記
---

## 前情提要

在 TypeScript 中，我們常見到兩種方法的定義方式：  
**方法簡寫語法(Method Shorthand Syntax) 和 物件屬性語法(Object property syntax)**。  
乍看之下，這兩種語法非常相似，但實際上，Method Shorthand Syntax 在類型檢查上的表現可能會導致潛在的運行時錯誤。  
本篇將討論這個問題，並提供避免這類錯誤的最佳做法。

## 本文

在 TypeScript 中，我們可以用兩種不同的方式定義物件的方法：

Method Shorthand Syntax：

```typescript
interface Animal {
  makeSound(): void;
}
```

Object property syntax：

```typescript
interface Animal {
  makeSound: () => void;
}
```

兩者表面上似乎只是不同的語法選擇，但實際上，它們在類型檢查時有著不同的行為。  
當我們使用 Method Shorthand Syntax 時，TypeScript 的類型檢查會出現雙變性（Bivariance），  
這意味著參數的類型檢查會變得寬鬆，允許接受與定義不完全符合的類型。

問題例子
讓我們看一個新的例子：

```typescript
interface Character {
  attack(character: Character): void;
}

interface Monster extends Character {
  counterattack: () => void;
}

const hero: Character = {
    attack(victim: Monster) {
      // victim do something
    },
};

const goblin: Character = {
    attack() {},
};

hero.attack(goblin); // 編譯時無錯，運行時錯誤！
```

在這個例子中，我們有一個 Character 介面和一個繼承它的 Monster 介面。  
Monster 介面具有一個 counterattack 方法，這表示怪物應該能夠進行反擊。  
接著，我們定義了一個 hero 物件，它可以攻擊任何 Monster 角色並呼叫它的 counterattack 方法。  
然而，我們創建了一個 goblin 物件，這個物件實現了 Character 介面，但並不符合 Monster 介面的要求。  
當我們試圖讓 hero 攻擊 goblin 時，雖然 TypeScript 在編譯時不會報錯，但在運行時會導致錯誤，因為 goblin 並沒有實作 counterattack 方法。  
這是由於參數類型的雙變性（Bivariance）造成的問題，因為 hero.attack 方法的參數類型過於寬鬆，導致運行時出現預期外的行為。

解決方案
為了解決這個問題，應該使用物件屬性語法來定義方法，這樣 TypeScript 會進行更嚴格的類型檢查，並能在編譯時捕捉到類型不匹配的問題。

改寫後的範例：

```typescript

interface Character {
  attack: (character: Character) => void; // 改用 Object property syntax
}

// 這裡會報錯，因為 attack 應該傳入的是 Character
const hero: Character = {
    attack(victim: Monster) {
      // victim do something
    },
};
```

在這個改寫後的範例中，TypeScript 會在編譯時警告我們 attack 方法的參數類型不匹配，從而避免了運行時錯誤。

## 結語

TypeScript 的類型系統非常強大，但也有一些容易被忽略的陷阱。  
雙變性可能看似便利，但它也可能導致運行(Runtime)時的錯誤。  
為了減少這類錯誤的風險，我們應該使用物件屬性語法來定義方法，  
可以讓 TypeScript 進行更加嚴格的類型檢查，從而在開發過程中及早發現問題。

## 參考

- [Method Shorthand Syntax Considered Harmful](https://www.totaltypescript.com/method-shorthand-syntax-considered-harmful)
- [TypeScript Variance](https://webmix.cc/tutorials/typescript/%E9%80%B2%E9%9A%8E%E5%9E%8B%E5%88%A5)

(fin)
