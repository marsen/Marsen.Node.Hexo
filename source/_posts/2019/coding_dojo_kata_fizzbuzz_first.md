---
title: " [實作筆記] Coding Dojo 第一個 Kata FizzBuzz"
date: 2019/02/06 04:11:17
tag:
  - TDD
  - 重構
  - Unit Testing
  - 實作筆記
  - Testing
---

## 前情提要

記錄一下 Kata 的思路。

## 實例化需求

```text
1 is 1
2 is 2
3 is Fizz
4 is 4
5 is Buzz
6 is Fizz
15 is FizzBuzz
```

雖然可以把上面的案例濃縮到 4 種，  
整除 3 是 Fizz、
整除 5 是 Buzz、
整除 3 又整除 5 是 FizzBuzz ,
不符合上述條件的都是原數字。

有沒有必要寫這麼多測試呢？
比如說 1、2、4 的測試是不是重複了？
日前 91 大有過類似的討論，

```text
第一個 test case 挑最簡單的，讓你可以從紅燈變綠燈。驅動出你需要的產品代碼。
接下來後面的幾個，都可以只是拿來「確認」是否滿足你期望的情境，
也就是你寫新的測試案例，你期望他就是綠燈了，然後驗證是否符合你的期望。
目的是「驗證」，不是「驅動」
```

測試的不是只有「驅動開發」而已。
而好的程式碼，也不能只依靠測試。

### 第一個測試案例，1 回傳 1

我一開始就寫成這樣，所以後面的 2、4 案例也都會是綠燈。

```csharp
public class FizzBuzz
{
    public string GetResult(int number)
    {
        return number.ToString();
    }
}
```

考慮另一種情況，也許有的人第一個測試案例會寫成這樣

```csharp
public class FizzBuzz
{
    public string GetResult(int number)
    {
        return "1";
    }
}
```

這時候就有可能需要靠 2、4 的測試案例來驅動程式碼的改變。  
實際上並沒有，第一種寫法對我來說就夠 Baby Step 了。

### 第二個測試案例，3 回傳 Fizz

```csharp
public string GetResult(int number)
{
    if (number % 3 == 0)
    {
        return "Fizz";
    }

    return number.ToString();
}
```

相信這是很好理解的，雖然我的案例是從 1、2、3 而來，  
但是在我的腦海中已經思考好了這個程式碼的「餘數規則」，

### 所有測試案例

實作出一個「餘數規則」後，程式碼應該很容易隨著測試案例變成下面這個樣子，  
用一堆 `if` 檢查「餘數」然後回傳指定的「字串」，就是我們的「規則」。
這個時候的複雜度是 4 。

```csharp
public class FizzBuzz
{
    public string GetResult(int number)
    {
        if (number % 5 == 0 & number % 3 == 0)
        {
            return "FizzBuzz";
        }

        if (number % 5 == 0)
        {
            return "Buzz";
        }

        if (number % 3 == 0)
        {
            return "Fizz";
        }

        return number.ToString();
    }
}
```

## 重構

我儘量還原當初的想法，並記錄下來，  
有許多值得改善的地方，換個順序重構起來就會更明快。

### 重構餘數檢查

這一步真的非常的的小，我想大多數的人甚至會跳過這步驟的重構，  
我只是把餘數檢查抽成私有方法，可以透過 `Resharper` 快速重構。

```csharp
public class FizzBuzz
{
    public string GetResult(int number)
    {
        if (IsDivisibleBy15(number))
        {
            return "FizzBuzz";
        }

        if (IsDivisibleBy5(number))
        {
            return "Buzz";
        }

        if (IsDivisibleBy3(number))
        {
            return "Fizz";
        }

        return number.ToString();
    }

    private bool IsDivisibleBy15(int number)
    {
        return IsDivisibleBy3(number) && IsDivisibleBy5(number);
    }

    private bool IsDivisibleBy5(int number)
    {
        return number % 5 == 0;
    }

    private bool IsDivisibleBy3(int number)
    {
        return number % 3 == 0;
    }
}
```

### 抽出 result 變數作為回傳值

這裡我是作了一個舖墊，主要是我看到了 `Fizz` 與 `Buzz` 的字串重複出現在 `FizzBuzz`，  
我預計下一階段要讓 `FizzBuzz` 是透過組合產生，而不是寫死在程式之中。
特別要注意的事是，我為了產生 result 變數，必須在最後多作一次空字串的檢查，  
這個時候的複雜度會達到 5 。

