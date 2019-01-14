---
title: "[上課筆記] 熱血 Coding Dojo 第一梯次"
date: 2019/01/13 11:17:16
tag:
  - TDD
  - 重構
  - Unit Testing
  - Testing
---

## 知道的事

活動連結 - [熱血 Coding Dojo - 第一梯次](https://yihuode.io/activities/718?fbclid=IwAR3LPQ8HudKS72isCmeWzpe8BsTRNTTG17tChxSysXE66S3xSBypKrVzMs8)(活動已結束)
講師 : Joey Chen
範例語言 : C#

## 上課隨筆

### 觀念

- 不要寫多餘的 Product Code
- 害怕別人看你寫Code是一道門檻
- 程式腐壞的速度遠比想像中的快(大概給 3 個人寫過就開始爛了)
- Pair 時不要一開始就寫 Code，先建立共識
- 當你碰到薪資天花板就只能轉成管理者(然後是彼得效應)
- 國際化的產品能提升自已的視野與能力
- 英文不夠好是一個門檻，特別在命名的時候
- 不要臉也是一種技能
- 先考慮正確性與可維護性 再考慮效能
- 你以為你在重構 但是代碼變得更難維護
- 看人家怎麼做？學怎麼想？
- 即時重構是很重要的，太晚重構會來不及(Side Effect會大到你無法克服心魔)
- ATDD 與 TDD 的軟體工序與心魔
- 你已經具備 knowledge 但是缺乏 Couching 與實務訓練
- 代碼會反應開發當時的思緒 (不要在精神狀態不好的時候開發)
- Poker Hand 91 大約2小時完成 (思考怎麼錄製與課後練習中…)
- 睡前練習可以增強肌肉記憶

### 實作技巧

- 紅燈時不要重構
- .if .var (C# in Visual Studio)
- 第一個test case 不要有判斷式
- r n . (Vim in Visual Studio)
- 紅燈→綠燈→重構→綠燈 ; 要學會節奏與時機
- F8 跳錯誤
- 一個變數活很久 最後可能被複寫 會容易產生side effects
- zcc (Vim in Visual Studio)


### 壞味道

- 重構的壞味道，使用私有變數而非方法
- if else if 簽章抽象相同是個壞味道(或是一個可以重構的 Pattern)
- ~~Q:請回饋這堂課好的地方 A:T社的HR~~

## 寫在最後

最近覺得寫程式真的是一種造業，創造就業機會。

**與善人居，如入芝蘭之室，久而不聞其香，即與之化矣；**  
**與不善人居，如入鮑魚之肆，久而不聞其臭。**

雖很想推[熱血 Coding Dojo - 第二梯次](https://yihuode.io/activities/767)，不過大概已經完售了。

(fin)