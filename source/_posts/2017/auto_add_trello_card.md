---
title: 自動化 Trello 操作
date: 2017/02/08 23:52:27 
tag:
  - Trello
---

Trello 是一個非常方便的工作管理工具, 
最主要的功能只有 Boards 、 Lists 與 Cards,
確可以發揮相當大的綜效功能,  
用來當作敏捷開發的白板、安排工作項目與進度,
作為協作的平台與工具。
也有相當多元的外掛可以供不同的情境使用,
開發人員也可以自行串接API與系統作整合。
簡單記錄自動化產生 Cards 的兩種方法。

## 第一種方法,使用mail

1. 開啟 Menu > More
2. 點選 Email-to-board Settings
3. 選擇你要產生卡片的 List 與 位置(頂部或底部)

![](https://i.imgur.com/PqDLtdO.gif)

4. 寄信，信件格式如下
	- email 的 subject 會成為卡片的標題
	- email 的內容會成為卡片的描述
	- 如果有附加檔案在郵件中，會附加到卡片中
	- 在 subject 加入 `#label` 可以在卡片加入標籤(Labels)
	- 在 subject 加入 `@member` 可以在卡片加入成員(Members)


更多請[參考](http://help.trello.com/article/809-creating-cards-by-email). 

## 第二種方法,使用API
1. 登入 Trello
2. 連線 https://developers.trello.com
點選`Get your Application Key`. 連線到 https://trello.com/app-key

### 取得Key
![](https://i.imgur.com/bBoiUCr.jpg)

### 生成Token

![](https://i.imgur.com/gHsdYv8.jpg)


![](https://i.imgur.com/bSXkChk.jpg)
點選allow後，就會顯示你的token，特別注意登入的身份權限，
並且千萬不要外洩你的token與key值。

試打API,建立一張 Card 到指定的 List 中.
並且設定期限(due)與標籤(labels),
更多的API設定請[參考](https://developers.trello.com/advanced-reference).
![](https://i.imgur.com/yk6SZYm.jpg)

## 分析
- Email:
	- 優點:簡單、方便、可以結合mail system 、附加檔案簡單
	- 缺點:部份功能無法實現(ex: 設定due date)
- API:
	- 優點:靈活、Resful的API可以實現大部份的功能
	- 缺點:實作比較麻煩

(fin)	