```csharp
public class FizzBuzz
{
    public string GetResult(int number)
    {
        string result = string.Empty;
        if (IsDivisibleBy15(number))
        {
            return "FizzBuzz";
        }

        if (IsDivisibleBy5(number))
        {
            result = "Buzz";
        }

        if (IsDivisibleBy3(number))
        {
            result = "Fizz";
        }

        if (string.IsNullOrEmpty(result))
        {
            return number.ToString();
        }

        return result;
    }

    private bool IsDivisibleBy15(int number)
    {
        return IsDivisibleBy3(number) && IsDivisibleBy5(number);
    }

    private bool IsDivisibleBy5(int number)
    {
        return number % 5 == 0;
    }

    private bool IsDivisibleBy3(int number)
    {
        return number % 3 == 0;
    }
}
```

### 組合 result 值

這個階段 'Fizz' 與 'Buzz' 在程式中只會出現一次，  
15 的餘數檢查也被移除了，這時的複雜度是 4 ，  
可惜的是我沒有意識到第三個 `if` 的明顯不同，
如果我能提早重構成 `result = number.ToString();`  
後面的重構也許會更簡潔一點。

```csharp
public class FizzBuzz
{
    public string GetResult(int number)
    {
        string result = string.Empty;
        if (IsDivisibleBy3(number))
        {
            result += "Fizz";
        }

        if (IsDivisibleBy5(number))
        {
            result += "Buzz";
        }

        if (string.IsNullOrEmpty(result))
        {
            return number.ToString();
        }

        return result;
    }

    private bool IsDivisibleBy5(int number)
    {
        return number % 5 == 0;
    }

    private bool IsDivisibleBy3(int number)
    {
        return number % 3 == 0;
    }
}
```

### 實作 FizzRule Class

這是繼 `FizzBuzz` 後產生的第二個 Class，  
算有指標意義，這裡原本的目的是想要消除 `if`，  
但無法一步到位，先試著把 Fizz 與 Buzz 的邏輯作分離，  
一樣我只聚焦在 Fizz 與 Buzz 身上，  
而忽略了 `其它` 的邏輯判斷，寫成了三元判斷除了變成一行外其實沒有其他好處。

```csharp
public class FizzRule
{
    public bool Check(int number)
    {
        return number % 3 == 0;
    }
}
```

```csharp
public class FizzBuzz
{
    public string GetResult(int number)
    {
        string result = string.Empty;
        var fizzRule = new FizzRule();
        if (fizzRule.Check(number))
        {
            result += "Fizz";
        }

        if (IsDivisibleBy5(number))
        {
            result += "Buzz";
        }

        return string.IsNullOrEmpty(result) ? number.ToString() : result;
    }

    private bool IsDivisibleBy5(int number)
    {
        return number % 5 == 0;
    }
}
```

### 實作 BuzzRule Class

一樣把 Buzz 的邏輯搬到新的 Class 中，
這裡故意用相同的方法名，是為了下一步要抽介面。

```csharp
public string GetResult(int number)
{
    string result = string.Empty;
    var fizzRule = new FizzRule();
    if (fizzRule.Check(number))
    {
        result += "Fizz";
    }

    var buzzRule = new BuzzRule();
    if (buzzRule.Check(number))
    {
        result += "Buzz";
    }

    return string.IsNullOrEmpty(result) ? number.ToString() : result;
}
```

### 介面 IRule

終於抽出了介面，自已為聰明的把關鍵字抽離到了介面之中，  
卻沒有考慮到真正的邏輯是組合 result 的行為仍然相依在 `FizzBuzz` Class

```csharp
public interface IRule
{
    string Word { get; }
    bool Check(int number);
}
```

```csharp
public class FizzRule : IRule
{
    public string Word => "Fizz";

    public bool Check(int number)
    {
        return number % 3 == 0;
    }
}
```

```csharp
public class BuzzRule : IRule
{
    public string Word => "Buzz";

    public bool Check(int number)
    {
        return number % 5 == 0;
    }
}
```

### IRule List

準備好了 IRule ，就是要讓 `FizzBuzz` 與 `FizzRule` 以及 `BuzzRule` 解耦的階段了，
這步我踩得有小，可以更直接一點重構，  
一樣的問題，我仍然沒有意識最後一個`if(?:)`其實也是一種 `IRule`，  
也沒有意識到 `result+=XXX` 與 `return YYY?number.ToString() : result;` 其實應該是屬於 `IRule` 的一部份  
這時的複雜度仍然是 4

