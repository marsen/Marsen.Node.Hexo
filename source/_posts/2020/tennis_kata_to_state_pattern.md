---
title: " [實作筆記] Tennis KATA 與 State Pattern "
date: 2020/07/21 08:16:27
tag:
  - TDD
  - Unit Testing
  - 實作筆記
  - OOP
  - Design Pattern
---

## 前情提要

Tennis Kata 是我最常練習的一個題目，  
就我個人而言，這個題目源起 91 大的極速開發，  
目前最快只有在 17 分左右，使用 Rider with Mac 的話可能還會再慢一些。  
我很熟悉，所以很少作需求分析，Test Case 也大多有即定的寫法。  
手動得很快，腦卻不怎麼動了，明明這是一個相當經典的題目，  
不過我卻被定錨了。

今年 5 月上了「測試驅動開發與持續重構」，  
第一天 91 大也有透過 Tennis Kata 展示了一下火力，  
那個時候又有提到可以使用 State Pattern 來實作，  
最近工作上又恰巧有使用到 State Pattern。  
於是我便決定要試著用 State Pattern 來進行 Tennis Kata 。

有兩種方法，一種是無到有的 Kata，  
一種是將現有 Tennis Production Code，  
透過重構轉換成 State Pattern，  
這次我選擇從無到有。

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
這樣作是為了符合 Tennis 的規則。

### Test Cases

- LoveAll
  - 產生 Context 類別與 Score 方法
  - 產生 LoveAll State
  - 產生 IState 介面，包含 Score 方法，讓 LoveAllState 實作 IState 介面
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
這次的 TDD 仍然不算順暢成功，設計階段就可以在腦中模擬的問題，
我拖到了開發階段，雖然後來收斂成 `NormalState` 時頗有一回事，
但如果在實務上，這段不確定且發散的時間仍然是太長。

## 第三次成功，仍然不足夠

參考第二次所作的 State Diagram。  
可以看到缺少了 Normal to Normal 的線條。  
第三次將它補上了。
![Final State](/images/2020/7/tennis_kata_to_state_pattern_03.jpg)

| States    | Sample          | Next States              |
| --------- | --------------- | ------------------------ |
| Same      | 0-0,1-1,2-2     | Normal                   |
| Normal    | 0-1,0-2,1-2,1-3 | Same、Normal、Deuce、Win |
| Deuce     | 3-3,4-4,5-5     | Advantage                |
| Advantage | 3-4,5-6         | Deuce、Win               |
| Win       | 5-3,5-7         |                          |

參考上表製作測試案例，
這裡我想強調的是狀態改變的動線，  
狀態由 `SameState` 開始。

簡單筆記一下測試與重構的幾個亮點
完整的 commit 可以從 [d792b2e](https://github.com/marsen/Marsen.NetCore.Dojo/commits/Kata/TennisWithStatePattern2?before=af1303f38d61abc0dba0a965e5dfadf55bc08ccd+105)開始看

### Highlight Test Cases

測試的案例的設計會依 State Diagram 的箭頭來設計，  
也就是狀態的改變，**初始狀態為 `SameState` 比分為 0 - 0** ，  
並將 Design Pattern 作為指引重構出 `SameState` 。

**第二個測試案例**也很簡單，  
因為 `SameState` 只會往 `NormalState` 移動，  
所以只要使用 1 - 0 或是 0 - 1 這個案例，我就可以建立出 `NormalState` 類別。
並且可以觀察到兩個 State 的共通性，這個時候就會重構出 `IState` 介面。

一樣看圖開發，  
`NormalState` 是最為複雜的一個狀態，他的狀態可能為

- 保持原樣 : `NormalState`
- 退回平手 : `SameState`
- 進入決勝 : `DeuceState`
- 直接獲勝 : `WinState`

而剩下的狀態都算是相當簡單，  
`WinState` 的狀態不會再改變，  
`DeuceState` 只會往 `AdvState` 狀態前進，
`AdvState` 是相對複雜的狀態，可能變成 `WinState` 也可能降回 `DeuceState` ，
與 `NormalState` 並沒有直接的關聯路徑，所以在案例設計上，應該放比較後面。

**第三個案例**，我會讓 `NormalState` 變回 `SameState`，除了可以完成一條狀態改變的路徑外，  
好處是我不用新增類別，作到最小異動。
一開始我的邏輯會放在 GameContext 之中，但是不論是依循著 Design Pattern 的設計，  
或是單純考量職責，很自然地將這裡的邏輯移至 `NormalState` 之中。
**這裡案例會選用 1-1** 。

但是要將邏輯移到 `NormalState` 之中時，會面臨一個問題。  
`NormalState` 之中，為了判斷是否會回到 `SameState`，
需要知道 GameContext 的資訊 ( ServerPoint 與 ReceiverPoint )，  
依循 Design Pattern 的指引，  
將會在 `IState` 之中建立 SetContext 方法。
儘管 `SameState` 其實不用 ServerPoint 或 ReceiverPoint 的資訊。
但是因為我們相依於介面 `IState` 之上，導致 `NormalState` 或 `SameState` 都要實作 SetContext 方法。

再進一步，SetContext 在不同的 State 實作上是不會有變化的，  
所以我會轉換 `IState` 為抽像類別 `State`。
而我也需要在比賽初始時呼蹈後 SetContext 並將初始狀態設為 `SameState`

**案例 4、5、6 我選擇了 0-1 、0-2 、0-3 等案例**，  
這裡實作的是 Normal to Normal 的狀態改變(其實是沒變)，透過 Dictionary 消除 if 的手法就略過不提。  
但是也許下次我不會急著完成 `NormalState` 而是先完成 `WinState` 與 `DeuceState` 。

**接下來讓 3-3 這個案例，帶我們走到 `DeuceState`**，
稍微繞了一點路作 4-4 這個案例，雖然一樣是 `DeuceState`，  
我一開始的設計案例上並沒有考量清楚，從 3-3 走到 4-4 必須經過 `AdvState`，  
下次應該先選擇 **3-4 或 4-3 的案例**，  
再透過 **4-4 的案例**實作 `AdvState` 回到 `DeuceState` 的這條路徑。  
最後再將透過**案例 5-3 讓 `AdvState` 走到 `WinState`** 與 **案例 4-1 讓 `NormalState` 走到 Win** 就可以結束主要的流程了。

收官的部份就是簡單的重構，與補足一些特殊比分的案例，  
特別要注意的是 `NormalState` 的 Score 仍然有許多的 if 讓我想重構移除，
目前暫時沒有想法。
此外，實際上 Tennis 會特別重視得分的順序，  
比如說 **100-100 Deuce 這種極端案例**，交互得分的情況應忠實呈現在測試之中。
所以在寫測試時，就要特別注意得分的順序性。

## 後記

這次還是有用到一些常用的重構套路，  
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
同樣的手段可以放在 `ReceiverScore` 方法再重構一次。

實務上 Design Pattern 本來就應該星月交輝，而非千里獨行。
在學習 Design Pattern 的路上，不是硬套，而是找出適用場景。

也是我比較建議的作法，透過重構自然走向 Pattern，  
透過限制改變來提昇品質，首先要有測試保護，再找尋套路或壞味道重構，  
最後讓 Design Pattern 成為指引，讓代碼自動躍然而上。

## 參考

- [State Pattern](https://refactoring.guru/design-patterns/state)
- [Template Method Pattern](https://refactoring.guru/design-patterns/template-method)
- [Tennis Rules](https://www.rulesofsport.com/sports/tennis.html)
- [SOLID 五則皆變](http://teddy-chen-tw.blogspot.com/2014/04/solid.html)

(fin)
