---
title: "[學習筆記] SDD 與 Spec Kit - 前言"
date: 2025/10/29 13:06:15
tags:
  - 學習筆記
---

TL;DR

## 簡介 SDD

> What is Spec-Driven Development?
  Spec-Driven Development flips the script on traditional software development.  
  For decades, code has been king — specifications were just scaffolding we built and discarded once the "real work" of coding began  
  Spec-Driven Development changes this: specifications become executable, directly generating working implementations rather than jus
  guiding them.  -- spec-kit

什麼是規格驅動開發 ???  
跟過往的開放方式什麼不同???  
什麼原因讓它興起 ???  

## [傳統開發流程(1990之前)](https://blog.marsen.me/2025/10/30/2025/sdd_spec_kit_water_fall/)

## 敏捷開發流程(1990~至今)

敏捷開發（Agile）的出現，是對瀑布式開發的反思。它強調：

- 個人與互動 > 流程與工具
- 可用的軟體 > 詳盡的文件
- 客戶合作 > 合約協商
- 回應變化 > 遵循計劃

敏捷的典型流程（以 Scrum 為例）：

### Product Backlog → Sprint Planning → Daily Standup → Sprint Review → Sprint Retrospective

這是一個迭代循環，每個 Sprint（通常 2-4 週）都會產出可用的軟體增量。

- Product Backlog: 對齊高層期望，主要角色 Stackholder/Product Owner
- Sprint Planning: 對齊實作時程與方式，主要角色 Product Owner/Team
- Daily Standup: 對齊每日進度，全角色參與
- Sprint Review: 對齊結果，全角色參與
- Sprint Retrospective: 對齊改善，全角色參與

### 雖然實施部分的Scrum 是可能的，但結果就不是Scrum 了

敏捷的特點：

- 迭代且增量 - 小步快跑，頻繁交付
- 溝通導向 - 強調面對面溝通勝過文件
- 早期且持續交付 - 每個 Sprint 都有可展示的成果
- 擁抱變化 - 將變更視為常態而非例外

在敏捷模式下，規格文件的角色變成了溝通工具和備忘錄。  

User Story、Acceptance Criteria 這些輕量級的規格，主要是為了確保團隊理解一致。

程式碼本身成為了真相的來源 - "Code is the truth"。

在迭代過後，這些文件不再有意義。

這解決了瀑布式的一些問題，但也帶來新的挑戰：

- 知識容易流失（口頭溝通沒有記錄，使用團隊記憶）
- 技術債累積（快速迭代可能犧牲品質）
- 規格與實作的差距依然存在
- 實際上團隊/文化等因素，在跑的只是 Scurm-but/小瀑布/隕石
- 商業化的行為已經扭曲原意，甚至作為 KOL 老王賣瓜的神主牌 (看看"敏捷社群")
- 在大型產品/專案仍然不足夠，因而衍生 SAFe/LeSS 等方法論
- 淪為表面功夫

## 隕石開發流程(Alpha~Omega)

典型流程

### 老闆一句話 → 隨便硬幹出來 → 夠奴就加班 → 交付疊加態 → 塊逃啊

- 老闆一句話：靈感一來就開幹，需求模糊。
- 隨便硬幹出來：原型被誤當產品，直接上線。
- 夠奴就加班：團隊燃燒工時與理智，一次衝到底。
- 交付疊加態：交付內容沒人知，時程沒人知，測沒測沒人知。
- 塊逃啊：事後檢討變成批鬥大會，成功不在你，但失敗一定有份。

### 特點與本質

高壓短命、方向即興、溝通崩壞、文件失效、驗收模糊。
看似混亂，其實是一種生存策略(實用主義)：

- PM 可保命：有 Demo、有交付，能交差。
- 行銷能吹噓：原型當 MVP，上媒體先贏聲量。
- 市場能行（騙）銷：先開賣、後補齊。
- 老闆能交代：有成果、有故事，股東就安心。
- RD 隨便作，洗個經驗跳糟加薪比較快

隕石開發不是技術問題，而是組織文化的選擇。

## 這些流程的異同、優劣與困境

瀑布、敏捷與隕石開發的差別，反映的是組織文化與生存策略。

瀑布式重規劃與控制，適合穩定需求與高風險專案，但變更成本高、回饋慢。

敏捷式重回饋與溝通，能快速交付並擁抱變化。

隕石式則靠爆發力求生存，短期有成效，不考慮長期品質與維運。

我的看法是新創成長到高原期時，就會從隕石走而敏捷，再大規模就會考慮瀑布/LeSS/SAFe，

而市場大部份都是中小企業，真的能成長成巨人者(amazon/spotify/netfilx)，

一定有它的原因，開發流程應該隨著公司業務成長，形成自已的文化，沒有一定的好壞，也不是可以輕易模仿的。

## 工程師的自我要求

我個人是傾向 XP 的開發模式，但是它的缺點是學習門檻高，  

對面試沒幫助，更多的公司寧願你寫白板題或刷 LeetCode。

團隊不願意投入，相關的 TDD/ATDD/BDD 或是測試左移更難導入小團隊之中。

高層如果沒有技術能力，無法識別好壞，就容易被唬弄，導致需求與實作有落差

## AI SDD 流程(2024~未來?)

AI Agent 或 SDD 出現後能不能帶來改變?

或是可以帶來什麼幫助呢?

對工程師而言

- 內建 TDD，但是工程師仍要學習其觀念
- 降低學習門檻，可以用更 Clean 的方式實作代碼
- 全權委託，但是工程師要對架構負責(變得更像 SA ?)
- 淘汰劣質工程師，ex 轉職班、就業班、刷題仔，但是有的很會行銷自已，會被唬弄過去

對產品而言

- 可以縮短想法到實作的距離
- 保持一定的品質，特別是你的工程師都是來自x成、x匠時…
- 自已開發，更高的回饋

問題，認知的差距，人只能賺到自已認知能力內的錢

之前有一個網紅行銷使用 Agent 開發賣課，犯了最低級的錯誤

這是因為他缺乏資安的認知; 所以原本不是開發的人員會需要快速的學習相關的知識(但是 AI 可以將門檻降低)

另外一些反思，那種工程師看不上眼的程式(普遍社群上的工程師都在嘲諷)，

人家已經拿來行銷賣錢，或許工程師應該反向思考怎麼朝行銷/產品靠攏

也或許這個低級錯誤也是行銷手段的一環?

接下來， 我將會用一系列的文章來試作 SDD 並分享我的看法

## 參考

- [隕石開發流程](https://eiki.hatenablog.jp/entry/meteo_fall)
- [隕石開發流程-中文版](https://bclin.tw/2018/06/01/meteo-fall/)
- [Martin Fowler 澳洲演講筆記-變質的敏捷，回歸初衷。](https://martinfowler.com/articles/agile-aus-2018.html)
- [軟體工匠的悲歌 The Tragedy of Craftsmanship.](https://blog.cleancoder.com/uncle-bob/2018/08/28/CraftsmanshipMovement.html)

(fin)
