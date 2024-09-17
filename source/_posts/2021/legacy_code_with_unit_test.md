---
title: " [上課筆記] 針對遺留代碼加入單元測試的藝術 "
date: 2021/01/11 17:07:35
tags:
    - Unit Testing
---

## 題目

- 聽到需求的時候，不要先想邏輯，先想怎麼驗收 ?
- OO 善用封裝繼承的特性，不要修改 Product 類別，不要修改測試方法
- Extract and Override 的 SOP
  - 找到 Dependency
  - Extract Method
  - private → protected
  - 建立 SUT 的子類
  - Override Dependency
  - Create Setter
  - 改測試
  - Assign 依賴值
  - 無法適用的情境
    - final
    - static
- SPR 的 anti Pattern
  - 職責過多
  - 單元切得過細

- 測試案例為什麼用 Snake 寫法?
  - 測試案例是用來描述情境
  - 反思：中文表達力強但描述精準度低
  - 要看見測試案例的因果關係
  - 測試案例的設計
    - 2+2 = 4
    - 2x2 = 4
    - 2^2 = 4

- 重構測試案例
  - 只呈現與情境相關的資訊

- 養成用 Domain 情境而不是 Code Level 的語言
- 不要為了測試把 Production Code 搞得複雜
- Mock Framework 的本質與要解決的問題是什麼
- 3A原則與Given When Then
- 減少 Content Switch(利用工具)
- ~~錢~~時間要花在刀口上
  - Unit Test
  - Pair Programming
  - Code Review
  - 整合測試
  - QA 測試
  - Alpha/Beta 測試
  - 部署策略

- Test Anti-Patter
  - 小心過度指定(使用 Argument Matcher
  - 測試案例之間相依
  - 一次測不只一件事（Data Driven)
  - 測試案例名字過於實作細節

- 不要為了寫測試使用 Virtual
- StoreProcedure 的測試不是單元測試，但是很好測試，只有與 Table 相依，透過 ORM 可以輕易作到，

## Nice to Have

- Coverage
  - Missing Test Cases
  - Dead Code
  - 實例分享：大家都不知道*
  - 童子軍法則(趨勢>數字)
  - \> 0%
- ROI
  - 出過問題
  - 經常變動的
    - 對不會變動且運作正確的程式寫測試是種浪費
    - 共用的模組
    - 商業價值

- 架構設計
  - 三層式架構
  - 六角架構

## 課程總覽與個人的建議順序

| 課程 | 說明 | 補充 |
| -------- | -------- | -------- |
| 熱血 Coding Dojo 活動  | 點燃動機的一堂課     | 對我個人影響最大的一堂課  |
| 極速開發     | 學過這堂課，會比較清楚 91 的日常開發是怎麼作的 | 這裡是一個檻，你需要學習 Vim 也可能需要買好一點的 IDE  |
| 針對遺留代碼加入單元測試的藝術    | 單元測試的基本概念  | 強烈建議讀過「單元測試的藝術」與 91的「30天快速上手TDD」雖然有點久遠，但是好東西是經得起年歲的 |
| 演化式設計：測試驅動開發與持續重構   | 2日課程，如果只能上一堂課的話，我會選這堂    | 資訊量很大的課程，不過如果上過前面的課，這時候應該可以苦盡甘來      |
| Clean Coder：DI 與 AOP 進階實戰   | 進階課程  | 如果上過前面的課，這時候應該可以苦盡甘來，但是建議可以對 Design Patter 稍作功課  |
|      |      |      |

## 反思:如果是我會怎麼設計內訓課程?

- 附錄課程
  - 為什麼寫測試?
  - 第一個測試(計算器或 FizzBuzz)
  - 5 種假物件(Test Double)
    - Stub(模擬回傳)/Mock(前置驗証)/Fake/Spy:驗証互動(後置驗証)/Dummy
    - 小心不要落入名詞解釋

- 主要課程（假設學生已經有一定的理解）
  - 第一個測試
    - 3A原則
  - 3正1反
  - 三種互動
    - 呼叫次數，參數，順序
    - 狀態改變
    - 回傳值
  - 隔離相依  
    - 原生操作
    - 使用框架
    - 測試案例
      - 可讀性:增進理解 / 規格書 / 穿越時空的傳承
  - TDD
  - 重構
  - Design Pattern

## 課程上的反思

- Setter 的重構要加強
- 依賴注入點，不一定要在建構子也不一定要在 Prod (抽方法)
- 要有能力判斷切入點
  - 善用工具
  - 練習
- C# 參數善用Optional
- OOP & UT
- Fp & UT
- UT & Refactor & Design Pattern
- Pattern 工廠／簡單工廠／
- 存在監控 Running Code 計算方法呼叫次數的工具嗎 ?
- 快速打出六個０ ?

## 參考

- [Repository](https://github.com/202101-unittest)
- [Paper](https://paper.dropbox.com/doc/202101-ERimcc1zVeIpED6vZjLtU)

(fin)
