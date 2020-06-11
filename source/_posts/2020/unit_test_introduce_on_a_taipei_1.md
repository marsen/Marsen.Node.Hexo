---
title: "[A社筆記] Introduce Unit Test --- 心法篇"
date: 2020/06/11 17:03:11
tag:
    - 實作筆記
---


## Why Unit Test 心法篇

網路上很多，自已找(兇)。

### 我的想法 about Unit Test

#### 點

- 有驗收才有品質，所以我需要測試
  - 黑箱
  - 白箱
  - 整合
  - 單元
- UT 不過是驗收的最基本的單位。
- 開發人員一定會測試的，不論用何種方法
  - Debuger
  - Console Output
  - Break Point
- 程式人員的好品德
  - Laziness
  - Impatience
  - Hubris
- 程式是照你寫的跑，而不是照你想的跑

#### 線

- 既然你會測試，那麼為什麼不讓它可以[重複/一鍵]被執行

#### 面

- 只是為了重複執行，何不使用現有的測試框架與工具?

#### 體

先訂驗收標準，再進行開發是正常不過的作法，  
只不過你太聰明而在腦中測試過了。
但是**程式是照你寫的跑，而不是照你想的跑**。
不如先寫下驗收標準(測試左移/DoD/TDD),再進行開發。
錦上添花的話，就透過工具讓「執行測試」可以快速重複。
剩下的問題是，這些驗收標準寫到多「鉅細靡遺」?
有沒有什麼方式可以提昇我撰寫的速度 ?

假設我有了測試保護，那麼重構將是一件安全的事。
壞味道可以給我提示，而 Design Pattern 可以是改善程式的一個指引。

## Unit Test

### 3A

程式寫的是 AAA (正序)心裡想的是AAA(逆序)

- Arrange
- Act
- Asset

### 紅、綠、重構

如果以 TDD 開發，寫完新的測試後，得到的一個「綠燈」反而是一個壞味道。

## 第一天分享

pair programming 30 min，QA 約 20分鐘。
先請同事實作 1+1 的 Unit Test 看一下他對測試的理解。

- 建立測試專案，可以選用 xUnit

    ```text
    不選 MsTest 的理由是，我比較喜歡建構子與解構子的寫法，  
    勝過 TestInitialize/TestCleanup 的 Attribute 的寫法
    ```

- 引導由測試寫出方法。

    ```text
    logic → function → class method  
    透過寫測試讓 Production 在思考中產生
    引導的沒有很成功
    ```

- 3A 的寫法，未來會介紹沒有 3A 的寫法(為了更好的理解)
- ~~方法~~測試案例的命名
- 介紹 Assert.Equal 取代 Debug.Assert
- 簡單提到三種邏輯，回傳值、改變值、互動。
- 為什麼我討厭 Static

### [未排序]預計要講的題目

- 範例是否為偶數 → 牛奶是否過期 → 如何透過一些手法，控制不可控的類別
- 怎麼對 Legacy Code 作解耦 ?
- 介紹 Mock Framework
- 怎麼寫出好理解的 Assertion ?
- 介紹 Assert Framework
- 建立 A 社的道場 Repo
- more ...

## 問題

- 開發~~完~~後，自已會完全不自已測試就丟給 QA 或客戶嗎 ?
- 什麼是 Dojo ?
  - 日文的道場，把寫程式想像成是在練功，建立一個練功的環境。  
- 什麼是 Kata ?
  - 日文的形，或是可以說是套路/招式，一樣透過練習招式來強化自已的能力。

## 參考

- [RIP TDD](https://www.facebook.com/notes/kent-beck/rip-tdd/750840194948847/)
- [Laziness Impatience Hubris](https://wiki.c2.com/?LazinessImpatienceHubris)
- [瞭解單元測試](http://otischou.tw/2019/08/02/unit-test.html)

(fin)
