---
title: "[活動筆記]蝦皮購物新加坡研發團隊技術分享會"
date: 2018/03/15 17:00:12
---

## 應該不用知道的事
1. 雖然是「技術分享會」實際上在徵才
2. 不過還是有半場的技術分享
3. 91app 至少去了10 個人(含前員工) 
4. 這篇文章對你應該沒有幫助

## 有關蝦皮
- 屬於[Sea 集團](http://www.seagroup.com/home)的一部份
- 東南亞多國服務(新加坡、泰國、馬來西亞、印度、台灣、越南…)
- 63e Request / Day
- 8G IO / Mins

## 選擇
- Native App / Web / Hybird / RN ?
- Clound / Self machine ?
- Php/ Nodejs / RoR / Django ?
- Apache / Ngnix ?
- C ++ / Java / GoLang
- Memcahed / Redis ?
- SPA / MPA ?
- Mesos / Kubernate ?

## Qiz & Ans

##### 1
> 用戶下單的時候, 先收錢還是先扣庫存?
> 扣掉最後一件庫存後, 收錢失敗怎麼辦？
> 你已經把「賣完」訊息發給了賣家, 怎麼辦？

##### 2
> 計算金額用整數還是浮點數？(浮點數不準)

##### 3
> Android 一共有幾種螢幕的 DPI ?
> Android WebView 和 Chrome 的 Webkit 有何不同 ?

##### 4
> Web Service 花最多時間在處理什麼 ? 
> 如何壓搾最高的吞吐量 ?
> IO, USE async

##### 5
> 什麼樣的情境適合增加伺服器數量來增進效能?
> stateless
> 那有狀態怎麼辦 ? 

##### 6
> load balancer 效能到達瓶頸怎麼辦 ?
> IP

##### 7
> 一天 25TB 的 Log 數量,怎麼不會查到天荒地老

##### 8
> Cache & 超賣問題
> 什麼時候要清 Cache ?

##### 9
> Database Master 與 Slave 哪個壓力大 ?(Slave)
> 增加 index 的代價為何 ?(Space)
> Table 多大要 shard ?
> Database 多大要分庫 ?
> 分庫如何作 transaction ?

## 實踐

1. Prototype 簡單 Production 困難 (邊際效應/熵)
2. 可靠:言出必行,作不到也要早點說(知難行易)
3. Redis 的資料超過 64G 就無法用 [bgsave](http://redisdoc.com/server/bgsave.html) 有效存檔
4. 在 Prodction 千萬別用 Redis 的 [key](https://redis.io/commands/keys) 指令
5. 衡量的基準(benchmark)為何？
6. 不要對邏輯下command(不要寫前因後果)
	- Dont command How
	- Command Why
7. 道
    - Collect your dots first
    - Connecting the dots


## 持久發展的研發團隊
- knowledge
- 保持開放
- 尊重事實
- 信任
- 可靠
- 找到根本原因(root cause)
- 分析 修復 記錄
- Docs
	- connection docs
	- collection docs

## 測試
- 白箱測試
- 黑箱測試


## 其它
- Hypergraph
- Tech Stack
- Roles
	- contries PM
	- fucntion PM
- Scurm 是跑給老闆看的(!!?)
- 馬來西亞不用小豬ICON(各地風俗民情不同)

## 參考
- https://careers.shopee.com

(fin)