---
title: " [實作筆記] Bowling Kata"
date: 2021/06/25 09:20:55
---

## 前言

之前有和同事試過，並且用 Pair Programming 的方式進行，  
代碼髒得很快，並且進入了死胡同，即使使用 TDD 有測試保護，  
還是難以重構。

原因有以下：

1. 不夠了解規則，開發到一半才重新解析
2. 未經足夠的設計與討論
3. Pairs 沒有相同的想法
4. Test Case 粒度太大，不夠 Baby Step , Production 常常會多出多餘的代碼

在讀完 TDD By Example 後，我想再試一次，  
用 Todo List 方式列下我想開發的項目再轉換成 Test Case，  
不看網路上已有的 Solution 進行獨立開發(至少現在不會有 Pairs 的想法相異問題)，  
不刻意設計物件，讓測試自然趨動整體開發。

## 開始之前

先改善前幾次的問題,  
由於這次由我一個人進行開發, 所以不會有想法不一致的狀況, 實務上或許需要更多的溝通,  
規則的部份我[參考](https://ronjeffries.com/xprog/articles/acsbowling/),  
詳細內容如下 :

```text
- Each game, or “line” of bowling, includes ten turns, or “frames” for the bowler.
- In each frame, the bowler gets up to two tries to knock down all the pins.
- If in two tries, he fails to knock them all down, his score for that frame is the total number of pins knocked down in
  his two tries.
- If in two tries he knocks them all down, this is called a “spare” and his score for the frame is ten plus the number
  of pins knocked down on his next throw (in his next turn).
- If on his first try in the frame he knocks down all the pins, this is called a “strike”. His turn is over, and his
  score for the frame is ten plus the simple total of the pins knocked down in his next two rolls.
- If he gets a spare or strike in the last (tenth) frame, the bowler gets to throw one or two more bonus balls,
  respectively. These bonus throws are taken as part of the same turn. If the bonus throws knock down all the pins, the
  process does not repeat: the bonus throws are only used to calculate the score of the final frame.
- The game score is the total of all frame scores.

＃ 中文

- 一個玩家(bowler)，每場(line)有 10 個回合(frames)
- 每個回合玩家可以打兩次
- 如果兩次打完，沒有全倒，回合分數為擊倒的瓶子總數
- 如果兩次打完，全倒，簡稱 Spare，回合分數為 10 分加上下一次擊倒的瓶子分數
- 如果在第一次打完，全倒，簡稱 Strike，回合直接結束，回合分數為 10 分加上下二次擊倒的瓶子分數
- 如果在第 10 回合，
  - 玩家擊出 Spare 可以有 1 次額外擊球機會
  - 玩家擊出 Strike 可以有 ２ 次額外擊球機會
  - 額外的擊球只是為了計算第 10 回合的分數
- 整場遊戲的分數是所有回合的加總
```

有了基本規則後, 我要參考 TDD By Example 一書的作法,  
寫下 Todo List 　用來記錄我要作的事情, 當然這也會是一份湧現式的清單.

## 第一次 Kata 的 Todolist

我想像中的 BowlingGame 會提供一個計算方法，
透過傳入一組整數列，回傳目前的分數，
過程中如果有目前 Todo List 沒考慮到的東西會逐步加上

- [ ] 計算總分
- [ ] 計算回合分數
- [ ] 一回合兩次擊球，沒有全倒
- [ ] Spare，加上額外一擊的分數
- [ ] Strike，加上額外二擊的分數
- [ ] API，給定一個數列，回傳一個分數
  - [ ] 0 分不等於沒有分
  - [ ] 初始分數是沒有分

### 第一次 Kata 中斷時的 Todolist

- [ ] 計算總分
- [ ] 計算回合分數
- [x] 一回合兩次擊球，沒有全倒
  - [ ] 前面有 Spare 或 Strike 的計算
- [ ] Spare，加上額外一擊的分數
- [ ] Strike，加上額外二擊的分數
- [x] API，給定一個數列，回傳一個分數
  - [x] 0 分不等於沒有分
  - [x] 初始分數是沒有分
  - [x] 第一球就洗溝，~~0 分~~ 沒有分
    - [x] 第二球就打倒１瓶，0 分
  - [x] 第一球就打倒１瓶，~~1 分~~ 沒有分
    - [x] 第二球就打倒 0 瓶，1 分
- [ ] 最後一回合的計算
- [ ] Frame 回合的概念
  - [ ] 消除重複的 null
  - [ ] 第一次就全倒就是一個 Frame
    - [ ] FirstTry
  - [ ] 打兩次就是一個 Frame
  - [ ] Strike Frame 的分數是 null

### 第一次 Kata 的檢討

設計上仍然不足, 單純只想靠測試 → 開發其實是有點鄉愿的,  
TDD 的概念應該是以 Client(TestCase)的角度去使用 Production Code,  
這個案例中, 我想設計的 API 是一次將目前擊倒的瓶數組合成一個 List 傳給 BowlingLine,  
計算後回傳總分.

這樣的設計, 對 Client 　來說簡單好用, 但是對　 BowlingLine 來說似乎職責太多了,  
另外 Frame 的概念就消失在 Client 的視野之中, 但 BowlingLine 應該要能夠區分出 Frame  
所以我預計寫下 Frame 的測試案例. 再來, 我們發現分數在某些情況是尚未決定的,  
比如說擊出 Strike/Spare 或是只擊出該 Frame 的第一次時, 是無法計分的.  
經過第一次 Kata 後重塑對 Bowling 的認知

### 重塑認知

1. 總分是 Frame 的分數的加總
2. Frame 的分數由兩次 try 與 bonus 作計算
3. 兩次 try 的加總等於 10 才有 bonus
4. 有 bonus 的話必須計算完 bonus 才有分數

### 第二次 Todolist

- [ ] Frame 的分數是 2 次 try 的加總加上 bonus
  - [ ] 一個 Frame 未 try 過 2 次的分數是 null
  - [ ] Try 的分數計算方式是加法
  - [ ] Bonus 的計算方式
  - [ ] 有 Bonus 但是還未計算的分數為 null
- [ ] Game 的總分是 Frame 的分數的加總

### 第一次 Kata 的遺留代碼

```csharp
    public class BowlingLine
    {
        public int? Calculate(List<int> fellPins)
        {
            var firstTry = 0;
            for (var i = 0; i < fellPins.Count; i++)
            {
                if (fellPins[firstTry] == 10)
                {
                    continue;
                }

                if (fellPins.Count == 2 && fellPins.Sum() == 10)
                {
                    continue;
                }

                if (fellPins.Count > 1)
                {
                    return fellPins.Sum();
                }
            }

            return null;
        }
    }
```

#### Frame 的實作

可以看到這些遺留代碼, 雖然可以通過目前的所有測試, 但是想更進一步的時候確寸步難行.
主因是我們的設計上缺乏 Frame 的概念, 由此我會先撰寫 Frame 的測試案例

```csharp
[Fact]
public void TestFrameScore()
{
  //Frame Playing
  Assert.Null(new Frame().Score);
  Assert.Null(new Frame(1).Score);
  //Normal Frame
  Assert.Equal(7, new Frame(4, 3).Score);
  //Spare
  Assert.Null(new Frame(4, 6).Score);
  Assert.Null(new Frame(0, 10).Score);
  //Strike
  Assert.Null(new Frame(10, 0).Score);
}
```

```csharp
    public class Frame
    {
        public Frame(int? firstTry = null, int? secondTry = null)
        {
            if (firstTry + secondTry != 10)
            {
                Score = firstTry + secondTry;
            }
        }

        public int? Score { get; }
    }
```

有了 Frame 之後我要來處理之前第一次 Kata 產生的遺留代碼
首先, _Game 的總分是 Frame 的分數的加總_ 這條規則吸引了我,  
理論上所有只有一個 Frame 的測試, 在我用 Frame 的寫法後, 　
測試應該都會通過. 而且幸運的是, 我之前的測試只有 2 個測試的情境進行到了 2 個 Frame,  
所以頂多壞 2 個測試, 我可以嚐試修復它.

修改成使用 Frame 的方式

```csharp
    public int? Calculate(List<int> fellPins)
    {
        var frames = new List<Frame>();

        //todo remove this condition after pass single frame test
        if (fellPins.Count == 2)
        {
            if (fellPins[0] != 10)
            {
                frames.Add(new Frame(fellPins[0], fellPins[1]));
            }
        }
        if (fellPins.Count == 3 && fellPins[0] + fellPins[1] == 10)
        {
            var frame = new Frame(fellPins[0], fellPins[1]);
            frame.SetBonus(fellPins[2]);
            frames.Add(frame);
        }
        for (int i = 0; i < fellPins.Count; i++)
        {
            frames.Add(new Frame(fellPins[0]));
        }
        return NullableSum(frames);
    }
```

有了 Frame 的概念後, 我們可以逐一將每個被擊倒的球瓶組成一個個 Frame

#### Loop 處理 Frame

先看一下目前的代碼

```csharp
    public int? Calculate(List<int> fellPins)
    {
        var frames = new List<Frame>();
         //todo remove this condition after pass single frame test
        if (fellPins.Count == 2)
        {
            if (fellPins[0] != 10)
            {
                frames.Add(new Frame(fellPins[0], fellPins[1]));
            }
        }
        if (fellPins.Count == 3 && fellPins[0] + fellPins[1] == 10)
        {
            var frame = new Frame(fellPins[0], fellPins[1]);
            frame.SetBonus(fellPins[2]);
            frames.Add(frame);
        }
        for (int i = 0; i < fellPins.Count; i++)
        {
            frames.Add(new Frame(fellPins[0]));
        }

        return NullableSum(frames);
    }
```

我們建一個 For Loop 目標要將這些醜醜的 if 　判斷式移到　 Loop 之中
結果大致如下,　過程當然也是逐步的抽離

```csharp
    public int? Calculate(List<int> fellPins)
    {
        var frames = new List<Frame>();
        var hasBonus = false;
        for (int i = 0; i < fellPins.Count; i++)
        {
            var firstTry = fellPins[i];
            int? secondTry = i < fellPins.Count - 1 ? fellPins[i + 1] : null;
            if (hasBonus)
            {
                frames.Last().SetBonus(firstTry);
            }
            if (firstTry != 10)
            {
                frames.Add(new Frame(firstTry, secondTry));
                if (firstTry + secondTry == 10)
                {
                    hasBonus = true;
                    i++;
                }
            }
        }

        return NullableSum(frames);
    }
```

前面的 Frame 在計算分數的時候, 並未考慮 Strike 或是 Spare 完成 Bonus 的情況 ,  
所以接下來我們會用測試案例來趨動, 而最小的案例就是只有兩個 Frame 的計分狀況
比如說, 這樣的測試案例

```csharp
Assert.Equal(13, _line.Calculate(new List<int> { 3, 7, 2, 1 }));
```

另一方面, 即使算分正確,
Frame 的個數也引起我的注意, 因為有時候 Frame 裡面只會有一次 Try Hit
擔心的事就用測試作為保護吧

```csharp
    _line.Calculate(new List<int> { 3, 7, 2, 1 });
    Assert.Equal(2, _line.FrameList.Count);
```

### Bonus

Bonus 也是我寫法改變最多的地方之一
我有用過 Flag, 計數器, 最後我選擇了 Type

```csharp
    _frames = new();
    var hasBonus = false;
    for (int i = 0; i < fellPins.Count; i++)
```

```csharp
    _frames = new();
    var bonusCount = 0;
    for (int i = 0; i < fellPins.Count; i++)
```

```csharp
    _frames = new();
    var bonusType = string.Empty;
    for (int i = 0; i < fellPins.Count; i++)
```

不過更重要的是, _為什麼 Bonus 　與 BowlingLine 有關？_ 　
我們已知這個 Frame 與接下來兩次的擊球數與 Bonus 才有正相關,  
所以我應該把這個職責移到 Bonus 身上, 原始判斷 Strike 與 Spare 的邏輯,  
SetBonus 的邏輯, 也應該一併移到 Frame 身上, 這也是 OOP 的體現

原始代碼如下, 真是有夠糟糕的

```csharp
    public int? Calculate(List<int> fellPins)
    {
        _frames = new();
        var bonusCount = 0;
        for (int i = 0; i < fellPins.Count; i++)
        {
            var firstTry = fellPins[i];
            int? secondTry = i < fellPins.Count - 1 ? fellPins[i + 1] : null;
             if (_frames.Any() && _frames.Last().BonusCount > 0)
            {
                _frames.Last().SetBonus(firstTry);
                bonusCount--;
            }
             if (firstTry == 10)
            {
                _frames.Add(new Frame(firstTry));
            }
            else
            {
                var frame = new Frame(firstTry, secondTry);
                if (firstTry + secondTry == 10)
                {
                    frame.Spare();
                    bonusCount = 1;
                }
                 _frames.Add(frame);
                i++;
            }
        }
         return NullableSum(_frames);
   }
```

下面的代碼已經將 Bonus 相關邏輯移到了 Frame 之中

```csharp
    public int? Calculate(List<int> fellPins)
    {
        FrameList = new List<Frame>();
        for (var i = 0; i < fellPins.Count; i++)
        {
            var firstTry = fellPins[i];
            int? secondTry =
                i < fellPins.Count - 1 ? fellPins[i + 1] : null;
             if (FrameList.Any()) FrameList.Last().SetBonus(firstTry, secondTry);
             secondTry = firstTry == 10 ? null : secondTry;
            var frame = new Frame(firstTry, secondTry);
            if (firstTry != 10)
            {
                i++;
            }
             FrameList.Add(frame);
        }
         return NullableSum(FrameList);
    }
```

最後透過幾個重構的技巧可以讓這段代碼更好理解

- 共用 Field(這個作法是否適合還可以討論)
- Extact Method
- Inline Variable

結果如下,更多可以參考[分支](https://github.com/marsen/Marsen.NetCore.Dojo/tree/Kata/BowlingGame2)

```csharp
    public List<Frame> FrameList { get; private set; } = new();
    private List<int> FellPins = new();
    public int? Calculate(List<int> fellPins)
    {
        FrameList = new List<Frame>();
        FellPins = fellPins;
        for (var i = 0; i < FellPins.Count; i++)
        {
            var firstTry = FirstTry(i);
            var secondTry = SecondTry(i);
            if (FrameList.Any()) FrameList.Last().SetBonus(firstTry, secondTry);
            if (firstTry != 10) i++;
            FrameList.Add(new Frame(firstTry, secondTry));
        }
         return NullableSum(FrameList);
    }
     private int? SecondTry(int i)
    {
        return i < FellPins.Count - 1 ? FellPins[i + 1] : null;
    }
     private int FirstTry(int i)
    {
        return FellPins[i];
    }
```

### 結語

這樣的結果其實還沒有完成, 我接下來將會測試一些邊際
或是不合理的輸入與呼叫. 過程中的幾個亮點仍然是讓人非常的開心

- Frame 的概念
- 沒有分的計算
- Bonus 職責的轉移
- 重構

其實還有一個概念沒有被寫出來,
那就是擊出 4 球計算一個 Frame 的分數,
再有一次 Kata 的話我或許會用這個概念下去實作.

## 參考

- <https://www.bowlinggenius.com/>
- <https://kata-log.rocks/bowling-game-kata>
- [Adventures in C#: The Bowling Game](http://ronjeffries.com/xprog/articles/acsbowling/)
- <https://codingdojo.org/kata/Bowling/>

(fin)
