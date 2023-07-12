---
title: "[KATA] 最大公因數與最小公倍數"
date: 2018/09/18 00:36:51
tags:
  - .Net Framework
---

## 簡介

數學系畢業後，在學寫程式雖然偶爾會用到一些數學特性來解題，
不過到了工作確不是那麼一回事，大多在處理商業流程，資料流 ;
然後在商業流程與資料流中穿梭，找蟲然後補丁 ;

這是最近在練習 [Kata 的題目](https://www.hackerrank.com/challenges/between-two-sets/problem)所用到的兩個國中數學，
「最大公因數」與「最小公倍數」，還用程式實踐了「輾轉相除法」，
真的是有點懷念啊 。

這是題外話，Rx(Reactive Functional Programing)感覺很有趣
最近參與 functional programing 的讀書會，
裡面似乎有比現在開發模式更優雅的解決問題方式。
不過門檻有點高，需要具備的抽象思考能力，是種非常燒腦的開發方式。

## 實作

```csharp
//// GCD
static Func<int, int, int> GCD = (numA, numB) => {
        return numB == 0 ? numA : GCD(numB, numA % numB);
    };
//// LCM
static Func<int, int,int> LCM = (numA,numB) => {
            return numA * numB / GCD(numA, numB);
        } ;
```

### 參考

- <https://www.geeksforgeeks.org/gcd-two-array-numbers/>
- <https://www.geeksforgeeks.org/lcm-of-given-array-elements/>

(fin)
