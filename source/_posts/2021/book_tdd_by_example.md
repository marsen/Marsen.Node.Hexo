---
title: " [實作筆記] Kent Beck 的測試驅動開發 Ch12~Ch17"
date: 2021/05/24 20:55:16
---

## 前情提要

最近疫情昇溫,公司開始 Work From Home,
少了通勤時間,少了晚上的聚會或運動課,
每天大概多了３～５個小時,趁現在多看點書囉,
「Kent Beck 的測試驅動開發」是一本 TDD 的經典,
加上是由 91 Chen 所翻譯,所以一出版我就買了,
不過一直到最近才有機會看,危機或許是就是轉機,趁現在補充一下自已的技能點數吧。

## 本書介紹

這本書分為三大部份,分別是第一部份貨幣範例,  
第二部份  xUnit 範例, 最後一部份是介紹 TDD 模式。
本篇文章著重在第一部份  Ch12 與  Ch13  章的部份,  
這裡有大量的實作, 書中作者是用 Java 實現的, 我是試著用 C# 與 xUnit 去實作。

這裡作者的思路對我來說實在是難以理解,  
重構步驟更是讓我消化不良,  
想了一下, 回歸初衷用我自已的步伐
試試看能不能「Driven」一些產品代碼出來。
特以此文記錄。

## 我在哪裡

我已經有了一些代碼, 並且開始了加法的計算
幣別或許是一個問題, 但在我們（在書中）先簡化這個問題, 只作美元的加法。
測試案例如下

```csharp
    [Fact]
    public void testSimpleAddition()
    {
        Money five = Money.dollar(5);
        IExpression sum = five.plus(five);
        Bank bank = new Bank();
        Money reduce = bank.reduce(sum,"USD");
        Assert.Equal(Money.dollar(10), reduce);
    }
```

five 是個簡單的 Money 物件，代表 5 美元。
這個測試案例存在一個運算式(IExpression)的隱喻,  
這個隱喻我可真得想不出來, 作者也說他是經過 20 次以上的練習才有這樣的神來一筆,  
我先接受這點繼續作下去

