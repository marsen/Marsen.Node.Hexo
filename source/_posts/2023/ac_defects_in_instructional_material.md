---
title: " [生活筆記] A 校的題目缺䧟"
date: 2023/12/15 13:24:33
tags:
  - 生活筆記
---

## 導言

在 A 校擔任雲端助教一陣子，主要有幾個原因，
我也是資策會出身，所以我對轉職的學生可以感同身受，  
我在教學的過程中，自已也會成長，很多時候可以學習到新知也得到成就感，
最後就是可以獲得額外的收入。  
不過 A 校在改制後，互動環節變少了，計時制改為定時制的給薪，也讓無法花太多的時間幫學生排查問題。  

除非!!我有興趣。

## 問題發現

這次就是一個我有興趣的題目，簡單說 A 社利用 [Replit](https://replit.com/) 網站提供作業，  
由學生進行撰寫，很聰明的整合方式，而且幾乎零成本。  
而且我覺得最棒的是，他可以寫測試腳本，透過測試腳本，就可以驗收大部份的作業，加速批改的時間。

但是，這次同學的作業在執行時期，發生了異常掉入了一個無限廻圈的狀態
問題不難，是一個邊際值的問題。 **應該可以加上測試保護**，這是我的第一個直覺。
我去檢查了 A 校提供的標準答案，不出意外的也有相同的問題。

另外一個問題是，**寫邊際值的測試案例**不是基本的嗎?  
查看了測試案例，竟然還真的沒有寫

## 問題排查

我選擇下載了 Replit 到地端開發，  
在開始前先簡單描述題目，在一個限範圍內進行猜數字

```javascript
const answer = Math.floor(Math.random() * 100) + 1
// start coding

let min = 1
let max = 100
let guess = Math.floor(Math.random() * 100) + 1
let count = 1

function getResult() {
  while (           ) {
    if (          ) {

    } else if (          ) {

    }
    guess = Math.floor((max + min) / 2)
    count ++
  }
  console.log(           )
}

// 以下為測試檔，請勿更動
getResult()

module.exports = {
  guess,
  answer,
  count,
  getResult
}

```

這是很好的題目，可以同時使用到迴圈與判斷，也可以接觸 Math 模組。
我們來看一下測試案例，在 Replit 右下角的 Unit Test

```javascript
test("", async function() {
  const spy = jest.spyOn(console, 'log')
  const { 
    guess,
    answer,
    count,
    getResult
  } = index;

  getResult()
  if (guess === answer) {
    expect(count).toBeLessThanOrEqual(10)
  }
});
```

測試很單純，**當答案在 1~100 之間時，執行迴圈的次數不應該超過 10 次**(其實 7 次內應該都猜得出來)  
我們應該加上一些邊際測試。
例如 1 與 100 的案例，

```javascript
  it("Answer_is_1", async function() {
    const spy = jest.spyOn(console, 'log');
    const originalMathRandom = Math.random;
    Math.random = jest.fn()
      .mockImplementationOnce(() => 0.001)
      .mockImplementation(() => originalMathRandom());
    //console.log('now random is', Math.random());
    const index = require('./index');
    index.getResult();
  
    // Clean up
    spy.mockRestore();
  });
```

```javascript
  it("Answer_is_100", async function() {
    const spy = jest.spyOn(console, 'log');
    const originalMathRandom = Math.random;
    Math.random = jest.fn()
      .mockImplementationOnce(() => 0.999)
      .mockImplementation(() => originalMathRandom());
    //console.log('now random is', Math.random());
    const index = require('./index');
    index.getResult();
  
    // Clean up
    spy.mockRestore();
  });
```

這裡注意到的是我們 mock 了`Math.random`，因為這才會影響我們的答案。

### 問題後的問題

當我們在測試案例為 100 時，會限入無窮迴圈，而 JavaScript 單緒的特性將無法離開這個測試，
雖然也不會報錯…
這驅使我加上一個測試案例，當執行次數超過 10 次時拋出例外。  
而學生使用的範本，我希望儘可能不去修改它，
這裡要對 jest 與 javascript 有足夠的理解才可以寫的好測試，  
所幸在 AI 的加持下難度下降了。

最後的測試程式如下

```javascript
describe('Guessing Game', () => {
  let originalMathRandom;
  let spy;

  beforeEach(() => {
    // Save the original functions
    originalMathRandom = Math.random;
    spy = jest.spyOn(console, 'log').mockImplementation(() => {});
  });

  afterEach(() => {
    // Restore the original functions
    Math.random = originalMathRandom;
    spy.mockRestore();
    jest.resetModules()
  });

  it("Answer_is_1", async function() {
    Math.random = jest.fn().mockReturnValue(0.001);
    const index = require('./index');
    index.getResult();
    expect(index.answer).toBe(1);
  });

  it("Answer_is_100", async function() {
    Math.random = jest.fn().mockReturnValue(0.999);
    const index = require('./index');
    index.getResult();
    expect(index.answer).toBe(100);
  });

  it('guess under 10 times', () => {
    Math.random = jest.fn().mockReturnValue(0.5);
    const index = require('./index');
    for(let i=0; i<9; i++) {
      index.getResult();
    }
    expect(() => index.getResult()).not.toThrow();
  });

  it('should throw error if count is more than 10', () => {
    const originalMathFloor = Math.floor;
    Math.floor = jest.spyOn(global.Math, 'floor')
      .mockImplementationOnce(() => Math.floor(Math.random() * 100) + 1)
      .mockImplementationOnce(() => 1000);
    const index = require('./index');
    expect(() => index.getResult()).toThrow('超過10次了，請重新開始');
  
  });


  // Add more tests as needed for the getResult function
});
```

給學生的原始程式如下

```javascript
console.log('random is', Math.random());
const answer = Math.floor(Math.random() * 100) + 1;
console.log(`answer is ${answer}`);
// start coding

let min = 1;
let max = 100;
let guess = Math.floor(Math.random() * 100) + 1;
let count = 1;

function getResult() {
  console.log(`Guess number is ${guess}`);
  while (guess !== answer) {
    if () {
      
    } else if () {
      
    }
    guess = Math.floor((max + min) / 2); //每次猜測為最大值與最小值之中間值
    count++;
    if (count > 10) {
      throw new Error('超過10次了，請重新開始');
    }
  }
  console.log(`第${count}回合，您猜${guess}，猜對了`);
}

// 以下為測試檔，請勿更動
// getResult();

module.exports = {
  guess,
  answer,
  count,
  getResult,
};
```

範本解答如下

```javascript
console.log('random is', Math.random());
const answer = Math.floor(Math.random() * 100) + 1;
console.log(`answer is ${answer}`);
// start coding

let min = 1;
let max = 100;
let guess = Math.floor(Math.random() * 100) + 1;
let count = 1;

function getResult() {
  console.log(`Guess number is ${guess}`);
  while (guess !== answer) {
    if (answer < guess) {
      max = guess; //猜太大取代最大值
      console.log(`第${count}回合，您猜${guess}，太大了，請猜介於${min}~${max}之間的數字`);
    } else if (answer > guess) {
      min = guess; //猜太小取代最小值
      console.log(`第${count}回合，您猜${guess}，太小了，請猜介於${min}~${max}之間的數字`);
    }
    guess = Math.floor((max + min) / 2); //每次猜測為最大值與最小值之中間值
    if (max - min === 1) {
      guess = max; // 當 max 和 min 只差 1 時，將 guess 設為 max
    }
    count++;
    if (count > 10) {
      throw new Error('超過10次了，請重新開始');
    }
  }
  console.log(`第${count}回合，您猜${guess}，猜對了`);
}

// 以下為測試檔，請勿更動
// getResult();

module.exports = {
  guess,
  answer,
  count,
  getResult,
};
```

## 學習與教訓

整個課程的調整與測試改寫，大概花了我 4~8 小時處理。
當然有些部份可能是我過度完美化了，但是如果需要與製作教材的人討論，
整個時間可能會更久。這題目我有興趣就順手解決了。
不過可惜的是，只算 A 校的制度下這個調整我拿不到合理的費用。

也再一次印証 AI 的強大，未來的人材需要有更高的整合能力，  
寫測試寫程式，讀技術文章賺取資訊落的錢應該會越來越難賺。

(fin)