```csharp
public class FizzBuzz
{
    private List<IRule> _rules = new List<IRule> {new FizzRule(), new BuzzRule()};

    public string GetResult(int number)
    {
        string result = string.Empty;
        var fizzRule = new FizzRule();
        if (fizzRule.Check(number))
        {
            result += fizzRule.Word;
        }

        var buzzRule = new BuzzRule();
        if (buzzRule.Check(number))
        {
            result += buzzRule.Word;
        }

        return string.IsNullOrEmpty(result) ? number.ToString() : result;
    }
}
```

### foreach List IRule

自以為帥氣的完成重構，而且用 `foreach` 消除了重複的 `if`...  
實際上複雜度完全沒有下降。
關鍵的 `result += rule.Word;` 與  
`return string.IsNullOrEmpty(result) ? number.ToString() : result;`  
我繼續忽視它。

```csharp
public class FizzBuzz
{
    private List<IRule> _rules = new List<IRule> {new FizzRule(), new BuzzRule()};

    public string GetResult(int number)
    {
        string result = string.Empty;
        foreach (var rule in _rules)
        {
            if (rule.Check(number))
            {
                result += rule.Word;
            }
        }

        return string.IsNullOrEmpty(result) ? number.ToString() : result;
    }
}
```

## 重構二、面對問題

### 消除 foreach

參考 Martin 大叔的作法，把 foreach 變成 pipelines  
光是這個作法就讓我的複雜度從 4 下降到 2 了，  
此時，`result += rule.Word;` 與  
`return string.IsNullOrEmpty(result) ? number.ToString() : result;`  
就顯得相當奇怪，第一個邏輯我認為應該放進實作`IRule`的類別之中，  
而第二個邏輯應該是一個未被實作的 Rule 。

```csharp
public class FizzBuzz
{
    private readonly List<IRule> _rules = new List<IRule>
    {
        new FizzRule(),
        new BuzzRule(),
    };

    public string GetResult(int number)
    {
        string result = string.Empty;
        var myRules = _rules;
        myRules
        .Where(r => r.Check(number))
        .ToList()
        .ForEach(n => result += n.Word);
        return string.IsNullOrEmpty(result) ? number.ToString() : result;
    }
}
```

### 實作 Apply

終於將`result += rule.Word;`的邏輯從 `FizzBuzz` 抽離到 `IRule` 之中，  
再由各自的 Rule 實作，這個時候就會覺得 `IRule.Check` 與 `IRule.Word` 有點累贅，  
基於 SOLID 原則，這部份邏輯甚至不該被揭露在 `FizzBuzz`之中。

```csharp
public interface IRule
{
    string Word { get; }
    bool Check(int number);
    string Apply(string input);
}
```

```csharp
public class FizzBuzz
{
    private readonly List<IRule> _rules = new List<IRule>
    {
        new FizzRule(),
        new BuzzRule(),
    };

    public string GetResult(int number)
    {
        string result = string.Empty;
        _rules
        .Where(r => r.Check(number))
        .ToList()
        .ForEach(n => result = n.Apply(result));
        return string.IsNullOrEmpty(result) ? number.ToString() : result;
    }
}
```

### NormalRule

終於加上 `NormalRule` Class 了，裡面只有一個方法 `Apply`，  
這裡是為了將來的介面準備，我想讓 NormalRule 成為 `IRule` 的一部份，  
不過可以看到的問題是，方法簽章並不一致。

```csharp
public class FizzBuzz
{
    private readonly List<IRule> _rules = new List<IRule>
    {
        new FizzRule(),
        new BuzzRule(),
    };

    public string GetResult(int number)
    {
        string result = string.Empty;
        _rules
            .Where(r => r.Check(number))
            .ToList()
            .ForEach(n => result = n.Apply(result));
        var normalRule = new NormalRule();
        return normalRule.Apply(number, result);
    }
}
```

### 修改 IRule.Apply

在我的認知中，對 Production Code 修改介面是件危險的事，  
這在 Kata 是可行的，但是在實際的 Production 恐怕就不夠 Baby Step 了，
我或許應該創造一個 IRuleV2 之類的介面，而不是直接修改 `IRule`。

