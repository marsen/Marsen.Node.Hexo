---
title: "CSM Day2-1 Planning"
date: 2018/11/3 23:47:10
tags:
  - Scrum
  - Agile
---

> 雖然實施部分的 Scrum 是可能的，但結果並不是 Scrum。Scrum，  
> 只有在完整的時候才會存在，也才能有效的成為其他技巧、方法論、和實務發揮的運作舞臺。  
> --- Scrum Guide 結語\*\*

## Planning \(with 2~4 hours timebox\)

### Planning 第一階段

- 說明要作什麼\(What\)。
- 對 Item 估點。
  - 基於 Velocity
  - 基於 Commitment

#### 估計的方法

有 Size \(大、中、小\) 或點數\(Story Point、Planning Poker\)等方法，  
實務上都使用點數較多，原因是多個項目的加總，在比較時點數較有感覺。

參考下例：  
Sprint 1 有 4 個 Item，Size 是分別是大、大、中、小。點數分別是 13、13、5、2。  
Sprint 2 有 5 個 Item，Size 是分別是大、中、中、中、小。點數分別是 13、8、8、5、1。

| Sprint   | Size             | 點數            |
| :------- | :--------------- | :-------------- |
| Sprint 1 | 2 大、1 中、1 小 | 13+13+5+2 = 33  |
| Sprint 2 | 1 大、3 中、1 小 | 13+8+8+5+1 = 35 |

> Velocity-Driven 或 Commitment-Driven 哪個比較好呢？  
> 大多數的 Scrum Master 恐怕會跟你說團隊決定。  
> 兩者的差異點在哪裡呢 ？ **Scrum 是立基於經驗導向的流程控制理論**  
> Velocity 有數據佐証，較為客觀；但是 Team members 與 Backlog Item 其實都是會變動的\(理想上當然是穩定更好\)，  
> 如果 PO 與 Team 的互信足夠， Commitment 其實也可以符合經驗導向，比如說：「 我的 Develop Team 平均每個 Sprint 裡面可以交付 90% 的 Item。」  
> 換句話說還是團隊決定，但是彈性是很大的，或許 Develop Team 有信心時使用 Commitment ，而團隊缺乏信心時或許可以用過往的 Velocity 作為標準。

### Planning 第二階段

- 組織 Task 以達到溝通與完成 Items 的目的
- 接受來自第一階段的 Backlog Items
  - 要相信「Velocity」
  - 不要馬上分派 Task
    - 會形成 Silo ，被分派者會提早將專注力從 Item 移到 Task 上
    - 保持專注在 Item 上
- 對 Task 估時。
  - 估時請包含假勤/會議/教育訓練…etc
  - 別忘了 Backlog Refactoring 的時間
  - 在 Time box 裡估完成估計的 Task
  - Maintain → 用經驗法則抓一點 Buffer 吧
  - 用小時去估時 ex:1~8，如果超過 8 小時稍後再作進一步的工作拆解
  - Task 如果仍有模糊不清的部份，可以先估一個較大的時數，再進一步拆解
  - 隨著 Sprint 進行，隨時調整 Task \(增/刪/修\)

![Sprint Backlog常常會長這樣](/images/2018/csm/day_2/Sprint_Backlog_Burndown.jpg)

- 這是用來估計而非報告，不要混為一談；Scrum 在乎的是結果，而非時間。
- 只管相對大小，ex: Task A 與 Task B，由 Alice 與 Bob 領走可能會如下呈現

  - Task A Alice 領走估時 8 hours，Task B Bob 領走估時 4 hours
  - Task A Bob 領走估時 2 hours，Task B Alice 領走估時 4 hours
  - 其實不用追究原因，或是是否相同，這因人而異，不求準確
    - 我自已的理解是
      - 而是某人估了高時數時，能不能有人來支援他？
      - 能不能透估時發現 Team Member 的能力差異，並引入其它方法提升 Member 能力 \(ex: Pair Programming、Sharing、讀書會...\)
      - 承上一點，注意佔用 Sprint 的時間，並回頭檢核是否有效；**沒有比較的改善，只是自我滿足。**

> **Sprint Backlog**  
> Tasks to turn product backlog into working product functionality.  
> Tasks are estimated in hours , usually 1-8 .  
> Team with more than 8 hours are broken down later .  
> Team members sign up for tasks , they aren't assigned .  
> Estimated work remaining is updated daily .  
> Any team member can add, delete or change the Sprint Backlog work for the Sprint emerges  
> If work unclear, define a Sprint Backlog with a larger amount of time break it down later .  
> Update work remaining as more is known, as items are worked

## Scrum Master 的 Check List

- Product Backlog Item 排序
  - 上一個 Sprint 未完成的 Item
  - Review Meeting 的反饋所產生的 Action Item
  - Acceptance Criteria 是否已經定義清楚了？
    - 需視 Scrum Team 的流程定義
    - User Story 的 Descriptions & Acceptance criteria 希望是 PO 與團隊一起共同完成。
      - 實務上蠻花時間的，寫出來的效果也不見得很好。\(\*也許是我們的操作方法不對\)
    - User story 是用來溝通的，不是用來寫的！
  - 是否有人請假？
    - 可以提前確認
    - 有人請假要評估是否吃得下？
    - 吃不吃得下由團隊評估

> PO 在 Sprint 中一直塞東西是不是一種不信任團隊的壞味道？  
> Team 毫無反應，就只是個接 Task 的 Team 是不是也是一種不信任？

![OKR in Scrum](/images/2018/csm/day_2/OKR_in_Scrum.jpg)

> 我對 OKR 與 Sprint Goal 的認知。  
> 不難理解為什麼 RD 會覺得干我屁事，~~又沒加薪也沒獎金，  
> 看了兩本書、用口說說就能讓人激勵人心太不且實際了~~。  
> 比如說：[作功德](https://www.ettoday.net/news/20171125/1059745.htm)，最後反而會讓第一線人當成幹話，幹話聽多了失去信任就更糟糕了。

## Team

適當的領取 Item，過多的 Item 進去，只會在 Sprint 結束時，有一堆半成品。  
而半成品在面對\(市場\)變化，失去產品的彈性。

> 軟體\(代碼\)的優化與重構只作一半，算不算是半成品？  
> 軟體隨著架構或相依的套件升級或補丁，在 Sprint 期間通常會有必要升級或是改變寫法。  
> 最後就變成下圖，擁有各種版本的 Code Base。

![version](/images/2018/csm/day_2/V1_V2_V3_V4_VVV.jpg)

> 這會拖慢開發速度，讓新進成員難以理解，也難以用 Code Review 或自動化 Tool 解決，  
> 目前比較理想的解法是 Pair-Programming 與時時重構。
>
> **可是，PO 願意花費這個時間去調整嗎？**  
> 可以用 Veloctiy 變低當作參考嗎？Veloctiy 變低的原因有很多種。
> 又或者由 Team 自行決定？
> Team 要能自組織要多久時間呢 ？**「不是拿掉主管，團隊就能自組織」**;  
> 如果是跨 Team 共同開發的產品又該如何？
> Arch 的調整需要更其它或上層的團隊允可(這也會占用不少時間)
> 把產品拆掉變成多個服務？讓團隊成為服務 Owner ？
> 拆成服務又需要多久時間呢？
> 組織架構調整？
>
> **當對 PO 來說，這是沒有商業價值的事。**，實務上該怎麼作？或該怎麼量化呢？  
> 註記：或許這只是個過度理想化的偽議題，畢竟「凡存在，必合理」。

![好像有點懂了，又有點不懂…](/images/2018/csm/day_2/thinking.jpg)

(fin)
