---
title: " [翻譯] TypeScript 寫給 C#/Java 工程師"
date: 2023/04/09 16:50:10
tags:
  - 實作筆記
---

## 前言

工作上有開發前端專案的需求，主流使用 JavaScript。  
而聽說了一些強型別語言的優點，加上我有開發 C# 的經驗，我常常改用 TypeScript,  
但實際上確常常覺得反而更笨重了，覺得開發上不太順暢。

### 一個是 node_module 的問題

相依性的管理十分麻煩，常常有新版的模組更新，但其相依的模組卻尚未更新。
運氣好等幾周就會有更新，運氣不好是模組的開發者已經不在維護，目前我的處理方式是使用 ncu ，  
但也有其極限，而在其上花費的大量時間，反而拖累開發的速度。

### TypeScript

另一個麻煩的點是使用 TypeScript ，雖然我有 C# 的經驗，也熟悉 JavaScript，  
但是常常覺得開發仍然不順，直到閱讀官方的這篇文章，才稍解一些疑惑。  
並稍作記錄如下。

## 翻譯

本文出處為[TypeScript for Java/C# Programmers](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes-oop.html)  
我不會逐字翻譯，只針對我認為重要的觀念作筆記。

### Class 的反思

在 C#/Java Class 是程式的基本單位，這類的程式語言我們稱之為 mandatory OOP  
而在 TypeScript/JavaScript 當中 Function 才是程式語言的基本單位。  
Function 可以自由的存在，而不需要寄生在 Class 之中。  
這帶來了靈活性的優點，在思考 TypeScript 時，應以此作為考量。  
因此，C#和 Java 的某些結構，如 Singleton 和 Static Class 在 TypeScript 中是不必要的。

### Type 的反思

Nominal Reified Type Systems vs Structural System
C#/Java 使用 Nominal Reified Type Systems ，  
這表示 C#/Java 程式中所使用的值或物件，一定會是 null 或基本型別(int、string、boolean 等…)或具體的 Class。  
而在 TypeScript 中，類別只是個集合。
你可以這樣描述一個值同時可能是 string 或 number

```typescript
let stringNumber: string | number;
stringNumber = 1;
stringNumber = "123";
```

同時在作型別的推導的時候， C#/Java 會有具體的型別，而 TypeScript 會使用結構作推導，  
參考下面的例子，我們沒有給 obj 具體的型別，但是在結構上符合 Pointlike 與 Named 所擁有的屬性，  
使得呼叫方法時的型別推導在 TypeScript 是合法的。

```typescript
interface Pointlike {
  x: number;
  y: number;
}
interface Named {
  name: string;
}

function logPoint(point: Pointlike) {
  console.log("x = " + point.x + ", y = " + point.y);
}

function logName(x: Named) {
  console.log("Hello, " + x.name);
}

const obj = {
  x: 0,
  y: 0,
  name: "Origin",
};

logPoint(obj);
logName(obj);
```

這樣的設計在 runtime 的時候，我們無法具體的知道型別是什麼，  
下面的概念在 TypeScript 將不可行，  
即使用 JavaScript 的 typeof 或 instanceof 你也只會拿到 "object"，而非具體的型別

```csharp
// C#
static void LogType<T>() {
    Console.WriteLine(typeof(T).Name)；
}
```

即使如此，這樣的設計在編譯時期的檢查在實務上是足夠的，  
另外兩個例子與長時間開發 C#/Java 的概念上可能會有所不同，

#### Empty Type

Typescript 允許設計無屬性的 Type，而基於 Structural System  
你可以建立一個方法傳遞任何物件進去(而不是用 any 或 object 去定義傳入值)

```Typescript
class Empty {}

function fn(arg: Empty) {
  // do something?
}

// No error, but this isn't an 'Empty' ?
fn({ k: 10 });
```

#### Identical Types

下面的程式不會發生錯誤，但是可能會讓 C#/Java 的開發者有點意外

```Typescript
class Car {
  drive() {
    // hit the gas
  }
}
class Golfer {
  drive() {
    // hit the ball far
  }
}
// No error?
let w: Car = new Golfer();
```

一樣基於 Structural System 的設計，有相同的屬性與方法簽章的兩個不同物件，  
是允許這樣子的行為，而以官方的看法來說，實務上並不太容易發生這種情形(兩個不同的模型，卻有相同的方法、屬性等…)。

## 小結

以下是 C#/Java 與 TypeScript 的相同和不同之處：

相同之處：

- C#/Java 和 TypeScript 都是物件導向編程語言。
- C#/Java 和 TypeScript 都使用類（class）和物件（object）的概念。
- C#/Java 和 TypeScript 都有靜態類型系統。

不同之處：

- C#/Java 的類型系統基於類型聲明，而 TypeScript 的類型系統基於屬性的兼容性。
- C#/Java 的類型在 runtime 是存在的，而 TypeScript 的類型在 runtime 時是不存在的。
- C#/Java 類型之間的關係通過繼承關係或共同實現的接口來定義。而在 TypeScript 中，類型之間的關係是通過屬性的兼容性來定義。

## 心得

> If it looks like a duck, swims like a duck, and quacks like a duck, then it probably is a duck.

TypeScript 的類型設計讓我想起了鴨子測試，也讓我想起 golang 對 interface 的處理方式。
甚至想起一些對物種起源的探討，先有分類還是先有特性呢 ? 真實的世界分類反而是人類主觀強加上去的。
TypeScript 這樣的設計似乎比起強制性的類別設計(C#/Java)，更貼近真實的世界。

## 參考

- [TypeScript for Java/C# Programmers](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes-oop.html)
- [[實作筆記] Hexo CI 自動執行 ncu -u 更新相依套件](https://blog.marsen.me/2022/09/28/2022/Hexo_CI_auto_npm_/)

(fin)
