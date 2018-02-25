---
title: "[翻譯]C# 的常見錯誤"
date: 2018/02/12 02:12:47
tag:
  - C#
---
## 出處
http://www.dotnetcurry.com/csharp/1417/csharp-common-mistakes

## 線上工具
https://dotnetfiddle.net

## 引言

C#是個好棒棒的言語,但是它仍會有超乎你想像的行為,  
而且就算你是有經驗的開發者,你也要看一看這篇文章.  
這篇文章不講幹話,還會給你代碼喔  

![C# Quiz](http://www.dotnetcurry.com/images/csharp/basics/csharp-quiz.jpg)

### Null Value

Null 很危險啦, 你別在 Null 身上調用方法  
(譯注:在公司的維運人員應該還蠻常見這個錯誤的 一ω一)  

> We are all aware that null values can be dangerous, if not handled properly.  
> Dereferencing a null-valued variable (i.e. calling a method on it or accessing one of its properties)  
> will result in a NullReferenceException, as demonstrated with the following sample code:  


```csharp
object nullValue = null;
bool areNullValuesEqual = nullValue.Equals(null);
```

就安全的角度,好像我們要不停的檢查 reference type 是不是 null ,  
雖然這件事常常發生,好像也很難說成是非預期的行為了...  
(譯注:又有種中槍的感覺)  

> To be on the safer side, we should always make sure that reference type values are not null before dereferencing them.  
> Failing to do so could result in an unhandled exception in a specific edge case.  
> Although such a mistake occasionally happens to everyone, we could hardly call it unexpected behavior.  

看看這個代碼, null 值在 runtime 的時候不會有 type 的  

```csharp
string nullString = (string)null;
bool isStringType = nullString is string;
```

**No**, null 值在 runtime 的時候不會有 type 的  
**No**, null 值在 runtime 的時候不會有 type 的  
**No**, null 值在 runtime 的時候不會有 type 的  
很重要所以說三次,  
當然你也別想呼叫 `GetType()` 方法  

> The correct answer is **No**.  
> 
> A null value has no type at runtime.  
> 
> In a way, this also affects reflection.  
> Of course, you can’t call GetType() on a null value because a NullReferenceException would get thrown:  

```csharp
object nullValue = null;
Type nullType = nullValue.GetType();
```

純量呢？

```csharp
int intValue = 5;
Nullable<int> nullableIntValue = 5;
bool areTypesEqual = intValue.GetType() == nullableIntValue.GetType();
```

那我們可不可能用反射(reflection)區分 nullable 跟 non-nullable 的值？  
答案是不可能, 看看後面的代碼  

> Is it possible to distinguish between a nullable and a non-nullable value type using reflection?  
>
> The answer is **No**.  
>
> The same type will be returned for both variables in the above code: System.Int32.  
> This does not mean that reflection has no representation for Nullable<T>, though.  

```csharp
Type intType = typeof(int);
Type nullableIntType = typeof(Nullable<int>);
bool areTypesEqual = intType == nullableIntType;
```

上面兩段程式在runtime拿到的type很不一樣喔,  
一個是`System.Int32`一個是 `System.Nullable'1\[System.Int32\]`  


### 當 null 遇上多載方法 (Handling Null values in Overloaded methods)

```csharp
private string OverloadedMethod(object arg)
{
    return "object parameter";
}
 
private string OverloadedMethod(string arg)
{
    return "string parameter";
}
```

上面有兩個`OverloadedMethod`  
猜猜看,傳入 null 時會呼叫哪一個方法？  

```csharp
var result = OverloadedMethod(null);
```

有人會猜編譯失敗嗎？
MAGIC ! 竟然可以編譯成功, 而回傳的值是 **"string parameter"** ,  
一般來說,在編譯時期會作型別檢查,相同簽章的方法參數可以被轉型成另一個型別時,是可以編譯成功的喔.  
而有明確型別的方法將被優先調用(譯注:求這段.Net Framework的原碼來看一下,知道的人請告訴我)  

如果要指定 null 參數呼叫的多載方法就要對 null 轉型唷,可以參考下面的方法.  

```csharp
var result = OverloadedMethod((object)null);
```

### 算術運算 (Arithmetic Operations)

好像很少用位移運算吼？  
回憶一下 左移移 右移移  

```csharp
var shifted = 0b1 << 1; // = 0b10
```

```csharp
var shifted = 0b1 >> 1; // = 0b0
```

bits 跑到底並不會重頭開始喔,一直移位到爆掉就變 0 了.  
(這裡會用32是因為 int 是32bit的數值,你可以試試放超過32的數值到for loop裡會發生什麼事)  

> The bits don’t wrap around when they reach the end.  
> That’s why the result of the second expression is 0.  
> The same would happen if we shifted the bit far enough to the left (32 bits because integer is a 32-bit number):  

```csharp
var shifted = 0b1;
for (int i = 0; i < 32; i++)
{
shifted = shifted << 1;
}
```

> The result would again be 0.  

那我們是不是可以一次移32bit,讓它一次變成0呢？  
靠北啊 竟然不行捏, 你只會拿到 1,  
這跟運算子(operator)基本運算有關,在作位元運算的時候,  
會拿第一個運算數除以第二個運算數後取餘數,  
這導致我們只會拿 32 % 32 的結果 , 也就是 1 啦  
(譯注:這段其實我不是很確定,如果錯誤請糾正)  

> However, the bit shifting operators have a second operand.  
> Instead of shifting to the left by 1 bit 32 times, we can shift left by 32 bits and get the same result.  

```csharp
var shifted = 0b1 << 32;
```

> Right? **Wrong.**  
>
> The result of this expression will be 1. Why?  
>
> Because that’s how the operator is defined. Before applying the operation,  
> the second operand will be normalized to the bit length of the first operand with the modulo operation,  
> i.e. by calculating the remainder of dividing the second operand by the bit length of the first operand.  
> 
> The first operand in the example we just saw was a 32-bit number, hence: 32 % 32 = 0.  
> Our number will be shifted left by 0 bits. That’s not the same as shifting it left by 1 bit 32 times.  

好棒棒 你竟然可以看到這裡,  
那我們繼續討論 & (and) 跟 | (or) 運算子吧,  
這兩個運算子跟一般的運算子有點不一樣  
- 通常只要看運算子的第一個運算數就能得知結果  
- 在有掛 [Flag] atturibute的列舉它們好好用(看一下範例)  

```csharp
[Flags]
private enum Colors
{
    None = 0b0,
    Red = 0b1,
    Green = 0b10,
    Blue = 0b100
}
```

```csharp
Colors color = Colors.Red | Colors.Green;
bool isRed = (color & Colors.Red) == Colors.Red;
```

上面這個刮號可不能省略喔, 因為(&)運算符的優先順序低於(==)運算符,  
不過這段程式沒有刮號的話連編譯都不會過,真是好加在  
另外在 .NET framework 4.0 之後的版本提供更棒的方法去檢查flags  


```csharp
bool isRed = color.HasFlag(Colors.Red);
```

### Math.Round()

猜一下這個值會是多少？  

```csharp
var rounded = Math.Round(1.5);
```

猜2的就答對了, 下一題  
猜一下這個值會是多少？  

```csharp
var rounded = Math.Round(2.5);
```

還是2 ,
因為預設會取最接近的偶數

> **No.** The result will be 2 again. By default,  
> the midpoint value will be rounded to the nearest even value.  
> You could provide the second argument to the method to request such behavior explicitly:  

```csharp
var rounded = Math.Round(2.5, MidpointRounding.ToEven);
```

這個行為可以透過`MidpointRounding`參數改變  

```csharp
var rounded = Math.Round(2.5, MidpointRounding.AwayFromZero);
```

另外要小心浮點數的精度問題,  
以下的例子結果會是1,( 因為float的0.1實際上小於0.1 一ω一 )  
這提醒我們在處理精確數值時,應轉換成整數處理.  
(譯注:使用 [dotnetfiddle](https://dotnetfiddle.net) 時並不會有這個問題, 在windows 環境下測試的確會有問題)  
 
```csharp
var value = 1.4f;

var rounded = Math.Round(value + 0.1f);
```
### 類別初始化

最佳實踐建我我們應該避免在建構子初始化類別,
特別是靜態建構子. 
在初始化一個類別的順序如下
1. 靜態欄位
2. 靜態建構子
3. 實體欄位
4. 實體建構子

看看這個例子
```csharp
public static class Config
{
    public static bool ThrowException { get; set; } = true;
}
 
public class FailingClass
{
    static FailingClass()
    {
        if (Config.ThrowException)
        {
            throw new InvalidOperationException();
        }
    }
}
```
當我們嚐試實例化FailingClass時,你會得到Exception;  
值得注意的事,你拿到的會是`TypeInitializationException`  
而並不是`InvalidOperationException`,  

那麼我們是不是可以試著透過try catch補捉錯誤,  
並修改靜態屬性,重新實體化class呢？
**答案是不行**

一個靜態建構值,如果它拋出一個異常,  
那麼無論何時你想創建一個實例或以任何其他方式訪問這個類,  
這個異常都會被重新拋出.  

```csharp
try
{
    var failedInstance = new FailingClass();
}
catch (TypeInitializationException) { }
Config.ThrowException = false;
var instance = new FailingClass();
```
這個類別在程序重啟前是不能再被使用了(會拋出錯誤),  
這在 C# 是個非常糟糕的實踐,  
千萬別這樣設計你的類別.  

> The static constructor for a class is only called once. 
> If it throws an exception, then this exception will be rethrown  
> whenever you want to create an instance or access the class in any other way.  
> 
> The class becomes effectively unusable until the process (or the application domain) is restarted.  
> Yes, having even a minuscule chance that the  
> static constructor will throw an exception, is a very bad idea.  

#### 繼承與類別初始化

繼承的類別初始化執行順序更加複雜,看看下面的例子

```csharp
public class BaseClass
{
    public BaseClass()
    {
        VirtualMethod(1);
    }
 
    public virtual int VirtualMethod(int dividend)
    {
        return dividend / 1;
    }
}
 
public class DerivedClass : BaseClass
{
    int divisor;
    public DerivedClass()
    {
        divisor = 1;
    }
 
    public override int VirtualMethod(int dividend)
    {
        return base.VirtualMethod(dividend / divisor);
    }
}
```

當我們初始化 DerivedClass

```csharp
var instance = new DerivedClass();
```

你會得到一個除0的錯誤 `DivideByZeroException`  
這與執行順序有關
1. 呼叫 BaseClass 建構子
2. 執行 DerivedClass VirtualMethod (overrid BaseClass)
3. divisor 未賦值拋出 `DivideByZeroException`

## 多形 Polymorphism

這個例子只是要說明多形的概念與應用,  
你可以透過轉形呼叫基底類別的方法.  

```csharp
var instance = new DerivedClass();
var result = instance.Method(); // -> Method in DerivedClass
result = ((BaseClass)instance).Method(); // -> Method in BaseClass
// The correct answer is: by using the new modifier.

public class BaseClass
{
    public virtual string Method()
    {
        return "Method in BaseClass ";
    }
}
 
public class DerivedClass : BaseClass 
{
    public new string Method()
    {
        return "Method in DerivedClass";
    }
}
```

> It’s typically used to hide the interface methods from the consumers of the class implementing it, 
> unless they cast the instance to that interface.
> But it works just as well if we want to have two different implementations of a method inside a single class.
> It’s difficult to think of a good reason for doing it, though.

另外一個例子是明確實作介面方法,  
如果你的類別已經有同名的方法的話.  
雖然沒有什麼好理由建議你這樣作.
(譯注:實務上我有在遇到歷史共業這樣作過...)  

```csharp
var instance = new DerivedClass();
var result = instance.Method(); // -> Method in DerivedClass
result = ((IInterface)instance).Method(); // -> Method belonging to IInterface
It’s explicit interface implementation.

public interface IInterface
{
    string Method();
}
 
public class DerivedClass : IInterface
{
    public string Method()
    {
        return "Method in DerivedClass";
    }
 
    string IInterface.Method()
    {
        return "Method belonging to IInterface";
    }
}
It’s
```
## 迭代器 Iterators

小心 Iterators 的陷阱
看看以下[代碼](https://dotnetfiddle.net/BxfF0d):


```csharp
private IEnumerable<int> GetEnumerable(StringBuilder log)
{
    using (var context = new Context(log))
    {
        return Enumerable.Range(1, 5);
    }
}

```

```csharp
public class Context : IDisposable
{
    private readonly StringBuilder log;
 
    public Context(StringBuilder log)
    {
        this.log = log;
        this.log.AppendLine("Context created");
    }
 
    public void Dispose()
    {
        this.log.AppendLine("Context disposed");
    }
}
```

假設我們 foreach 呼叫 GetEnumerable 方法,  
你預期 Context 類別會有什麼樣的行為？  
我們會印出以下的output嗎？

> Context created
> 1
> 2
> 3
> 4
> 5
> Context disposed

```csharp
var log = new StringBuilder();
foreach (var number in GetEnumerable(log))
{
    log.AppendLine($"{number}");
}
```

不是的,  
實際上印出的是

> Context created
> Context disposed
> 1
> 2
> 3
> 4
> 5

這點很重要,  
因為實務上你很有可能 using dbconnetion 之類的物件,  
那麼你在取得真正的資料之前,  
你的連線就已經中斷了 

> This means that in our real world database example, the code would fail –  
> the connection would be closed before the values could be read from the database.

看看以下的[修正](https://dotnetfiddle.net/IgJaak)

```csharp
private IEnumerable<int> GetEnumerable(StringBuilder log)
{
    using (var context = new Context(log))
    {
        foreach (var i in Enumerable.Range(1, 5))
        {
            yield return i;
        }
    }
}
```
譯注:看到這裡對 `yield return` 的使用情境才比較有感啊...

如果你不太熟`yield return`,其實它只是個語法糖,允許增量執行,  
參考以下範例,或許能更容易理解

```csharp
private IEnumerable<int> GetCustomEnumerable(StringBuilder log)
{
    log.AppendLine("before 1");
    yield return 1;
    log.AppendLine("before 2");
    yield return 2;
    log.AppendLine("before 3");
    yield return 3;
    log.AppendLine("before 4");
    yield return 4;
    log.AppendLine("before 5");
    yield return 5;
    log.AppendLine("before end");
}
```

```csharp
var log = new StringBuilder();
log.AppendLine("before enumeration");
foreach (var number in GetCustomEnumerable(log))
{
    log.AppendLine($"{number}");
}
log.AppendLine("after enumeration");
```

> before enumeration
> before 1
> 1
> before 2
> 2
> before 3
> 3
> before 4
> 4
> before 5
> 5
> before end
> after enumeration

值得注意的事, 如果你在loop當中重複執行以上的代碼,  
那麼 Iterators 也會重複執行

```csharp
var log = new StringBuilder();
var enumerable = GetCustomEnumerable(log);
for (int i = 1; i <= 2; i++)
{
    log.AppendLine($"enumeration #{i}");
    foreach (var number in enumerable)
    {
        log.AppendLine($"{number}");
    }
}
```
輸出如下,可以明顯看到 `GetCustomEnumerable` 方法,  
實際上被隱含的執行了兩次,  
這在 Code Review 的階段也是難以被察覺的.

> enumeration #1
> before 1
> 1
> before 2
> 2
> before 3
> 3
> before 4
> 4
> before 5
> 5
> before end
> enumeration #2
> before 1
> 1
> before 2
> 2
> before 3
> 3
> before 4
> 4
> before 5
> 5
> before end

比較好的作法是將 `IEnumerable` ToList(),
如果你真的需要對 `IEnumerable` 的結果作 loop 的操作

```csharp
var log = new StringBuilder();
var enumerable = GetCustomEnumerable(log).ToList();
for (int i = 1; i <= 2; i++)
{
    log.AppendLine($"enumeration #{i}");
    foreach (var number in enumerable)
    {
        log.AppendLine($"{number}");
    }
}
```
輸出結果

> before 1
> before 2
> before 3
> before 4
> before 5
> before end
> enumeration #1
> 1
> 2
> 3
> 4
> 5
> enumeration #2
> 1
> 2
> 3
> 4
> 5

## 譯者小結

如果真的能夠預期所有的行為的開發人員,  
真的是好棒棒,  
對我來說 static class constructor 的行為是超乎預期的,  
然後對 `yield return` 的使用場景更有感覺了.  
本來預計農曆年就可以完成的翻譯,  
竟然也拖了這麼久,看來我英文還是不行啊. 

希望對大家有幫助,也請多多看原文 :)

(fin)