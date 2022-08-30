---
title: "CSM Day2-3 Scrum Master"
date: 2018/12/16 02:45:08
tag:
  - Scrum
  - Agile
---

### Scrum Master

哪些人適合作 Scrum Master ?  
常見組織轉型時會直覺的找現有的角色作 Scrum Master ，  
但是 Scrum Master 是一個 **Coaching** 的角色，  
傳統公司裡面並沒有 **Coaching** 的角色，通常都行不通的。

![從 Manager Driven 變成 Custom Driven](/images/2018/csm/day_2/closer_customer.jpg)

> 傳統公司裡面的 Manager ，要麼是 R&D 的 Manager ，要麼是 PMO 的 Manager。通常會有幾個問題
>
> 1. R&D 通常由資深 RD 升上來，但是不善於管理\(彼得效應\)
> 2. PMO 會有自已與部門的考績考量，而這不一定能瞄準市場目標
> 3. 通常工作都會由 Manager 分派，而不是由成員主動爭取
> 4. 失敗的時候，總要有人背鍋，那個人就是 Manager \(但是實務上常常看到 Manager 丟鍋給成員\)
> 5. ~~不太關注成員的成長。~~\(好啦，也許有的有\)
> 6. Manager 要作太多的決策

### Scrum Master 不是 Decision maker

情境：當 Team Member 無法決定求助 Scrum Master 時。  
解法：也要看情境，干預或不干預是要付出成本的。  
你要關心「風險」與「成長」，在沒有風險的情境下 Scrum Master 應該引導團隊自行作決定。  
問問題是一個好的引導方式，過多的干預會讓團隊無法成長。

以小孩作比喻

| 情境                               | 作法                                        |
| :--------------------------------- | :------------------------------------------ |
| 小孩爬桌子（低風險）               | 讓團隊試試看，讓團隊學習、成長。            |
| 小孩爬馬路（高風險）               | 作出 Decision，避免失敗\(失敗就沒有下一次\) |
| 別人家的孩子爬桌子（不穩定的團隊） | 防東防西，風險至上\(他成不成長干我啥事?\)   |

如果把時間拉長一點，可以觀察出自已的取捨是否太偏向某一方\(風險 or 成長\)，  
比如說幾個月，如果團隊仍無法自行決定，可能是你\( Scrum Master \)干預太多了，  
Scrum Master 每一個決定都會影響到團隊。  
Scrum Master 應該給自已訂一些目標，來判斷自已的取捨是否合理。

Ex:

- 團隊是否會自行分工
- 團隊是否會自行作決定
- 團隊是否會蜂擁處理最高優先權的東西
- more…

更多情況 Scrum Master 不見得能作 Decision。

Scrum Master 的責任

- Coach PO

  - 消除客戶與開發之間的障礙
  - 教導 PO 如何透過 Scrum 最有效率達到目標\(How to maximize ROI and meet their objectives through Scrum \)

  以下圖來說，Y 軸代表價值，X 軸代表時間。上圖的策略表示產品初期就發佈高價值的增量，  
  隨著時間過去，單位時間能帶來的價值太少時，也許我們就不作了\(虛線之後\)，因為不符成本。  
  而實務上，可能會更接近下圖，在初期有些基礎建設，這些建設不一定能帶來較高的\(客戶\)價值，  
  但是可以降低風險，有時候更可能是初期必要的相依項目。這兩種策略沒有好壞，關鍵點仍是要能結合你的產品，  
  與 PO 共同討論出取捨的方向。 User Story Mapping 是一個工具，怎麼樣找到 Walking Skeleton ，  
  怎麼在這個基礎上豐富你的產品，這都是 Scrum Master 的職責。

![兩種不同的迭代策略](/images/2018/csm/day_2/increment_pattern.jpg)

