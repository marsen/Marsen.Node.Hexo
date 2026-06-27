---
title: "[SDD 與 Spec Kit]敏捷開發（2000-Now）"
date: 2025/11/03 11:02:36
tags:
  - 學習筆記
---

## 輕量級方法百花齊放

由於原始工程的冗長沉重，各式輕量的方法被提出，

XP/Scrum/RAD(Rapid Application Development)/DSDM（Dynamic Systems Development Method/Crystal 等等…

[Scrum](https://zh.wikipedia.org/zh-tw/Scrum) 更是自 1986 就提出，雖然到 1995 才整理成為完整的方法論。

XP 則是 Kent Beck 於 1996 年在克萊斯勒專案開始的第一個完整實驗案例。

## 敏捷宣言

2001年2月11-13日，17位軟體開發者在猶他州雪鳥滑雪場寫下敏捷宣言。

這不是隨便17個人——他們每一位都是當時軟體開發方法論的領導者。

這17位是：

| 分類                  | 代表人物                             | 主要著作／理念                                                 | 切入角度 | 貢獻重點                               |
| ------------------- | -------------------------------- | ------------------------------------------------------- | ---- | ---------------------------------- |
| **一、極限編程（XP）與工匠精神** | **Kent Beck**                    | 《Extreme Programming Explained》 (1999)                  | 技術實踐 | 以測試驅動（TDD）、重構、持續整合為核心，建立可持續變更的技術文化 |
|                     | **Ron Jeffries**                 | 「You Aren’t Gonna Need It」(YAGNI)                       | 極簡哲學 | 實踐 XP 原則於 C3 專案，推廣極簡與實用的設計理念       |
|                     | **Martin Fowler**                | 《Refactoring》(1999)、〈Continuous Integration〉            | 軟體架構 | 推動可演化架構、自動化測試與持續整合文化               |
|                     | **Robert C. Martin (Uncle Bob)** | 《Clean Code》系列、SOLID 原則                                 | 工匠精神 | 提倡程式專業紀律、代碼品質與職業倫理                 |
|                     | **James Grenning**               | 嵌入式 TDD 推廣者                                             | 硬體開發 | 將 XP 精神帶入嵌入式與硬體領域，重視可靠性與可測性        |
|                     | **Andrew Hunt & Dave Thomas**    | 《The Pragmatic Programmer》(1999)                        | 實用主義 | 提倡開發者自我改進與實用思維，落實工匠式開發文化           |
| **二、Scrum 與組織敏捷管理** | **Ken Schwaber**                 | 《Agile Software Development with Scrum》(2001)           | 流程管理 | 建立 Scrum 框架，確立三角色與三工件              |
|                     | **Jeff Sutherland**              | Scrum 共同創始人、「Inspect & Adapt」理念                         | 組織文化 | 建立迭代循環與回饋節奏，強調可視化管理與持續改進           |
|                     | **Mike Beedle**                  | Scrum 實踐推廣者                                             | 企業轉型 | 將 Scrum 落地於企業場景，推動組織敏捷轉型           |
| **三、人本導向與情境化方法**    | **Alistair Cockburn**            | Crystal 方法、《Writing Effective Use Cases》(2000)          | 人際互動 | 從人與溝通出發，倡導情境化與最小可行流程               |
|                     | **Jim Highsmith**                | 《Adaptive Software Development》(1999)                   | 學習導向 | 將開發視為學習與演化過程，重視適應性與協作              |
|                     | **Brian Marick**                 | Context-Driven Testing 理念                               | 測試文化 | 測試應依專案情境調整，反對僵化流程與統一標準             |
| **四、模型導向與架構思維**     | **Steve Mellor**                 | 《Object-Oriented Systems Analysis》(1988)、Executable UML | 模型驅動 | 面向物件分析先驅，後期推動模型驅動架構（MDA）與可執行模型概念   |
| **五、組織級與治理導向方法**    | **Arie van Bennekum**            | DSDM 方法共同創始人                                            | 專案治理 | 從企業治理與預算控制出發，平衡交付靈活性               |
|                     | **Jon Kern**                     | Feature-Driven Development (FDD) 實踐者                    | 大型專案 | 以功能特徵為單位規劃開發，適用大型分散式團隊             |
| **（補充人物）**          | **Ward Cunningham**              | Wiki、Pattern Repository                                 | 團隊學習 | 推動知識共享、模式化思維與團隊學習，是 XP 精神的早期基礎     |

### 簽署內容

> Individuals and interactions over processes and tools
>
> Working software over comprehensive documentation
>
> Customer collaboration over contract negotiation
>
> Responding to change over following a plan

## 敏捷的變質：從理想到現實

簽署宣言20多年後，敏捷已經成為主流，但也出現了嚴重的變質。

有一些問題，我先列在這著，其他的下一篇再說明

- 儀式化的敏捷
- 認證機類與產業鏈的興起
- 工程實踐的缺／消失
- 文化衝突與假敏捷

### 結果是各種「偽敏捷」

小瀑布式 Scrum：兩週一個瀑布，需求分析→設計→開發→測試，只是週期變短

命令式敏捷：「我們要敏捷！」但所有決定還是老闆說了算

報告式敏捷：Jira 票開得很勤，但主要是為了產生報表給管理層看

自得其樂的 Scrum But：守破離本身的概念沒錯，但是守得人少，離的人多，沒見過什麼有效的破

### 簽署者們的反思

- Ron Jeffries 在2018年的文章《Developers Should Abandon Agile》中寫道：「如果你的組織扭曲了敏捷，讓它變成壓迫開發者的工具，那就放棄它，專注於寫好程式。」
- Dave Thomas 在2014年的演講《Agile is Dead》中說：「敏捷應該是形容詞（Agile），不是名詞（Agile）。你應該追求敏捷性，而不是實施大寫的Agile。」
- Robert Martin 在《Clean Agile》(2019)書中直言：「敏捷工業複合體（Agile Industrial Complex）已經形成，充斥著證照、教練、顧問，但失去了原始精神。」
- Kent Beck在2019年的訪談中提到，XP的核心是五個價值觀：溝通、簡單、回饋、勇氣、尊重。這些價值觀驅動實踐，而不是反過來。

這些先驅者從來不是教條主義者，他們一直在實驗、調整、演化。

台灣軟體業面對的不只是方法論的選擇，還有文化適應的挑戰。

沒有銀彈，正如 Alistair Cockburn 在 Crystal 方法中強調的：每個專案都是獨特的，需要調整方法來適應團隊和環境。

重要的不是完美複製矽谷的敏捷，而是找到適合我們的方式。

理解這 17 位先驅者的初衷——更好的軟體開發方式 —— 比死守任何框架都重要。

## 參考資料

- [敏捷宣言 Agile Manifesto](https://agilemanifesto.org/)
- [敏捷已死 Agile is Dead • Pragmatic Dave Thomas • GOTO 2015](https://www.youtube.com/watch?v=a-BOSpxYJ9M)
- [我们都只是在假装着做Agile【让编程再次伟大#43】](https://www.youtube.com/watch?v=vJGoQigiXAE)
- [軟爛的 Scrum Flaccid Scrum](https://martinfowler.com/bliki/FlaccidScrum.html)
- [敏捷報告 The 17th State of Agile Report](/assets/RE-SA-17th-Annual-State-Of-Agile-Report.pdf)

(fin)
