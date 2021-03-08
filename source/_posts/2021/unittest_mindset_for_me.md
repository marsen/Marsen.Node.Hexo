---
title: "[生活筆記] 我對單元測試的想法"
date: 2021/03/02 15:29:04
tag:
  - Unit Testing
---
## 問題

你對所謂 "單元" 測試的想法爲何。

## 想法

單元測試是一種有效的方法、手段、工具 etc…
目的是為了「品質」，從兩個角度來說:

1. 對客戶來說: 功能符合需求
2. *對開發者來說: 好擴充易修改

額外的好處:

- 快速揪錯、快速回饋 : 如果團隊有執行 UT 或是 TDD 的習慣，在開發過程中就可以發現部份的錯誤
- Test Case 就是文件、就是 Use Case
- TDD 要從 UT 開始寫，讓開發者優先考慮與其互動的 Service 或 Module 會怎麼使用這個方法
- UT 是重構的基礎，有了 UT 作保護網，可以大膽重構
- *重構才能往 Design Pattern 的方向走

![Mindset](https://i.imgur.com/WbIlLiz.jpg)

如上圖，
但是我現在卡在 Design 的部份，
我可以讓測試趨動「開發」，
但是沒有辦法產生良好的設計。

所以接觸了 DDD (一知半解的狀態)，
然後想到以前學過的 Design Pattern ，但是實務上並沒有套用得很靈活，
很多時候為了 DP 而 DP, 而重構我可以作到小幅度的重構, 精簡程式碼
但是如果要重構成另一個 Pattern 時就又有點卡住了，所以我想我可能沒有掌握住軟體 Design 的技巧

## 導師 W 回饋

你的想法有些問題。

1. 最基本的 "單元" 看起來你們並沒有定義出來。
2. 測試案例並非單指文件，而是指你轉換爲測試方法那個測試名稱。重點是哪種 "單元" 需構思哪些測試案例，這才是重點
3. DP 只是結構設計過程可以應用的 "模式" 。但如果你並不瞭解該設計模式的本意與應用場合，那反而會讓開發更繁雜
4. DP 可不是爲了重構，這是兩回事的。

感覺上，你現在自學得很雜，但比較流於表面的實作技術。
又蠻想應用在開發實務上，但這樣反而會讓系統搞得更複雜。
我是建議還是要回到軟體設計的基礎功夫鍛鍊上
我覺得可能還是先鎖定在某一個主題上，
例如:"單元測試" 或 "重構"，然後端看這個主題所需培養的基礎功夫有哪些。

"單元" 指的是以 "類別" 爲單位，並依據該類型來撰寫單元測試程式碼。
以你所舉的購物車就是個很好的例子
你會把購物車相關的邏輯落實在哪一個類別呢？
然後依據購物車的計算邏輯，你會寫各種方法來測試它，這些 "各種方法" 就是測試案例了 (test case)
對於結構設計來說，最爲重要的會先界定各種類型的物件。
例如 Page (View), UI Controller (Controller), Service, Dao, Entity 等個類型的物件。
這些各種類型的物件，正是軟體人員需要爲其測試的 "單元"

## 反思與小結

最近參加了 [Implementing Domain-driven Design](https://www.tenlong.com.tw/products/9787121224485) 的讀書會，  
像是導師 W 所說，學習了單元測試與 TDD 後，  
試著應用在實務上還是有所困難的，  
所以我刻意建立了一些專案用來學習。  

Test First 或 TDD 不應該省略設計的部份，  
Domain-driven **Design** 常被縮寫成 DDD，  
TDD 則為 Test-Driven **Development**。  
而當華語文人士整天說著 ATDD、BDD、DDD 與 TDD 時，  
有注意到這個 D(Design) 不是 D(Development) 嗎 ?  

進一步來說，TDD 的要求開發之前先寫測試，意味著要先寫測試案例，  
這個步驟會讓你思考「你要怎麼呼叫你的代碼」，也就是說「你要如何設計的代碼」，  

接下來，我會用一個購物車的開發作為案例，  
試著用這個過程找到自已的盲點。  
![購物車 Sample](/images/2021/unittest_mindset_for_me_sample.jpg)  

購物車的畫面如上，我會使用 C# 的 ASP.NET Core 進行開發，  
雖然我會延用 ASP.NET 所提供的 MVC 框架，但我也會試著使用 DDD 的概念去設計，  
我會設計一系列的 Domain Model 並且使用 Domain Service 作為隔離，  
MVC 的 Controller 我會視為 DDD 的 Application Service，
這裡會出現 View Model，不同於 Domain Model，View Model 由 UI 所需要的資料決定。
更多的細節會記錄在後續的筆記當中。

![DDD](/images/2021/unittest_mindset_for_me_ddd.jpg)  

(fin)