> 在書中我比較能接受是錢包裡面有很多國家錢幣的概念
> 不過我可能還是會用一個 List 丟給計算器，而不是透過運算式
> 運算式我會想像成我有 5 美元紙鈔跟  5 元法郎(現在先簡化成美元）‘five.plus(five);‘
> 然後請銀行依匯率（目前暫時沒有換匯的需求）算錢給我 ‘bank.reduce(sum,"USD");‘

第一個問題是, 雖然測試綠燈, 不過其實是假的
Hard Code 寫死回傳 10 美元, `return Money.dollar(10);`
所以這裡開始會跟書上的發展有些不同, 但應該是殊途同歸才對

```csharp
    public class Bank
    {
        public Money reduce(IExpression sum, string usd)
        {
            return Money.dollar(10);
        }
    }
```

### [Step 1](https://github.com/marsen/Marsen.NetCore.Dojo/pull/36/commits/6947759e40bfd6c35696a356077366b8698ba3a8)

參數名命怪怪的先改一下

```csharp
public Money reduce(IExpression expression, string currency)
```

### [Step 2](https://github.com/marsen/Marsen.NetCore.Dojo/pull/36/commits/ef72d8480230fcad0034e04e7a396443dd98f576)

讓 Bank 的 reduce 要有意義, 所以我們想像這個運算式應該是總和(Sum)
只有型別是 Sum 的時候, 才作運算，其它部份就拋出錯誤

```csharp
    public Money reduce(IExpression expression, string currency)
    {
        if (expression.GetType() == typeof(Sum))
        {
            return Money.dollar(10);
        }

        throw new NotImplementedException();
    }
```

這裡有兩個問題, Class Sum 還沒有建立,  
這是小問題, 下一個 [Commit](https://github.com/marsen/Marsen.NetCore.Dojo/pull/36/commits/54d8fda10da4984b38a3aedd3734343fd1900e8e) 我們就把他實作出來  
另外一個問題是 NotImplementedException 並沒有被測試包覆,  
但這不是我主要的情境, 讓我學習 Kent Beck 寫到待辦清單吧。

```text
TODO List
- 只有 Sum Type 進行運算
- Hard Code 寫死回傳值
- NotImplementedException 並沒有被測試包覆
```

不過我拿到了一個紅燈, 看一下錯誤訊息

`System.NotImplementedException: The method or operation is not implemented.`

看來 expression 的 Type 並不是 Sum.

### [Step 3](https://github.com/marsen/Marsen.NetCore.Dojo/pull/36/commits/e0417ceae79ddf6c831de7dec18e3cb09b692469)

我們試著先將 Client 端(也就是我們的測試案例)的部份,  
轉換成 Sum

```csharp
    IExpression result = five.plus(five);
    Sum sum = (Sum) result;
```

這個時候編譯會失敗, 原因是 Sum 未實作 IExpression 介面,  
一樣一個[Commit](https://github.com/marsen/Marsen.NetCore.Dojo/pull/36/commits/3c7d5cc0f89b45236cfdad686890a0c526910dc2)搞定他
重新跑一下測試, 還是紅燈但是錯誤訊息變了, 無法將 Money 轉型成 Sum

> System.InvalidCastException Unable to cast object of type  
> 'Marsen.NetCore.Dojo.Tests.Books.TddByExample.Money' to type  
> 'Marsen.NetCore.Dojo.Tests.Books.TddByExample.Sum'.`

### [Step 4](https://github.com/marsen/Marsen.NetCore.Dojo/pull/36/commits/e3ba9ac67234f9a8894077d56604c8e34896ab1a)

直接 new Sum() 回傳就好, 記得嗎？我目前仍未通過測試。

```csharp
    public Sum plus(Money money)
    {
        return new Sum();
    }
```

到這一步就通過測試了, 現在只有 Sum 會回傳 `return Money.dollar(10);`
書中則是另外寫了一個測試, 但過程中我總會改壞另一個測試, 現在的步驟比較適合我

再看一下我們的待辦清單

> TODO List
> ~~只有 Sum Type 進行運算~~
> Hard Code 寫死回傳值
> NotImplementedException 並沒有被測試包覆

### Step 5

接下來我想處理 Hard Code 的部份,  
但是目前的 test Case 有點凌亂, 稍微整理一下

```csharp
    [Fact]
    public void testSimpleAddition()
    {
        Money five = Money.dollar(5);
        Sum fivePlusFive = five.plus(five);
        Assert.Equal(Money.dollar(10), _bank.reduce(fivePlusFive, "USD"));
    }
```

### [Step 6](https://github.com/marsen/Marsen.NetCore.Dojo/pull/36/commits/7902242c7ff66a741ff8aca2004cd95d686c8dd0)

加上一個新的案例來產生紅燈

```csharp
    [Fact]
    public void testSimpleAddition()
    {
        Money five = Money.dollar(5);
        Money four = Money.dollar(4);
        Sum fivePlusFive = five.plus(five);
        Sum fivePlusFour = five.plus(four);
        Assert.Equal(Money.dollar(10), _bank.reduce(fivePlusFive, "USD"));
        Assert.Equal(Money.dollar(9), _bank.reduce(fivePlusFour, "USD"));
    }
```

### Step 7

回傳計算結果, 這個職責在此回到 expression 手上,  
那 Bank 要作什麼？
在我的想像中將會是匯率與幣別的運算,  
總之, 目前還輪不到它,

```csharp
public class Bank
{
    public Money reduce(IExpression expression, string currency)
    {
        if (expression.GetType() == typeof(Sum))
        {
            Sum sum = (Sum) expression;
            return sum.reduce();
        }
        ...
```

### Step 8

讓 Sum 實作  reduce 邏輯

```csharp
    public class Sum : IExpression
    {
        private int augend;
        private int addend;

        public Sum(Money augend, Money addend)
        {
            this.augend = augend.Amount;
            this.addend = addend.Amount;
        }

        public Money reduce()
        {
            return Money.dollar(this.augend + this.addend);
        }
    }
```

目前就可以通過測試,接下來我將試著接回書上的第 14 章.
完整分支請[參考](https://github.com/marsen/Marsen.NetCore.Dojo/pull/36)

## 20210602 補充

其實完整的代碼大約在 5 月底就已經補完,  
雖然在最後整體的完整性與書上大致相同,  
但部份的實作與心路歷程是不一樣的,  
這也是我為什麼要不斷的練習相同的 Kata 的原因,  
每次的 Kata 我可能都會有不同的想法, 這點是我覺得相當有趣的地方.  
並不是所有的問題都只有唯一解, 而選擇的權利能帶來自由.

### 加上換匯的測試案例

這裡書上的範例會跑出 addRate 的方法,  
以 TDD 原則上來說不應該先有這方法才對,  
不過以 Todo List 的想法, 反而可以接受這樣的空方法(回傳或實作皆為空)  
未了避免忘記, 我會加上 `//Todo` ,  
這樣的作法更接近**如果我是使用這個 API 的人我想怎麼用**的設計理念,  
接下來是 reduce 用假實作先完成.

### 重整檔案

接下來的步驟有點脫離實作, 總之, 我覺得代碼太肥了, 開始切割成不同的檔案,  
甚至移到不同的 Namespace 之間, 這類的工作交給 IDE 作就對了.

### 離開書本

這一步是讓匯率能夠透過不同的幣別轉換(CH14),  
與書中不同的作法是直接透過查表法(LookUp)來進行轉換,  
我使用法朗兌美金(2:1)與美金兌美金(1:1)來作測試案例
書中的作法  與 Java 的語言特性有關, 而且與其時代背景有關, 所以跳過.

我想作者想表達的是,  
你可以透過測試案例來掌握一些你不熟悉的語言特性,  
甚至可以傳達意圖給閱讀這個測試案例的人.

### 回到書本

法郎與美金的加總,  
這裡要掌握的還是 Expression 的隱喻,  
使用錢包的概念我還是比較好了解, Expression 感覺在這之上墊了一層抽象.  
最後將一些共用的／抽像的方法往上抽到介面就可以了.

雖然我在上一步驟離開了書本的實作, 但其實再次回到書本之中並不難,  
原因是我走得步伐並不算大, 另一點是我的基礎設計仍與書中相同(第一個測試案例),  
試想 Expression 的隱喻如果不是我一開始設計的理念,  
而是單純的加法與轉換器的作法, 並有相對應的測試保護,  
也實際應用到了產品之中, 我有能力不影響產品的情況下重構成 Expression 的作法嗎？(反思…)

(fin)
