---
title: "[活動筆記] 單元測試這樣玩就對了"
date: 2017/04/23 00:01:39
tag:
  - Unit Testing
---
## 應該知道的事

- 使用 C# , 但是其他語言也適用
- 使用 Visual Studio
- 案例一有基本數理的專有名詞
  - 上界(Upper Bound)、下界(lower Bound)、左邊界(Left Bound)、右邊界(Right Bound)
- 報名資訊(已結束)
[Agile Meetup 2017/04 意外版: 單元測試這樣玩就對了](http://www.accupass.com/go/unitestwriting)

## 案例一、數值區間

```text
假定給任一整數區間
ex:
(1,6] = {2,3,4,5,6}
[-2,4) = {-2,-1,0,1,2,3}
透過一個function(x)檢查x是否包含在整數區間內,
並撰寫測試,驗証 function(x)是對的。
```

### 解析

如上範例所示,
「(」「)」小括號(parentheses)表示`OPEN`(不包含,大於或小於)
「[」「]」中括號(square brackets)表示`CLOSE`(包含,大於等於或小於等於)
 (1,6] , 代表這個區間大於1小於等於6,包含的整數有 2、3、4、5、6
 [-2,4), 代表這個區間大於等於-2小於4,包含的整數有-2、-1、0、1、2、3

![解析](https://i.imgur.com/TDHhx0A.png)

這題比較單純,只需要考慮所有的情況,
並且寫成單元測試即可。

1. x 落在區間內
2. x 落在左邊界外
3. x 落在右邊界外
4. x 落在左邊界上,左邊界為`OPEN`
5. x 落在左邊界上,左邊界為`CLOSE`
6. x 落在右邊界上,右邊界為`OPEN`
7. x 落在右邊界上,右邊界為`CLOSE`

有幾種特殊的情境,特別說明一下

1. 假設區間為(0,1),這個區間是不包含任何整數
2. 假設區間為(1,1),這個區間是不包含任何整數,且不包含任何數值
3. 假設區間為[1,1],這個區間恰巧包含1個整數,且只包含1這個整數
4. 假設"區間"為[2,1],或任何左邊界大右邊界的表示,這不是一個正確的區間,將要作例外處理。

讓我們回歸單元測試,
這裡的重點是**一個測試只作一件事**,
只把一個情境釐清,並且在測試的程式碼中
**明確的表達測試目的**

```cs
private int leftBound = 1;
private int rightBound = 6;
private int testNum = 4;

[TestMethod]
public void IncludeWhenLeftOpenRightClose()
{
    var checker = new RangeChecker(Bound.Open,this.leftBound,Bound.Close,this.rightBound);
    bool expect = false;
    bool result = checker.IsContains(testNum);
    Assert.IsTrue(result);
}
```

## 案例二、現在時間轉字串

```text
寫一個方法GetNowString,不傳入任何參數,
取得現在的時間字串,需要精準到豪秒。
再寫一個測試去測試這個方法是對的‧
```

### 版本1

最簡單的寫法:

```csharp
public class DateHelper
{
  public string GetNowString()
  {
    return DateTime.Now.ToString("yyyy-MM-dd hh:mm:ss ff");
  }
}
```

撰寫測試

```csharp
[TestMethod]
public void GetNowString()
{
  var
  //// 寫不下去,因為我們無法凍結系統的時間
  string expect = "2017-04-19 20:45:17.88";
  string result = dater.GetNowString();
  Assert.AreEqual(expect, result);
}
```

#### 解析1

`GetNowString`與系統的時間`DateTime.Now`,
是具有耦合性,要解耦需要透過一些IoC的手段去處理。

### 版本2

利用繼承的方法,作出假的類別

```csharp
public class DateHelper
{
  protected DateTime now;
  protected virtual DateTime GetNow()
  {
    now = DateTime.Now;
    return now;
  }

  public string GetNowString()
  {
    GetNow();
    return now.ToString("yyyy-MM-dd HH:mm:ss.ff");
  }
}

class StubDateHelper: DateHelper
{
  protected override DateTime GetNow()
  {
    return now;
  }

  public void SetNow(DateTime datetime)
  {
    now = datetime;
  }
}
```

撰寫測試

```csharp
[TestMethod]
public void GetNowString()
{
  StubDateHelper dateHelper = new StubDateHelper();
  var fakeNow = new DateTime(2017,4,19,20,45,17,880);
  dateHelper.SetNow(fakeNow);
  string expect = "2017-04-19 20:45:17.88";
  string result = dateHelper.GetNowString();
  Assert.AreEqual(expect, result);
}
```

#### 解析2

基本上這樣就可以測試了,
原來的代碼,經過一定的重構,
透過`virtual`方法GetNow,
將`Datetime.Now`作了隔離
適當利用假類別,取代掉GetNow的方法。

這樣夠好了,但是我們可以看看另一種作法

### 版本3

先看看我們的`DateHelper`,
在這裡我們將GetNow交由IDateProvider的類別去實作,
如此一來就斷開了耦合性。

```csharp
public class DateHelper
{
  private IDateProvider DateProvider;

  public DateHelper(IDateProvider dateProvider)
  {
    this.DateProvider = dateProvider;
  }
  
  public string GetNowString()
  {
    var now = this.DateProvider.GetNow();
    return now.ToString("yyyy-MM-dd HH:mm:ss.ff");
  }
}
```

實作IDateProvider的類別,
在這裡其實不重要.

```csharp
public class DateProviderV1 : IDateProvider
{
  public DateTime GetNow()
  {
    return DateTime.Now;
  }
}
```

讓我們看看測試,
在這裡我們透過一個假的`IDateProvider`的實作`DateProviderStub`,
完成了測試,
IDateProvider將`DateTime.Now`作了隔離,
並且提供更容易修改的假物件(僅僅需要實作觀注的方法即可,不用擔心繼承帶來的附作用)

```csharp
[TestMethod]
public void GetNowString()
{
  DateProviderStub dateProvider = new DateProviderStub();
  dateProvider.now = new DateTime(2017, 4, 19, 20, 45, 17, 880);
  var dateHelper = new DateHelper(dateProvider);
  string expect = "2017-04-19 20:45:17.88";
  string result = dateHelper.GetNowString();
  Assert.AreEqual(expect, result);
}
```

```csharp
public class DateProviderStub : IDateProvider
{
  public DateTime now;
  public DateTime GetNow()
  {
    return now;
  }
}
```

### 圖例解析

我們剛剛究竟幹了什麼？
![我們剛剛究竟幹了什麼？](https://i.imgur.com/qeqzaoO.jpg)
看看原本的情況,本來的方法因為相依與`Datetime`而無法測試
![相依Datetime](https://i.imgur.com/Mquk1Cm.png)
讓我們開始下刀,
先用一個新的方法`GetNow`
將它與待測的方法作分割,
但是對整個類來說仍舊是耦合。
![耦合](https://i.imgur.com/c0Xg4vw.png)
繼續把這刀往下切,
我們墊一層介面,
待測方法不再直接呼叫`GetNow`
而是透過介面執行,當然會有額外實作介面與注入的功(請參考程式碼,不在此處繪出了.)
![透過介面執行](https://i.imgur.com/8dDlWi2.png)
最後,別忘了我們的目的
測試原本的待測方法,
我們可以透過一個`假的`類,
來操控他的行為(ex:凍結時間).  
如此一來,就可進行測試了.
![測試](https://i.imgur.com/c3mW59v.png)
另外,這種被待測方法呼叫後
會回傳一個假值的方法或類
被叫作`STUB`
![STUB](https://i.imgur.com/KXvYMsx.png)

## 案例三、發送郵件

事先聲明,這題沒有程式碼,
有興趣實作的人可以試試看.
如果可以分享實作後的資訊給我更好XD

> Q:註冊發送郵件如何寫單元測試？

**解析**
很明顯的發送郵件需要依賴外部的郵件系統,
這裡就會有耦合性,我們可以參考案例2的方式解耦
不過發送郵件並不會有回傳值,
我們要如何驗証正確性呢？

A:檢查調用次數、參數

**圖例解析**
在案例2的單元測試,
我們透過STUB偽造的回傳值完成測試
並執行驗証.  
但是在沒有回傳的值的方法中(被稱作`MOCK`)
我們只能透過傳遞的參數(如果有多載)
與方法被調用的次數來進行驗証。

![MOCK](https://i.imgur.com/zbllutC.png)

## 重點摘要

- 單元測試要能清楚表達測試的目的(**達意**)
  - 命名
  - 減少意外的細節
- 單元測試一次只作一件事  
- new 本身就是一種邏輯 一種偶合
- static 是一種高偶合
- 繼承也是高偶合,能使用繼承的情境很少
  - A is a B 通常只有這種情境才適合繼承
- STUB & MOCK
  - STUB 會有回傳值,可以在 Unit Test 作驗証(ASSERT)
  - MOCK 沒有回傳值,可以在 MOCK 本身中 作驗証(ASSERT)

## 其它

- SLIM  
- 注入相依的幾種方式
  - Pool
  - Constructor
  - Property
- 書單 : XUnit Test Patterns

## 直播影片

_如果連結失效,煩請告知._

- [影片1](https://www.facebook.com/AgileCommunity.tw/videos/948509765286712/)
- [影片2](https://www.facebook.com/AgileCommunity.tw/videos/948548118616210/)

文章內容如有謬誤,煩請指正.

(fin)