- Coach Team

  - Improving the lives of the development team by facilitating creativity and empowerment
  - 以任何可能的方式提高開發團隊的生產力

    團隊常見的兩個問題，作太少或是作不完。可以嘗試一些工程實踐，但是別忘了工程實踐的目的是讓 Sprint Done。  
    比如說：

    > mini-waterfall 的流程可能會導致 Item 作不完，原因是 Testing 的角色在最後面才會進來，會有 Items 作不完，  
    > 提早發現其實是件好事，不論是**全都作不完**、**高優先權的作不完**或是**低優先權的全都作完**都是很好的干預點，  
    > 只要在 Retrospective 將作不完的東西攤開\(透明\)，分析問題就可以有機會改善
    >
    > 引入不同的流程、開發方式都會有一個學習與生存的焦慮在裡面。可以透過 Coaching 降低學習焦慮，  
    > 你要尋找適合的人選與資源，這是 Scrum Master 的職責。  
    > 有的團隊的抗拒會比較強，Scrum Master 要找好時機進行，例如：Sprint 失敗時。

  - Improving the engineering practices and tools so each increment of functionality is potentially shippable。

    > 實例化需求\(SBE\)、驗收趨動開發\(ATDD\)  
    > 實踐上怎麼作呢 ?  
    > 在 Sprint 中的 Item 應該都有驗收標準\( Acceptance Criteria\)，  
    > 這都應該在 Planning 或 Refinement 的階段被列出來，更進一步應該變成 test case。  
    > Example :  
    > Sprint 裡有 Item1、Item2、Item3、Item4
    >
    > Item1 應該會有 test case 1.1，test case 1.2...，Item2 應該會有 test case 1.1，test case 1.2...，  
    > 正常一個 Item 應該在兩三天完成這個功能。  
    > 這時候就可以測試 test case，假設 test case 4.3 作不完就不作了，這樣就不會留半成品。
    >
    > 再透過持續整合\(CI\)的實踐，避免後面的改動，影響到前面。
    >
    > 實務上，一種是團隊很喜歡作這類的實踐與改變，另一種更多  
    > 「團隊作不到 Done，在 Retrospective 階段來趨動改善的過程」，為了解決問題。

```text
Question:
1. Item1、Item2、Item3 會改到同一個模塊，所以 RD 會習慣同時開發(F2E反應)
2. 沒有持續集成(CI)，或是 CI 不包含自動化測試怎麼辦？
    - 如何快速寫出一個自動化測試試？
**
```

### 問一個問題，**「什麼讓我們慢下來**？，**什麼讓我們不能更快？」**

通常第一線的人員\(Developer\)不覺得慢是一個問題，甚至不會在 Retrospective 提出。Scrum Master 的職責是找到這個問題。

小結：

- Scrum Master 要讓團隊與 PO 深入理解 Scrum
- Scrum Master 就像是牧羊犬要保護羊群\(團隊\)，因為會有狼\(插件 or something...，或是羊群裡有狼\)
- Scrum Master 不作決策，更多是關心決策的過程甚於決策本身
- 具有生產力的團隊就是 Scrum Master 的產出
- Scrum Master 要發揮影響力\(知易行難，怎麼作？\)

### 情境題

避免爆雷，不描述課堂上的情境，但是將一些原則列下:

1. 很多人第一時間會找 Scrum Master 問問題。
   - 這是好事 Scrum Master 要發揮影響力
   - Scrum Master 可以籍些說明 Scrum 在作什麼，團隊是怎麼運作的。
   - Scrum Master 不作決策更關注過程。
   - 不要急著解決問題。
2. Product Owner 要基於 Product Value 作排序
   - 如果要最大化價值，一個產品就一個 Product Backlog，不管是多少個 Team。
   - 一個插件有幾種可能
     - 放進下面幾個 Sprint Backlog。
     - 放進 Product Backlog。
     - 很異常的情況，才會終止 Sprint。
   - Scrum Master 要關注 Root Cause
3. 保持團隊與 Product Owner 的連結
   - 有時候 Product Owner 也身不由已。
   - 如果 Product Owner 不在決策圈或未被授權，要了解背後的原因。
   - 如果 Product Owner 太忙，想辦法減輕他的壓力。
   - 如果 Product Owner 有別的角色\(Sales、Boss…\)，找個適合的人作 Product Owner。
   - 改變文化 改變組織 改變作法。
4. 讓 Product Backlog 作為團隊工作項目唯一的入口
5. 在不被 fire 的情況下，對組織帶來改變與價值！要有勇氣。
6. 不要讓團隊與 Product Owner 成為甲方乙方。
7. 讓團隊自已挑選工作，而不是分派工作。
8. 觀注事實。
   - 團隊的 Velocity
   - 上個 Sprint 完成的點數
   - 誠實面對失敗\( _柯語錄:面對挫折打擊不是最困難的；最困難的是面對各種挫折打擊，卻沒有失去對人世的熱情_\)。
   - 思考著如何讓 Product 的 Impact 發生
     - 昧著事實去滿足時程與範圍，可能會為此喪失品質與生產力\(欠債…\)。
     - 找到正道，但是也許會更花時間。
     - Change Your Company。
   - 觀注 Product 的成功勝於作了多少工作。
   - 盲人摸象的故事，每個人都對，也都不對。

