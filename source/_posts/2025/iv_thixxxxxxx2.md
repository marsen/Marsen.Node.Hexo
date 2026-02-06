---
title: "[生活筆記] 2025 T社面試心得 - 二面"
date: 2025/12/08 22:18:41
---

## 前情提要

延續[一面的紀錄](https://blog.marsen.me/2025/11/28/2025/iv_thixxxxxxx/)，在完成 Python 基礎與演算法測驗後，很快地收到了二面的邀請。  

二面同樣採用線上面試的形式，主要聚焦在演算法題目的實作與程式碼重構能力的考察， 
 
並且會有主管線上監考。

## 面試流程

二面總時長約 90 分鐘，分為兩個部分：

1. **演算法題目實作**（約 60 分鐘）
2. **程式碼重構討論**（約 30 分鐘）

## 題目與解題

### [Q1: 最長不重複子字串](https://leetcode.com/problems/longest-substring-without-repeating-characters/description/)

**題目描述**：給定一個字串 s，找出最長的不含重複字元的子字串長度。

**解題思路**：

- 使用 sliding window 的概念
- 用 `strtmp` 維護當前不重複的子字串
- 當遇到重複字元時，清空 `strtmp` 重新開始
- 持續追蹤最大長度

---

### [Q2: 股票交易最大利潤](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/description/)

**題目描述**：給定股票每日價格陣列 prices 和交易手續費 fee，求最大利潤。  
可進行多次交易，但每次交易需支付一次手續費，且賣出後才能再買入。

#### Q2 解法評價與反思

**坦白說**，這題在本來想展示 TDD 的過程，  

並在過程中覺察重複，並使用重構技巧完成解答。

但可能弄巧成拙，看起來只是把測試案例的答案硬編碼進去。

**這個策略的問題**：

- ✅ **優點**：至少讓測試通過，展示我理解題意
- ❌ **缺點**：面試官會認為我不會動態規劃，或者時間管理有問題
- ⚠️ **風險**：如果面試官追加測試案例，立刻露餡

未來再用 TDD 來解上面兩題

---

### Q3: 程式碼重構

**題目背景**：

這是一個批次匯入資料的函式，會有效能與欄位合併的問題，需要重構：

**原始程式碼，基於對面試公司的尊重，不予公佈**：

我的重構建議：

1. config 敏感資料
2. config MAX_RECORD_PER_ROUND
3. 拆單一職責子方法(取資料/組 SQL/寫資料)
4. SQL BULK
5. 重命名參數

#### Q3.解法評價與反思

面試官更著重在使用 python 的語法使用
如下，

| 語法糖 | 用途 | 優勢 |
| -------- | ------ | ------ |
| `**dict` | 字典解包 | 簡化參數傳遞 |
| `with` | Context Manager | 自動資源管理 |
| `f-string` | 字串格式化 | 可讀性高 |
| `sql.SQL()` | SQL 組裝 | 防止 SQL Injection |
| List Comprehension | 列表生成 | 簡潔、快速 |
| Generator | 惰性求值 | 省記憶體 |
| `:=` Walrus | 賦值表達式 | 減少重複運算 |
| `@dataclass` | 資料類別 | 減少 boilerplate |
| `@contextmanager` | 自訂 CM | 封裝 setup/teardown |
| Type Hints | 型別註記 | IDE 支援、可讀性 |

## 面試心得與反思

從 Web 2.0 到 AI 時代，工作方式已經改變：

**Web 2.0 時代**：手寫演算法是基本功，熟悉語法是專業展現

**AI 時代**：演算法交給 AI，重點是整合工具。我的工作流程：TDD + AI → Review → 上線＆維運

往前有需求分析與設計，往後有系統維運，之間交錯的成本控管／資安／流程優化，都是在開發之外的能力

但這次面試仍在考：能不能手寫 LeetCode、熟不熟 Python 語法、時間壓力下能不能寫出正確代碼。

### 角色認知的斷層

**Senior Engineer**：LeetCode 能力 = 技術能力

**Tech Lead**：系統思維 = 技術決策能力，整合能力 = 創造價值

這次面試用 **Senior Engineer 的標準**在評估 **Tech Lead 的職位**。

我用 Tech Lead 這個角色，應徵 Senior Engineer。

### 雙向選擇

面試不只是公司選我，我也在選公司，可惜沒有給我問問題的機會：

- 他們看重：LeetCode、語法(糖)熟悉度、coding speed
- 我看重：系統設計討論、技術決策空間、團隊文化

還在用舊時代標準評估新時代角色的公司，或許不是我的理想選擇。

而我的多語言開發背景，可能也只是 jack of all trades master of none

## 參考資料

- [一面面試記錄](https://blog.marsen.me/2025/11/28/2025/iv_thixxxxxxx/)
- [LeetCode - Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)
- [LeetCode - Best Time to Buy and Sell Stock with Transaction Fee](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

(fin)
