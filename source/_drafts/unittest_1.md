---
title: "單元測試分享(一) --- Why ? How ? What ?"
date: 2020/10/19 15:59:51
tag:
    - Unit Testing
---

## 前情提要

目前參加的讀書會有點進入了倦怠期，  
參加的人數出席不穩定，會議過程有點感覺像是在照本宣科。  
所以我們安排每隔幾周，由不同分享一下個人工作上不同的經驗。  
讓負責分享章節的人可以有更多時間準備，  
同時也可以讓聽者換換口味同時喘口氣。  

對於我來說，可以重新整理一下過往的經驗。  
試著能不能更有組織的分享單元測試這個工具與知識。  
同時，因為是線上分享會的形式，當我無法觀察觀眾的表情時，  
遠端會議應該怎麼進行才能更有效呢?  

![Conversation Cost](/images/2020/10/video_phone_conversation.jpg)

### 進行方式

- 先問為什麼?
- 輪流發表一下想法(TimeBox:60s)
- ~~Live Coding~~ (如果是實體分享，我會希望多一點實作)

### 議程

- 為什麼測試?
- 什麼是單元測試？
- 第一個測試 1+1
- 好的測試案例

### 思考一下

- Q. (單元)測試是什麼呢 ?
- Q. 為什麼要寫(單元)測試 ?
- Q. 什麼樣的情境需要(單元)測試?

我的回答，單元測試是品質保証的一種手段，  

### 困難:難以修改的代碼

### 耗費時間

- 理解流程
- 理解方法

### 建立信心

- 改 A 壞 B
- 想要重構，沒有把握
- 有了重構才能走向 Design Pattern

### 部署上線的品質保證

- 所有單元測試都綠燈
- 覆蓋率沒有下降
- 自動化測試通過(整合測試)
- 人工測試作業通過(E2E測試)

```text
一個單元測試是一段自動化的程式碼，這段程式會呼叫被測試的工作單元，
之後對這個單元的單一最終結果的某些假設或期望進行驗証。
單元測試幾乎都是可以使用單元測試框架進行撰寫的。
撰寫單元測試很容易，執行起來快速。單元測試可靠，易讀，並且很容易維護。
只要要產品不發生改化，單元測試執行結果是穩定一致的。

--- <<單元測試的藝術2nd>>
```

### F.I.R.S.T Principles

```text
- Fast : 快速→不夠快就不會被頻繁執行
- Independent :獨立→互相依賴的測試，會讓除錯變得困難
- Repeatable : 可重複→在任何環境下執行都有相同的結果(EX:時間／網路)
- Self-Validating : 自我驗証→測試是否通過，不需額外的判斷與操作
- Timely : 即時→產品代碼前不久先寫測試

---<<Clean Code>>
```

![Unit Test is basic tool](/images/2020/10/unit_test_is_basic_tool.jpg)

## First Test

### 第一個單元測試，加法計算器

#### Case:Add(+) : 1 + 1 = 2

```csharp=
[Fact]
public void Add_1_1_is_2()
{
    var target = new Calculator();
    Assert.Equal(2, target.Add(1, 1));
}
```

----

Production 代碼

```csharp=
public class Calculator
{
    public int Add(int first, int second)
    {
        return 2;
    }
}
```

#### Next Case

用 Test Case 逼出邏輯，用最簡單的方法實踐

```csharp=
[Fact]
public void Add_2_1_is_3()
{
    var target = new Calculator();
    Assert.Equal(3, target.Add(2, 1));
}
```

```csharp=
public class Calculator
{
    public int Add(int first, int second)
    {
        return first + second;
    }
}
```

#### 3A

- Arrange
  - `target = new Calculator();`
  - first number is `2`
  - second number is `1`

- Act
  - `target.Add(2, 1)`
- Assert
  - `Assert.Equal(3, acted result);

#### 3A 的補充

- 如果 Arrange 過長會是一個壞味道，
  - 表示這方法相依太多參數、服務或模組

## Live Coding

(fin)
