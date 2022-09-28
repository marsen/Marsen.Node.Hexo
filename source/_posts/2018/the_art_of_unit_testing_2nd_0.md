---
title: "[讀書會] 單元測試的藝術 - 導讀、序與第一章"
date: 2018/03/22 01:20:12
tag:
  - TDD
  - Unit Testing
  - Testing
---

## 要知道的事

1. 這是[單元測試的藝術](http://www.books.com.tw/products/0010765689)的閱讀筆記
2. 筆記的意思就是不一定會有心得
3. 這篇主要是導讀

## 譯者序

- TDD , Test First to Think First
- 什麼是好的單元測試？
- 單元測試三支柱:可信任 可讀性 可維護
- 綠色安全區域
- 實務上導入的指引

### 入門建議

- 了解如何隔離相依(Part II)
- Stub 與 Mock 的差異,熟練隔離框架(NSubstitute)
- 如何撰寫優秀的單元測試(Part III)

### 進階建議

- 如何撰寫優秀的單元測試(Part III)
- 如何在組織中導入單元測試(Part IV)
- 針對遺留代碼的重構與測試,以及可測試性設計(Part IV)

### 避免

1. 測試不穩定
2. 過度指定
3. 一次不只測一件事
4. 測試程式重複過多
5. 可讀性差

## 關於本書

### 前言

有寫測試, 也不保証專案成功,  
一個失敗的單元測試案例,  
作者歸納原因如下,

- 脆弱的測試(Prod 改一點,測試就錯一大片)
- 不易維護
- 測試間相護依賴
- 可讀性差

### 作者推薦的框架

- [NSubstitute](http://nsubstitute.github.io)
- [FakeItEasy](https://github.com/FakeItEasy/FakeItEasy)

### 學習路線圖

- Part I 基礎知識
- Part II 測試框架
- Part III 最佳實踐
- Part IV 組識導入/遺留代碼/設計

## 目錄

1. 入門
   - 什麼是優秀的單元測試
   - 單元測試與整合測試的分別
   - 第一個單元測試
2. 核心技術
   - Stub
   - IoC(DI)
   - 值、狀態與互動
   - 測試框架
   - 事件
   - 深入了解測試框架
3. 測試程式碼
   - 自動化
   - 綠色安全區域
   - 可信任/可維護/可讀性
4. 設計與流程
   - 組織導入
   - 遺留代碼
   - 設計與可測試性

## 定義單元測試

### 什麼是優秀的單元測試

1. 自動化, 可重複執行
2. 容易實現\*
3. 到第二天還有存在的意義(非臨時性的,ex:hotfix)
4. 任何都可以一鍵執行
5. 執行速度快
6. 結果一致
7. 可以完全控制(不與外部相依)
8. 獨立於其他測試
9. 失敗時,錯誤應該是明確的

### 整合測試

1. 整合測試相依於真實物件
2. 整合測試的結果不穩定
3. 整合測試與單元測試應該被分開(見 ch7.2.2)
4. 整合測試執行時間長
5. 依據現實狀況無法完全控制
6. 缺點: 一次測試的東西太多

### 理解測試趨動開發

1. TDD 不保證產品會成功
2. 步驟
   1. 寫一個失敗的測試
   2. 寫一個符合測試預期的產品程式碼,以通過測試
   3. 重構

### TDD 的核心技能

1. 可維護、可讀、可靠(這本書的目的)
2. 寫出可維護、可讀、可靠的測試不等於 TDD,至於如何寫優秀的 TDD,作者推薦閱讀[〈Test-Driven Development:by Example〉](https://www.tenlong.com.tw/products/9780321146533)
3. 就算執行 TDD,也不保証能設計一個完善的系統,作者推薦閱讀[Growing Object-Oriented Software, Guided by Tests](http://tl.big5.zxhsd.com/kgsm/ts/big5/2010/07/30/1801246.shtml)與[無瑕的程式碼](https://www.tenlong.com.tw/products/9789862017050)

簡單說就是:

- 寫好測試
- 測試先行(TDD)
- 設計

作者認為這是三種技能, 同時學習三種技能門檻會相當的高, 最後導致放棄.

### 小結

- 優秀的測試就是
  - 自動化
  - 容易撰寫
  - 執行快速
  - 任何人都可以執行,並得到相同結果

## 揪錯

![印刷錯誤](https://i.imgur.com/olnQxQ2.jpg)

## 本書資源

1. [Samples](https://github.com/royosherove/aout2)
2. [The Art Of Unit Testing](http://artofunittesting.com/)
3. [Videos](http://osherove.com/videos/)

(fin)
