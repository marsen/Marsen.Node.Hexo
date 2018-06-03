---
title: "[活動筆記]變異測試 - 一種改進測試和代碼的 「新」 方法"
date: 2018/03/20 01:44:00
tag:
  - Testing
  - TDD
  - Unit Testing
  - Mutation Teseting
---

## 應該知道的事
- 範例使用Java
- 這場活動使用人肉找尋變異
- 實務上應使用工具
- 但是不能完全相信工具
- [活動聯結](https://www.facebook.com/events/966762773499308/)
- 講師是 Odd-e 的姚若舟
- [簡報preview版](http://boolan.com/lecture/1000001089)

## 什麼是變異 ?

### 前言
想像一下產品(`Prodction`)就是你的身體,  
我們可以透過健康檢查(`Unit Test`);  
檢查你的身體有沒有異狀 ?  

但是檢查真的可靠嗎 ?  
比如說一般的流感的快篩只有50~60%的準確率,  
我們的測試也無法達到100%準確率(這裡不是指覆蓋率喔).  
如何抓到測試抓不到的**漏網之魚**就是變異測試的目的.  

我們透過讓 Prodction 產生變異(Mutation)  
來確認我們的 Unit Test 是否可靠.

>題外話,當大流行的時候會跳過快篩節省醫療資源,  
因為可能有一半(50%)的患者都是流感,  
而快篩準確率也只有50%,加上時間及醫材成本,  
不如直接開克流感能有效抑止疫情



### 變異測試(Mutation Testing)
變異後導致測試失敗？
**yes , good**
應該要失敗,表示你的測試有覆蓋到這個變異

**no , test not covered**
這表示你的測試並未

### 測試不一定能補捉變異
比如說 `邊際值` 或是 `隱含的互動`;  
測試覆蓋率100%也不一定能補捉變異  
要麼少了test case,  
要麼多了無意義的代碼  
看看以下例子  
ex:
```csharp
foo(x,y)
{
    //// logic here
    sideeffct();
    return z;
}
```

反思一下, 測試過了代碼就沒問題 ?  
不能捉到變異的測試,  
有發揮它的功能嗎 ?  
一般來說如果透過 TDD 進行軟體開發,  
我們的測試應該是會恰巧符合一項 Test Case  
而如果是先寫代碼再寫測試,  
將很難通過變異測試(容易產生多餘的代碼)

### 找到變異的幾個方向
- 邊界條件(`<` => `<=`)
- 反向條件(`<` => `>`)
- 移除條件(永真/永偽)
- 數學
- 遞增/遞減
- 常量
- 返回值
- 移除代碼

### 先寫代碼再寫測試有問題是很正常的

## Kata-PokerHands 範例

### [原碼(使用java)](https://github.com/JosephYao/Kata-PokerHands)

### 變異實例
有問題 ,反向測試案例不足

```java
private	List<Integer> getPairCardRanks(List<Integer> cardRanks) {
         List<Integer> result = new ArrayList<>();
         for (int index = 0; index < CARD_COUNT - 1; index++)
         	if (isTwoNeighborCardRanksEquals(index, cardRanks))
         		result.add(cardRanks.get(index));
         return result;
}
```

有問題 , -1 但是預期中的行為
```java
protected   Integer   getThreeOfAKindCardRank(List<Integer\> cardRanks) {
     for (int index = 0; index < CARD_COUNT - 2; index++)
     	if (isThreeNeighborCardRanksEquals(index, cardRanks))
     		return   cardRanks.get(index);
     throw   new   IllegalStateException();
}
```
## 其它
1. 沒有TDD 沒有單元測試,別跑變異測試
2. 至少要有行級別的覆蓋率(line coverage)
3. 分支覆蓋(Branch Coverage)好一點 仍不夠
4. 在需求不變的情況下，再作變異測試
5. 以變異測試的角度來說,覆蓋率100%是木有用的(testing coverage is useless)
6. 發現變異怎麼辦？
	- 報告(記錄)
	- 重現 
	- 評估
	- 修改 或 補測試
7. 依靠工具不要相信工具,上一步的評估
Ex: mock 物件會取代互動實際的行為,導致變異測試失敗



### Tools
- http://pitest.org
- https://en.wikipedia.org/wiki/Mutation_testing
- 使用 Sonarqube with mulations test(應該不用錢)
- tudou.com/home/yaoruozhou

## 參與者心得
1. [變異測試 (Mutation Test) — 一種提高測試和代碼質量的 ”新” 方法速記](https://medium.com/@loverjersey/變異測試-mutation-test-一種提高測試和代碼質量的-新-方法速記-35bde79a5c7a)

2. [Test - 變異(Mutation)測試之你的測試到底是寫爽的，還是有效的?](https://dotblogs.com.tw/im_sqz777/2018/03/15/004634)

## 心得
1. 佩服當天就能寫出文章的人
2. 變異測試是好上加好的測試
3. Odd-e 的講師真的很粉棒, 雖然不致到毀三觀 不過眼界大開


## 參考
1. [Mutation Testing(变异测试)](http://www.cnblogs.com/TongWee/p/4505289.html)

(fin)