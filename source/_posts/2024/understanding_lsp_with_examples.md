---
title: "[學習筆記] 里氏替換原則(Liskov Substitution Principle, LSP)"
date: 2024/11/25 11:39:00
tags:
  - 學習筆記
---

## 前言

OOP 中的五大原則之一---里氏替換原則,開發 OOP 的工程師應該或多或少都有聽過,  
最近與同事討論後有新的體悟,特別記錄一下  

```text
若對某個型別 T 的物件 𝑥 能證明具有某個性質 𝑞(𝑥),那麼對於 T 的子型別 S 的物件 𝑦 同樣應該滿足 𝑞(𝑦)。
```

```text
If a property 𝑞(𝑥) can be proven for objects 𝑥 of type 𝑇,
then 𝑞(𝑦) should also hold true for objects 𝑦 of type 𝑆, where 𝑆 is a subtype of 𝑇.
```

里氏替換原則(Liskov Substitution Principle, LSP)詳解  
定義：  
里氏替換原則由 Barbara Liskov 在 1987 年提出,  
是 SOLID 原則中的 "L"。它強調 子類(Subclass) 應該能夠替換其 父類(Base Class) 而不影響程式的正確性或行為。  

核心概念：  
"任何使用父類的地方,都應該能夠使用其子類,而不會影響系統的功能。"  

## 主要原則

子類應該擁有父類的所有行為特性。  
**子類不應改變父類方法的預期行為。**  
子類可以擴展父類的功能,但不能削弱或改變父類的功能。  

違反 LSP 的常見問題：  
替換後引發錯誤： 子類**重寫**某個方法,導致使用者無法正常使用原本的功能。  
返回值不一致： 子類方法返回值與父類期望的類型不同。  
拋出未預期的異常： 子類新增或拋出父類未預期的異常。  

## 舉例說明,錯誤示例

```typescript
class Bird {
  fly(): void {
    console.log('Flying...');
  }
}

class Penguin extends Bird {
  // Penguins can't fly!
  fly(): void {
    throw new Error("Penguins can't fly!");
  }
}

function makeBirdFly(bird: Bird) {
  bird.fly();
}

// 當傳入 Penguin 時,會引發錯誤
const penguin = new Penguin();
makeBirdFly(penguin);  // Throws an error!
```

違反原因：  

在 makeBirdFly 函數中,傳入 Penguin 子類後,原本預期的行為(鳥會飛）被破壞,導致異常發生,違反 LSP。  

正確示例：  

```typescript
class Bird {
  move(): void {
    console.log('Moving...');
  }
}

class FlyingBird extends Bird {
  fly(): void {
    console.log('Flying...');
  }
}

class Penguin extends Bird {
  swim(): void {
    console.log('Swimming...');
  }
}

function makeBirdMove(bird: Bird) {
  bird.move();
}

const penguin = new Penguin();
makeBirdMove(penguin);  // 正常運行,無異常
```

## 修正方式

將 Bird 分解為更細的抽象類別(如 FlyingBird)。  
確保所有子類的行為符合其父類的行為約束。  
實踐 LSP 的要點：

- 正確使用繼承： 只有在 "是某種(is a)" 的關係中使用繼承;例如:Dog 是 Animal 的一種(我們都不認為這是一個好例子)。
- 避免強制行為： 子類不應強迫覆蓋父類的方法,除非行為一致。
- 使用介面： 如果子類具有不同行為,可使用多個接口來定義不同能力。

總結：
LSP 促使開發者在繼承和多型設計中保持一致性和正確性,保證系統的可擴展性和穩定性。  
遵循 LSP 可以避免許多因繼承導致的潛在問題,使代碼更健壯且易於維護。

## 我們的一些關鍵 Q&A

Q.LSP 的意思是不要用 override ?  
A.
不是的,LSP(里氏替換原則)並不是要求完全避免使用 override,  
而是要正確使用 override。核心在於: **子類覆寫父類方法時,不能破壞父類的行為預期。**

Q. 要正確使用 override 的原則是什麼？  
A. 子類應維持父類方法的語義一致性  
覆寫後的行為應與父類保持一致,並且在替換後不會引發錯誤。

Q. 有更具體的說明嗎 ?  
A. 不要改變方法的輸入輸出參數規格  
遵循開放封閉原則(Open/Closed Principle),開放擴展,封閉修改。  
方法的參數類型不能比父類方法的參數更具限制性。  
方法的返回類型不能比父類方法的返回類型更具擴展性。  

Q.
以你的例子來說Penguin 的確不會 fly , 所以應該拋出錯誤  
但是這樣就違反了 LSP ,所以我們的建議是將 fly 再抽出成為介面?  
A.
沒錯!這是關鍵點了。解決這種情況的最佳方式是使用介面(Interface)來抽象行為,而不是在基類中定義所有可能不適用於子類的行為。

子類應擴展父類的功能,而不是修改父類的行為。

### 正確的覆寫示例

```typescript
class Animal {
  makeSound(): string {
    return "Some sound";
  }
}

class Dog extends Animal {
  override makeSound(): string {
    return "Bark";
  }
}

function playSound(animal: Animal) {
  console.log(animal.makeSound());
}

const dog = new Dog();
playSound(dog);  // Output: "Bark" (符合父類預期)
```

說明：
Dog 覆寫了 makeSound(),但返回類型和行為保持一致,替換 Animal 後,系統的正確性不受影響。

### 違反 LSP 的覆寫示例

```typescript
class Rectangle {
  constructor(protected width: number, protected height: number) {}

  setWidth(width: number) {
    this.width = width;
  }

  setHeight(height: number) {
    this.height = height;
  }

  getArea(): number {
    return this.width * this.height;
  }
}

class Square extends Rectangle {
  // 覆寫後改變了行為,違反 LSP
  setWidth(width: number) {
    this.width = width;
    this.height = width;  // 強制將高度設為相等
  }

  setHeight(height: number) {
    this.height = height;
    this.width = height;  // 強制將寬度設為相等
  }
}

function testRectangle(rect: Rectangle) {
  rect.setWidth(5);
  rect.setHeight(10);
  console.log(rect.getArea());  // 預期：50
}

const square = new Square(0, 0);
testRectangle(square);  // Output: 100 (違反父類預期)
```

說明：
Square 覆寫了 setWidth 和 setHeight 方法,但改變了 Rectangle 的行為。  
傳入 Square 後,計算面積的結果與 Rectangle 的預期不同,這破壞了 LSP。  

兩個例子的差異在於
Dog 的覆寫行為符合父類預期,Square 改變父類邏輯(強制高寬相等）,破壞原有功能,違反 LSP。

## 總結

可以使用 override,但要謹慎處理。
確保子類的行為符合父類的預期,不會引入不一致或異常行為。
遵守 LSP 能保證系統在多型使用時的穩定性和可預測性。

對「少用繼承多用組合」的體悟多了一層，如果發現繼承了父類的方法，  
但是修改了行為(拋錯誤、改變本來沒有改動到的屬性、長出新的邏輯分更支…)，這就是一種壞味道。　　
大部份應該都可以透過介面排除這個問題。

(fin)