### Part-time or Full-time

Scrum Master 觀注改進，同時兼任多個角色時，容易陷入可量化產出的角色之中。  
要想辦法讓 Scrum Master 的工作可視化，不然容易淪為開會召集人或訂訂便當與飲料的角色。  
實踐：

- 使用 Scrum Master Check List [http://scrummasterchecklist.org/](http://scrummasterchecklist.org/) 。
- 建立 Scrum Master 的改善 Impact Backlog，並且設定優先級。
- 按優先級逐步的改善。
- 尋找一組 Scrum Master 彼此討論與評量，讓這個流程形成一個循環。
- Less 的解決方式是 Full-time Scrum Master 兼任多個 Team。

相同的作法，對不同的 Team 不一定有用， Scrum Master 如果能接觸不同的團隊是好的。或是從其它 Scrum Master 汲取經驗。

### 呂毅老師的實務分享

常見一個問題，Product Owner 常常單向對 Team 輸入訊息，導致最後的結果與 Product Owner 的想法有落差。

一些壞味道

- Planning 的時候大家「帶電腦」作自已的事或是在「滑手機」。
- 開會人數太多，部份人在討論時，其它人放空。

Solution：

- 讓成員不要帶電腦，收走手機
- Product Owner 講完換 Team 講
- 測試
- 拆散小組至 2-3 人
  - 有點類似 Lean Coffee 的作法，讓團隊拆成小組討論 。
  - PO 輪流在小組之間被問問題。
  - 如果等不到 PO 可以先寫在便利貼貼到白板上。
  - 設定 Time Box ，時間到請 PO 回答問題。

### Ending

不要反射性的去解決問題，讓子彈飛一會兒…。

- 讓問題變透明，讓團隊看見問題
- 不急著干預，試著讓團隊自行解決問題
- 讓 Team 與 PO 直接交流，不要成為 PO 與 Team 的傳話筒。
- 要保持「意識」，不要條件反射去干預，要克服這個心魔。

Real Team

- 所有人的目標是一致的，而不是臨時組成的一群人。
- 俱備 End to End 的完整。
- 有限的人數，Scrum 建議 5~9 人。
- Mutual accountability
- Agreed way of working

![Scrum Team 是自組織的團隊](/images/2018/csm/day_2/authority_matrix.jpg)

一個好的 Scrum Master 的產品是 Well-Working Team， 這需要時間\(以年計算…\)。  
如何打造一個 Team ，這比 Scrum Maser 有更多的討論，但是實務上在成為 Scrum Master 時，大多數人打造 Team 的基本功是缺乏的\(彼得原理？\)，這需要更多的學習…

## 參考

- [\[N 社筆記\] 敏捷路上跌倒站起來 2018/11/1](https://blog.marsen.me/2018/11/18/2018/csm/91app_scrum_masters_growth_camp/)
- [\[閱讀筆記\] The Great Scrum Master 第一章](https://blog.marsen.me/2018/07/15/books/book_the_great_Scrum_master_ch1/)
- [The role of the Scrum Master – Part I](https://stayrelevant.globant.com/en/the-role-of-the-scrum-master-part-i/)

## 心得小結

1. 還好上課有記錄，課程很有料，過了一個月想法仍源源不絕的出來。
2. 你可以繼續 Scrum 自助餐，但是那個不是 Scrum 不是守破離。
3. 一邊觀察 N 社團隊的運作一邊寫筆記，Change My Company。
4. GitBook 也很方便的筆記工具，寫完後同步到 Github 就能拿到九成完美的 markdown 。
5. Scrum 給團隊更多職責，所以團隊要更強才行。
6. Cross Learning 不僅僅是溝通層面而已，而是為了真正的 End to End。你的 End 到底到哪裡 ？如何 DoD？
7. 所有聲音都是真的，不要急著說服別人，記住盲人摸象沒有一個瞎子說謊。
8. 第二步我在哪裡？第一步看見全貌、先要透明，透明的意思是有共識，再此之上是溝通與信任…不要把人僅僅當作 Resource。
9. 團隊裡面不要有小團隊/人數控制在 5~9/保持團隊穩定/Real Team，好難…實務上怎麼作 ？
10. Walking Skeleton 可以貫通在 PBI/User Story/Task 之間，要盡可能的薄但暢通，再此之上才有增量。
11. 可交付的增量不是 Release，也不是 MVP。MVP 比你想像的還大。
12. Sprint 的觀念在蕃茄鐘或 GTD 也有反覆出現過，在 TimeBox 中反覆實驗，尋求改善。

(fin)
