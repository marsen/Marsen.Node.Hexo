---
title: "[閱讀筆記] 大話設計模式 --- 橋接模式(Bridge Pattern)"
date: 2020/08/12 08:22:18
tags:
  - 閱讀筆記
  - OOP
  - Design Pattern
---

## 前情提要

因緣繼會下參加了一個線上讀書會，  
讀的是一本老書，[「大話設計模式」](https://www.tenlong.com.tw/products/9789866761799)，
簡體書應該是 2007 年出版，但是國內有繁體書，  
我是在 2010 年左右入手這本書的，號稱簡單易懂，  
當時翻了幾遍，但是對 Design Pattern 與 OOP 並沒有很深刻的認知。

如果對 OOP 沒有什概念的人，這本書的附錄也有簡單的介紹。
算是適合當作入門的書。

在實務上，卻很少看到同事在用 Design Pattern 在解決問題，  
更多是前人怎麼作，我就怎麼作。  
一直到我學習了 TDD 與重構，  
我才漸漸了解 Design Pattern 是怎麼一回事，  
書中小菜有幸遇到大鳥而我沒有，只能更多努力了。

## 本書小得

這篇 blog 主要是寫書中的第 22 章--橋接模式，  
但是我想提一下心得，也許之後會寫別的章節，也許不會，  
但心得就先收錄在這裡了。

首先是這本書面向的讀者應該是小菜，  
同時是書中的主角，對物件導向與設計模式不熟悉的人，  
所以書中用了很多現實生活中的例子來舉例，  
當然我們可以將現實投影到程式當中，但我實際上的感受還是有差異的。  
比起生活實例，我現在可能更希望是代碼實例吧。

第二點，書中的背景是 2007 年，所以時空背景已經不太相同了，  
以本章(22 章，橋接模式)為例，書中提到的手機遊戲跨平台問題，  
在 2020 年已經不存在了，現在的智慧型手機與書中的「掌上電腦」功能描述差不多了。

第三點，網路上有簡體版的書在流通，但是用語與正體中文有所差異。

想讀這本書的人參考一下上述幾點，  
或許結合一些其它的書籍或是網路資源，  
對你來說可以對設計模式有更好的理解。

## 橋接模式

我猜想命名的原因是來自完成後的類別圖看起來的樣子，
在本書中舉了手機品牌(Brand)與手機軟體(Soft)的關係作為例子，  
你可以以 `Brand` 作為分類，也可以用 `Soft` 作為分類，  
但最後都會長出三層繼承的類別圖，
然後其中都會包含奇怪的職責; `Soft` 包含 `Brand` 的資訊(或是 `Brand` 包含 `Soft` 的資訊)。

而真正的問題是，難以擴充，每當我們需要一個新的 `Soft` 或是 `Brand`，
我就需要為所有的 `Brand` 或 `Soft` 新增一整組的類別。
而這問題背後的原因就是過度繼承。

書中的例子，有一處我覺得不佳的地方，  
在使用橋接模式前，主邏輯如下

```csharp
Console.WriteLine("Run N Brand Game");
```

使用橋接模式後的代碼如下，

```csharp
Console.WriteLine("Run Game");
```

作者可能想強調其橋接的觀念，而不討論橋接的兩端實際有資訊互通的需求，  
但是我認為這是不正確的，我們不應該套用了某種 Design Pattern 而改變其原始行為。

接下來我會以下面這張圖，Demo 一下怎麼重構到橋接模式

![手機品牌](/images/2020/8/talk_DP_bridge_pattern_01.jpg)

首先，先寫測試，由於我視作這個結構是遺留代碼的產物，  
所以我不認為他會有完整的單元測試，當然如果有的話就要考慮這些測試是否仍然適用。

```csharp
  var handsetNokiaAddressBook = new HandsetNokiaAddressBook();
  handsetNokiaAddressBook.Run();
  var handsetNokiaGame = new HandsetNokiaGame();
  handsetNokiaGame.Run();
  var handsetMotorolaAddressBook = new HandsetMotorolaAddressBook();
    handsetMotorolaAddressBook.Run();
  var handsetMotorolaGame = new HandsetMotorolaGame();
  handsetMotorolaGame.Run();
```

我寫了一些 End To End 測試列舉出所有的情境。

> 延申問題
>
> 1. 目前只是 2 x 2，所以要寫 E2E 測試似乎不難，如果是 3 x 3 或更多呢 ? 你會怎麼作 ?
> 2. 這裡隱含著一件事，當你看到你的繼承鏈與商業邏輯的交互，  
>    已經開始出現 2 x 2 的現象時，是一個明示你應該重構它了。

下一步，我可以很明顯的發現中間層的類別，其實一點意義也沒有

```csharp
  public override void Run()
  {}
```

所以我們要將繼承鏈中最葉端(leaf node)的類別繼承關係移除，

```csharp
public class HandsetNBrandGame : HandsetNBrand
```

改成

```csharp
public class HandsetNBrandGame
```

當然我不認為實務上有這麼簡單能移除一個繼承關係，  
所以要達到這一步之前，我們也許要先創造無意義的中間層。
因為這步是對葉端類別的處理，所以我喜歡稱它為「修剪枝葉節點」。

下一步，當我把所有葉端的類別剪除後，我會先作分類，
實務上我會更傾向在腦海中作好分類再剪除，然後一個分類一個分類重構。
比如說 Game 類別:

```csharp

  public class Game
  {
      public void Run(string brand)
      {
          Console.WriteLine($"Run {brand} Game");
      }
  }

```

> 可以看到我已經將 brand 抽出來作為方法變數，  
> 作為過渡時期多載(overloading)或許是個手段
> 此外，可以看到我透過參數傳遞來解除相依，
> 這個手段甚至可以套用在 delegate 或是複雜型別。

同樣的步驟再作一次，

```csharp

  public class AddressBook
  {
      public void Run(string brand)
      {
          Console.WriteLine($"Run {brand} Address Book");
      }
  }

```

我們可以明顯發現重複的項目可以抽出介面，  
實務不需要特別介意是 `interface` 或 `abstract class`，  
依最小改動為原則，選擇適當的手段進行即可。

```csharp

  public interface Application
  {
      void Run(string brand);
  }

  public class AddressBook : Application
  {
      public override void Run(string brand)
      {
          Console.WriteLine($"Run {brand} Address Book");
      }
  }

  public class Game : Application
  {
      public override void Run(string brand)
      {
          Console.WriteLine($"Run {brand} Game");
      }
  }

```

這時候我們必須將 `brand` 傳入，

```csharp

  public class HandsetNBrand
  {
      protected HandsetSoft Soft;
      protected string Brand;
      public HandsetNBrand(HandsetSoft soft)
      {
          Soft = soft;
          Brand = "NBrand";
      }

      public void Run()
      {
          this.Soft.Run(Brand);
      }
  }

```

而 Client 端就可以組合起來使用。

```csharp
  var game = new HandsetNokia(new HandsetGame());
  game.Run();
```

同樣的手法作在 `HandsetMBrand` 之中

```csharp

  public class HandsetMBrand
  {
      protected HandsetSoft Soft;
      protected string Brand;
      public HandsetMBrand(HandsetSoft soft)
      {
          Soft = soft;
          Brand = "MBrand";
      }

      public void Run()
      {
          this.Soft.Run(Brand);
      }
  }

```

這時候可以發現重複，重構後如下。  
除了各自的品牌資訊，大多可以共用的方法與欄位，  
我們就抽到父類別。

```csharp

  public class HandsetBrand
  {
      private readonly Application _app;
      protected string Brand;

      protected HandsetBrand(Application app)
      {
          _app = app;
      }
      public void Run()
      {
          this._app.Run(Brand);
      }
  }

  public sealed class HandsetNBrand : HandsetBrand
  {
    public HandsetNBrand(Application app):base(app)
    {
        Brand = "NBrand";
    }
  }

  public sealed class HandsetMBrand : HandsetBrand
  {
    public HandsetMBrand(Application app):base(app)
    {
        Brand = "MBrand";
    }
  }

```

![完成](/images/2020/8/talk_DP_bridge_pattern_02.jpg)

## Recap

1. 修剪枝葉，
2. 製造重複，重構並產出 `Implementor`
3. 如果橋接的兩端有需要傳遞的資訊，考慮使用方法參數
4. `Abstraction` 抽象呼叫 `Implementor` 來建立橋接
   ![Bridge](/images/2020/8/talk_DP_bridge_pattern_03.jpg)

## 參考

- <https://refactoring.guru/design-patterns/bridge>
- <https://sourcemaking.com/design_patterns/bridge>
- <https://simpleprogrammer.com/design-patterns-simplified-the-bridge-pattern/>
- [深入淺出 Design Pattern](https://www.tenlong.com.tw/products/9789867794529)

(fin)
