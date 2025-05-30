---
title: " [N社筆記] 沒有後端不行嗎？"
date: 2020/08/29 13:03:47
---

## 前情提要

我在今年３月已經離開 N 社了，
不過這幾天在社群與朋友蠻多人在討論一個來自[靠北工程師](https://kaobei.engineer/cards/show/5087?fbclid=IwAR0QxJCSvn_ULFo_gwqQmORL5y8UX22LBdf5neAXikALWfjo7DGA91Ohlw0)的話題，
我看了一下後，有些自已的想法，稍微記錄一下。

另外延伸討論另一個最近炎上的話題，  
前端不用寫程式 !? 出道(?)為~~全端~~網站工程師的我，談談我的看法。

## 事件 靠北工程師-把序號寫在 JS 裡面

[靠北文章聯結](https://www.facebook.com/init.kobeengineer/photos/a.1416496745064002/3246423438737981)
簡單的說，這是一個抽獎的功能，可以抽最高 550 元的折價券，
然後所有的實現都在前端，facebook 留言的回覆蠻有趣，
已經有人寫[相關文章](https://medium.com/@kurosean/%E9%96%8B%E7%AE%B191app%E7%98%8B%E5%9E%8B%E8%BD%89%E8%BD%89%E6%A8%82-7e74d402e708)，有興趣可以看看始末。

回覆的留言大多分為幾種，

1. (只?)有註解很棒
2. 序號、次數等不應該寫在前端
3. 資料筆數很多，應該怎麼處理等等…
   另外有一些來自 N 社的員工(?)留言這是行銷部門替 T 廠找的外包，  
   所以不代表 N 社的工程單位會寫出這樣的代碼。

> 20200902 補充:
> 可靠消息指出「那包 code 不只是外包而已，是給一包範本
> 叫廠商自己改完，然後放到某位置去做活動」
>
> 以上需求為前提，陣列的處理也可以理解了，  
> 整份代碼也是相同適合廠商修改，雖然不知道改得人會是工讀生還是總經理。
> 總之這樣的需求下，這份代碼我覺得是合格的。  
> 反思 : If I Had Only One Hour To Save The World I Would Spend 55 Minutes Defining The Problem And 5 Minutes Finding The Solution.

### 反思 靠北工程師

這邊說說我的看法，  
現在的我已經離開前端開發一陣子了，  
但是我覺得這份代碼(只論抽獎 js 的部份)是簡單好懂的，  
反而釣出有人作免費的 Code Review ，很多建議也算是不錯。  
但這是公司文化層面上的一個問題，
有沒有遇過「前輩」堅持某種(好壞不論)寫法呢 ?

撇除 Style 與 Code smell，這段代碼我覺得已經用最簡單的方式達到目的了。
抽獎與增券，稍微操作一樣可以理解了，抽獎完後，你可以拿到一個折價券號。
然後…這些資訊都在 js 代碼之中。  
稍微有一點概念的話，你可以直接拿到 550 元的折價券的序號，

**然後呢 ?**

如果你去消費的話，T 廠與 N 社行銷的目的就達成了… 。
而合理推測， 550 元已經在原本的計劃成本之中，而越多人買好像反而是個多贏的結局。

1. 行銷目的達成
2. 廠商銷售成功
3. 節少內部開發效能

那另一方面，這樣的代碼出去會有什麼壞處 ?

1. 讓人覺得 N 社技術不專業 ?
2. 轉移行銷焦點，從折扣碼到程式碼 XD

有辦法改善嗎 ? 首先，我認為行銷找外包技術單位實作，是一個合理且正確的行為。  
技術人員在各個公司仍是稀缺資源，所以流程上通常會被加上層層保護，  
而行銷兵貴神速，用找傭兵的方式， 迅速達成客戶的需求將是必然的選擇。

再談談外包，雖然不知道收的價金多少，最主要的功能他們是實現了，  
那有沒有辦法再好一點呢 ? 對我來說，這樣是有點傷害到 N 社開發部門的形象，  
比如先簡單的 uglify 與 minify，
再進一步用一些 serverless 的作法將折扣碼隱藏起來 ?  
或是真的起一個小小 api 服務來作這層的運算 ?  
以我來說，頂多就多作 uglify 與 minify 就足夠了吧。
這也是取捨，倒是沒有什麼對錯。

### 如何信賴專業

再來我想討論行銷人員*因為缺乏專業，所以尋求外包。但是沒有專業的他們，如何進行驗收?*
~~本想舉年初館長 300 萬網站的話題，但是館長的事另有後續，故不在本篇詳述了。~~
舉蓋房子為例，雖然房子蓋好了，也能住人。但是風一吹就倒，或是海砂屋，你能接受嗎？
目前業界主要幾種解法：

- 合約法：

> 透過合約的法律強制力來要求甲乙方，但是在文字表述上與理解上都有太多的模糊空間，
> 最後合約會變成一本厚厚的法律全書，好像什麼都有又好像什麼都沒有。
> 專案的運作，最後仍然回歸於人;當專案有所爭議之時，怎麼解讀合約反而會取代產品本身成為焦點。

- 靠關係法:

> 找信任的人作為代理人來處理，但是這樣的方式只是轉價問題，代理人缺乏專業也不少見，
> 層層溝通反而更沒效率，更常見到上線即掛點的慘案

- 第三方認證:

> 這個方法，是透過機構的認証來証明這個人具備某種能力。
> 對單一的技能是很好的門檻設定，畢業証書或是駕照都是類似的套路。
> 但是軟體開發真的只是「單一技能」嗎 ? 而你對第三方機構的信任，是不是會犯了「靠關係法」同樣的錯呢 ?  
> 証照的產業，現在也有另一個困境，
> 比如說「學店」或是「雞腿換駕照」的情況不勝玫舉，這裡我不再深入討論了…
> 舉個例子來說，n 小時學會 xxx 與 1x 天學會 ooo 真的好嗎 ?

最後，這件事其實對一間軟體公司的專業形象是有損的，  
我就用 [N 社的回應](https://www.facebook.com/91apptech/posts/164919178538886) 作為小結。
雖然我不知道為什麼不同步回覆到原始文章的底下，可能是公關想冷處理(之前 PTT 或其它事件也都無聲無息)
整個事件有很多反思的點，也有所取捨，這就是人生吧(茶)。

## 事件二、前端不用寫程式

另外這個事件也引起許多討論，因為我不想幫別人賣課，所以就不貼上啦。
可以 google 看看「 前端不用寫程式」加「15 天」，應該會有意外的驚喜。
批判的話我就不說了，不過參考上面的事件，是不是再次說明了第三方任証機構的問題 ?  
這種廉價的認証或是「學園」 ，他們的公信力真的可以相信嗎 ?

這裡我想談的是，[marketing funnel(行銷漏斗)](https://medium.com/marketingdatascience/%E7%B2%BE%E6%BA%96%E8%A1%8C%E9%8A%B7%E7%9A%84%E8%90%BD%E5%AF%A6-%E8%A1%8C%E9%8A%B7%E6%BC%8F%E6%96%97-304c1d1e8197) 。  
當行銷在推廣一件事情的時候，會盡可能的把餅作大，  
但是有時候會將一些事情去脈絡化或簡化，這樣的好處是可以減少門檻讓新手玩家快速進入，  
缺點是很有可能會誤導學員，有些觀念會被錯置。  
而學生如果再深入一點，肯定會遇到問題。  
對我來說，這反而是一個過濾器(filter)，進一步將專業分類，  
學生必須學會分辨，這是成為專業的第一步。  
有的人只是想要體驗，有的人想學以致用，有人想深入學習...。

以程式來說，先入門程式大概是什麼概念，而目前主流市場大概會用到什麼技術 ?
整個軟體開發的系統為何? 前端、後端、DBA、DevOps...
你感興趣的是哪一段 ? 你擅長的又是哪一段 ?  
往廣度延伸或是往深度延伸，或是擴及整個產業，  
而我的看法是當這個產業，越多人參與且越多人討論的時候，才會越健全。

### 後續補充

[CSScoke 的 Amos](https://www.youtube.com/embed/B5alI7bYwHw) 的直播直接拉事主出來討論~~謝罪~~了，
有這樣的討論是很好的，也可以看出來，軟體的教學方式與系統還有很多嘗試與調整的空間，  
看看[網頁 15 天 Taker(道歉與回應)](https://www.youtube.com/watch?v=wqB8w1osofY)
寫程式這行仍在茁壯擴展之中，並且深入百業之中，但人學習的速度確漸漸的跟不上了。
舊有的學校系統跟不太上了，所以才產生了補教業的市場，而其中當然有良莠不齊的現象，  
當然市場成熟後，市場機制本身就可以作一些汰除，
也或許這些講師也正在學習當中，我們可以繼續 ~~Diss~~ 督促，讓整個產業逐漸更加成熟。

順便評論一下兩方的公關處理方式，冷處理或直面道歉似乎效果都不錯，  
不過我比較欣賞後者就是了。

(fin)
