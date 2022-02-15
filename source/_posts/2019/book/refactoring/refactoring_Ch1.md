---
title: "[閱讀筆記] 重構---改善既有程式的設計，第一章"
date: 2019/03/01 11:17:16
tag:
  - 重構
  - Testing
---

## 為什麼

### 知道自已不知道

上次在公司內部開始進行 [Coding Dojo](/2019/01/30/2019/coding_dojo_in_company/)，在 [FizzBuzz 的 Kata](/2019/02/06/2019/coding_dojo_kata_fizzbuzz_first/) 嚐到了甜頭  
但是下一道題目「Bowling」，卻卡住了。
我們有測試，也通過測試，但是卻寸步難行，  
在重構上我們非常的弱，這裡的重構不是指一次性的全面翻掉，  
而是逐步的、可靠的前進，  
我想習得這樣的技能，因為現場的代碼腐敗的更加嚴重，
如果連 Kata 產生的代碼都不能優化，  
那想對產品指手劃腳只不過是說幹話。

## 閱讀經典:重構---改善即有的程式設計

![重構](https://i.stack.imgur.com/BrLmD.jpg)

這是一本來自 Martin Fowler 的經典書籍，新版已經出了，而且是以 `JavaScript` 作為範例語言。  
不過我手頭上借到的是以 `Java` 作為範例的板本。

### CH1 重構，第一個案例

第一個問題就是我找不到書中說的「線上範例」，  
即使找到我也沒有 `Java` 的開發環境，所以心一橫就開始了改寫成 `C#` 的計劃  
這部份比我想像中的簡單很多，兩個語言是相同類似的，  
[第 1 章，第一個案例](https://github.com/marsen/Marsen.NetCore.Dojo/commit/6e600db029fe2f62df724d0179b708c97a0b3313)

**接下來只要照著書上一步一步作就會...覺得越來越沒 fu …**
為什麼 ???

其實 Martin 大叔在書中有提到「為即將修改的程式建立可靠的測試…畢竟是人，可能會犯錯。所以我需要可靠的測試」。  
沒 fu 的原因就是**我沒加測試**，即使重構了，我也不知道好壞。
沒有反饋是很糟糕的一件事。

### CH1 重構，第一個案例，加上測試

那麼要怎麼加測試呢 ?
書上的案例我分析了一下，其實重構的目標只是一個單純的方法  
會針對不同的情境回傳不同的字串。

簡單的說，我只要讓測試覆蓋這個方法就可以開始重構了，  
我選擇 [dotCover](https://www.jetbrains.com/dotcover/)來檢驗我的覆蓋率。  
選擇的原因很簡單，因為我買了 [ReSharper](https://www.jetbrains.com/resharper/?)，  
其中就包含這個好用的工具。  
試著投資一些金錢在工具上的回報是相當值得的。
~~如果有更好用更便宜的工具也請介紹給我。~~

> OS:課金真的能解決很多人生問題啊(茶)...

最後的結果，我開了一個[分支](https://github.com/marsen/Marsen.NetCore.Dojo/commit/d8d6f463960572af6ffdb3a5612fd00623d0d7e2)包含了 100%的測試覆蓋率，  
這樣就可以開開心心重構了，相信我有測試真得很有感覺。

重構的技法請自行看書，我只稍微作個記錄，有興趣可以 fork 回去玩。

- Extract Method
- Move Method
- Replace Temp with Query
- Replace Type Code with State/Strategy Pattern
- Replace Conditional with Polymorphism
- Self Encapsulate Field

在重構的過程中我儘可能讓步驟小(Baby Step)，看我的 commit 歷程即可知道，但是最好可以自已作作看。
另外有一些心法，也稍作個記錄

- 把一坨爛 Code 抽到獨立的方法之中
- 如果一個類別方法並沒有使用到該類別的資訊
  - 考慮職責，是不是要讓它搬家
  - 提醒自已這是個壞味道
- **拆分職責時，有個方法相依兩個不同的類別的資訊，那應該將方法放在哪裡呢?**(這裡花了點時間理解)
  - 將方法放在未來可能變化較大的類別之中
  - 相依的資訊作為方法參數傳進來
  - 這樣未來有異動就被縮限在這個類別裡面。
- 暫存變數常常會帶來問題(壞味道)
  - 儘可能的把它消除
  - 要考慮效能的問題(書上後面會說。)
- 保持小步調、頻繁測試
  - 使用中繼方法可以縮小重構步調(特別是對 public 的方法)
  - 讓新的 return 值插在舊的 return 之前
  - 測試 ok 就可以刪掉舊 code (有時刪不掉也還是可以運作的)
  - 善用[變異測試](https://blog.marsen.me/2018/03/20/2018/mutation_testing/)
- UML 可以幫助對程式重構前後的理解
- Java 與 C# 對繼承的處理是不同的
  - [Java inheritance vs. C# inheritance](https://stackoverflow.com/questions/13323099/java-inheritance-vs-c-sharp-inheritance)

## 後記

第一章的範例完成後的結果大致如下  
![100%!!!](/images/2019/3/test_cover_100.jpg)  
很帥氣的 100%啊，這樣的 code 測試覆蓋率 100 % 全綠燈，  
而且完成了重構，根本是現場不可能出現的完全體程式碼!!!  
代碼的部份我會放在最後的參考區塊。

![我有沒有可能讓它更好？或是找出他的缺陷呢？](/images/2019/3/cell.jpg)  
下一步，我有沒有可能讓它更好？或是找出他的缺陷呢？

這個時候我想起了[變異測試](https://blog.marsen.me/2018/03/20/2018/mutation_testing/)  
還沒有實作過，來玩看看好了。

首先要選擇測試工具，這裡使用了[Stryker Mutator](https://stryker-mutator.io/)，  
但是注意只能用在 .Net Core 的版本
照著官網安裝完成後執行

```shell
>dotnet stryker
```

跑下去竟然真的找到有存活的變異
![存活的變異](/images/2019/3/run_stryker.jpg)  
這兩個變異存活的原因是類似的，

```csharp
double result = 2;
if (daysRented > 2)
    result += (daysRented - 2) * 1.5;
return result;
```

變異點發生在 `daysRented > 2` 的判斷式之中，  
現有的測試在變異發生(`daysRented >= 2`)時，無法提出警訊，也就是測試上的不足。
不過依現有的邏輯，不論是進入 `if` 進行了加 0 運算，或是直接回傳 result，  
都是等價的(回傳 2 )，目前還沒有想法怎麼強化我的測試，  
希望有先進願意不嗇指點，實務上跟本沒在跑變異測試。

## 後記 2

回歸一下我們當初 Kata 的目的:

- 學習 Pair ，並透過 Pair 彼此學習
- 學習 TDD ，並透過 TDD 學習重構
- 學習 Vim，並提昇開發速度

事情沒有那麼簡單，比如說學習 Vim 的過程中，  
我們的目的是增進開發速度，但是一開始反而會變慢，
一定要刻意的練習才能習得，
你必須擁有以下的能力。

- 打字速度，網路上很多資源，我是使用[Ratatype](https://www.ratatype.com/)作練習
  - 能盲打
  - 指法要正確(特別在特殊符號)
  - 快速切換中英(建議加入英文輸入法用 win + space 之切換過去)
- 英文能力。命名是開發很重要的一課，英文不好看不懂寫得差，命名自然不會好。
- 熟悉工具，特別是你的 IDE 與外掛
  - Visual Studio
  - Resharper
  - OzCode
  - more ..
- Vim
  - Vim Basic 基本功(v、c、i、s、j、k、g、h、l....)
  - VimRc 要學會配置自已的 VimRC，這裡不僅要刻意練習，還要**刻意試錯**找到自已最順的模式

彼此學習方面需要相當的軟技能，  
溝通、尊重、謙虛…;這些一生的功課我就不贅言了。  
Pair Programming 一半是 Pair 一半是 Programming；  
而在進入 Programming 之前請搞懂你**要作什麼**。

同樣的在 TDD 的過程之中，我們沒有事先理好需求，  
沒有想好作好需求分析，隨便選了測試案例就開始進行，  
如果好好分析，是可以歸納出其中的邏輯，  
甚至是理出 test case 的順序。

重要的是過程，但是**我們太在乎結果，以致程式快速的腐敗。**  
甚至到了難以修改的狀態，僅管有測試保護，卻無法重構。

這是很好的一課，特別在這裡記錄一下。

## 參考

- [Ratatype](https://www.ratatype.com/)
- [重構 ─ 改善既有程式的設計， 2/e (Refactoring: Improving The Design of Existing Code)](https://www.tenlong.com.tw/products/9789861547534)
- [dotCover: A Code Coverage Tool for .NET by JetBrains](https://www.jetbrains.com/dotcover/)
- [Java inheritance vs. C# inheritance](https://stackoverflow.com/questions/13323099/java-inheritance-vs-c-sharp-inheritance)
- [marsen/Marsen.NetCore.Dojo](https://github.com/marsen/Marsen.NetCore.Dojo/tree/Refactoring_Improving_The_Design_of_Existing_Code_With_Test)

(fin)