首先編譯會不過，這會趨動我去修改 `FizzRule` 與 `BuzzRule`
另外，這個時間點 `IRule.Check` 與 `IRule.Word` 作為 public 的資訊就顯得相當多餘了。  
所以我會進一步將這些資訊從 `IRule` 介面中拿掉，  
這也會使得 `FizzBuzz` Class 產生 Error，趁這個時候把 `.Where()` 與 `.ToList()` 一併拿掉，  
但是要記得將 `IRule.Check` 與 `IRule.Word` 包含至 `IRule.Apply` 之中。

```csharp
public interface IRule
{
    string Apply(int number, string input);
}
```

```csharp
public class FizzRule : IRule
{
    private string Word => "Fizz";

    private bool Check(int number)
    {
        return number % 3 == 0;
    }

    public string Apply(int number, string input)
    {
        return Check(number) ? input += this.Word : input;
    }
}
```

### NormalRule 與 IRule

這裡讓 `NormalRule` 實作 `IRule` 介面，  
實際上在上面幾步已經完成了，`IRule` 反而比較像一個標籤掛在 `NormalRule` 上，  
如此一來，就能夠在 `FizzBuzz` 裡面透過 `List<IRule>` 統整所有的規則。

```csharp
public class FizzBuzz
{
    private readonly List<IRule> _rules = new List<IRule>
    {
        new FizzRule(),
        new BuzzRule(),
        new NormalRule()
    };

    public string GetResult(int number)
    {
        string result = string.Empty;
        _rules
            .ForEach(n => result = n.Apply(number, result));
        return result;
    }
}
```

```csharp
public interface IRule
{
    string Apply(int number, string input);
}
```

### 收尾

作到這裡大概把我想作的東西都作掉了，  
`if` 散落在各個 `Rules` 裡面，  
如果是 Production Code 我想我會使用 NameSpace 與專案資料夾再作進一步的整理吧。  
最後把 `FizzRule` 與 `BuzzRule` 的 `Check` 與 `Word` 拿掉只是一點潔癖。

```csharp
public class FizzRule : IRule
{
    public string Apply(int number, string input)
    {
        return number % 3 == 0 ? input += "Fizz" : input;
    }
}
```

## 結語

過程中一直考慮著想要拿掉所有`if`，或是套用職責鏈(Chain of Responsibility Pattern)的 Pattern，  
現在想想都有點走歪了方向，一再忽視責職的歸屬而讓後面的重構有點吃力，  
不過透過 TDD 仍然讓程式碼重構到了一定的程度。
如果重來一次的話，我會選擇提早分離職責，  
不過當中的取捨可能需要練習更多的 KATA 吧。

有人說這麼重構，會不會有點 Over Design 了，  
我想說的是，反正是練習嘛，刻意練習到過頭也只是剛好而已，  
如果不在練習時下點苦功，在戰場上用得出來嗎？
至少我的天賦而言，我應該是用不出來的。

### 後記 1. 20190207

#### Aggregate

文章貼出後，同事的回饋，可以使用 `Aggregate` 取代 `Foreach`，  
程式碼可以更加精鍊。

```csharp
public class FizzBuzz
{
    private readonly List<IRule> _rules = new List<IRule>
    {
        new FizzRule(),
        new BuzzRule(),
        new NormalRule()
    };

    public string GetResult(int number)
    {
        return _rules.Aggregate(string.Empty, (r, n) => n.Apply(number, r));
    }
}
```

#### 參數優化

把 r、n 這類較沒意義的命名改成 input 與 rule，
單純是為了讓 `Aggregate` 的可讀性較高一些。  
接下來這個異動的幅度較大，實務上我不會這樣作，  
讓 `Apply` 的方法簽章順序與 `Aggregate` 一樣把 `input` String 放在最前面，  
真的真的非常沒有必要，因為會異動到介面。

```csharp
public class FizzBuzz
{
    private readonly List<IRule> _rules = new List<IRule>
    {
        new FizzRule(),
        new BuzzRule(),
        new NormalRule()
    };

    public string GetResult(int number)
    {
        return _rules.Aggregate(string.Empty, (input, rule) => rule.Apply(input, number));
    }
}
```

## 參考

- [Coding_Dojo_Csharp](https://github.com/marsen/Coding_Dojo_Csharp/tree/fizzbuzz/20190205/UnitTestProject6)
- [Refactoring with Loops and Collection Pipelines](https://martinfowler.com/articles/refactoring-pipelines.html?fbclid=IwAR0uG0IXa_i6JoSRPtO6s-gXj-0jOAZDNrBYRmaHAJ2_RYFpiqcrbr4Z86k)

(fin)
