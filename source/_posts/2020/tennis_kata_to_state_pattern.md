---
title: "[實作筆記] Tennis KATA 與 State Pattern "
date: 2020/07/21 08:16:27
tag:
    - TDD
    - Unit Testing
    - 實作筆記
---

## 前情提要

Tennis Kata 是我最常練習的一個題目，  
就我個人而言，這個題目源起 91 大的極速開發，  
目前最快只有在 17 分左右，使用 Rider with Mac 的話可能還會再慢一些。  
我很熟悉，所以很少作需求分析，Test Case 也大多有即定的寫法。  
手動得很快，腦卻不怎麼動了，明明這是一個相當經典的題目，  
不過我確被定錨了。

今年 5 月上了「測試驅動開發與持續重構」，  
第一天也有透過 Tennis Kata 展示了一下火力，  
那個時候講師有提到可以使用 State Pattern 來實作，  
最近工作上又恰巧有使用到 State Pattern。  
於是我便決定要試著用 State Pattern 來進行 Tennis Kata 。  

有兩種方法，一種是無到有的 Kata，  
一種是將現有 Tennis Production Code，  
透過重構轉換成 State Pattern，  

## 第一次失敗

總歸來說，需求分析作得不夠徹底，  
Test Case 設計不良，所以很難自然而然的讓 State 產生

![第一次失敗](/images/2020/7/tennis_kata_to_state_pattern_00.jpg)

上圖是我第一次畫的 State ，  
現在回過頭來想想，圖型上其實可以很明顯看出重複的壞味道。  
但是我當下完全沒有「覺察」，明明是想要消除 if else，  
卻在 State 裡面產生了大量的 if else。  

## 第二次不成功

總而言之是作完了，但是不是很順暢。
![第二次不成功](/images/2020/7/tennis_kata_to_state_pattern_01.jpg)

如上圖，我蠻粗暴的將所有比分轉換成可能的 State，  
一樣我沒有注意到重複，但是比較起第一次的失敗，  
這次的狀態機是將所有可能的狀態攤平，  
當初會這樣作是為了符合 Tennis 的規則。

### Test Cases

- LoveAll
  - 產生 Context 類別與 Score 方法
  - 產生 LoveAll State
  - 產生 IState 介面，包含 Score方法，讓 LoveAllState 實作 IState 介面
- FifteenLove
  - 產生 ServerScore 方法
  - 產生 FifteenLove State
  - 產生 SetContext 方法
  - 產生 ChangeState 方法
  - 轉換 IState 介面成為 State 抽象類別
- LoveFifteen
  - 產生 ReceiverScore 方法
  - 產生 LoveFifteen State
- LoveThirty
  - 產生 LoveThirty State
- LoveForty
  - 產生 LoveForty State
- FifteenAll
  - 產生 FifteenAll State
略…
- **覺察重複，重構**
  - 產生 NormalState
  - 產生 ServerPoint  
  - 產生 ReceiverPoint  
  - 使用 Dictionary 消除 if else
  - 產生 SameState
- Deuce
  - 產生 DeuceState
以下略…

![Final State](/images/2020/7/tennis_kata_to_state_pattern_02.jpg)

最後的狀態會如上圖，當大量的 State 產生之時，心裡真的有點慌慌的，  
這也是我為什麼覺得這次的 TDD 仍然不算順暢成功，  
雖然後來收練成 `NormalState` 時真得很爽。

另外這次還是有用到一些套路，  
比如說，用 Dictionary 消除 if else 的手段。
另一個則是 `Template Method Pattern`，
讓我們看看以下的 commit  

- [e8ddf8](https://github.com/marsen/Marsen.NetCore.Dojo/commit/e8ddf89fdee94d6a82115f4449e213a4874269f8)
- [145d3c](https://github.com/marsen/Marsen.NetCore.Dojo/commit/145d3cb408ed5b39a729c4a9b22fb6744b62c48f)

以 `ServerScore` 方法為例，
本來是在定義在 Abstract Class State 之中的抽像方法，  
由各個 State (Normal、Same、Deuce、Adv 與 Win)實作，
但是我們可以明顯發現， `Context.ServerPoint++` 是重複的，  
而 `ChangeState` 才是真正抽像的地方，  
所以我們可以在 Abstract Class State 加入以下的方法與抽像方法，  

```csharp
public void ServerScore()
{
    Context.ServerPoint++;
    ChangeState();
}

protected abstract void ChangeState();
```

這不就恰巧是 `Template Method Pattern` 嗎 ?  
實務上 Design Pattern 本來就應該星月交輝，而非千里獨行。
在學習 Design Pattern 的路上，最難且最重要的不是使用，而是找出適用場景。  

也是我比較建議的作法，透過重構自然走向 Pattern，  
透過限制改變來提昇品質，首先要有測試保護，再找尋場景或壞味道重構，  
最後讓 Design Pattern 成為指引，讓代碼自動躍然而上。

## 參考

- [State Pattern](https://refactoring.guru/design-patterns/state)
- [Template Method Pattern](https://refactoring.guru/design-patterns/template-method)
- [Tennis Rules](https://www.rulesofsport.com/sports/tennis.html)
- [SOLID 五則皆變](http://teddy-chen-tw.blogspot.com/2014/04/solid.html)

(fin)
