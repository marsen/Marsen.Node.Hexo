---
title: "代碼審查與交付的戰爭ー標準、風格與原則"
date: 2018/01/04 00:51:27
tag:
  - Coding Standard
---
###### Coding Standard / Code Review / Pull Request & Delivery
![](https://i.imgur.com/tssraf0.jpg)
## 故事背景
1. 團隊的部署流程是 Github Flow 與 Git Flow 混用 , 給它起個名字叫 GG Flow 好了.
2. GG Flow 的過程需要開發人員需要透過 **Pull Request** 將修改推送給產品 
3. 擁有權限 Merge Pull Request 的成員被叫作 Reviewer
4. Reviewer 通常由較資深人員或部門主管擔任,所以通常有比較多的~~無用~~會議要開
5. Reviewer 在 Merge 之前需要作 **Code Review**
6. Reviewer 需要遵循 **Coding Standard** 作 Code Review

## 實務面臨的問題與副作用
#### Coding Standard 並不能考慮到所有狀況 

1. 所以 Reviewers 會定期針對不同的狀況開會討論 Coding Standard
    - Coding Standard 會**不定期改變** , 但是透過Reviewer佈達的方式,讓第一線的RD其實難以知道其全貌.
    - Coding Standard 改變後不會全面的翻改程式,實務上是作到哪裡改到哪裡
    - 以上兩點導致 Source Code 裡面有很多符合不同時期的 Coding Standard 的 Code
    - 任一個時間點, 誰都無法保証完全符合最新的 Coding Standard 
2. 人性,開發者會~~COPY/PASTE 方法開發~~參考Legacy Code開發
    - Legacy Code 不符合新的 Coding Standard
    - Reviewer 也是人, 所以 Code Review 時也會疏漏,而 Merge 進去不符合新的 Coding Standard 的 Code
    - 所以 Source Code 裡面還是有很多不同時期的 Coding Standard 的 Code
3. 回歸一開始的問題 Coding Standard 並不能考慮到所有狀況 
    - 還沒有開會前, 不同的 Reviewer 會有不同的想法
    - 開會後,在執行Code Review時, 不同的 Reviewer 會有不同的作法
    - 當一個 PR 有多人 Reviewer 時, 會有不同的意見 PR 因此被延遲 Merge
    - 結果,**交付會變慢**.

## 反思,標準還是風格？ 

思考一下,開發程式碼的目標與價值是什麼 ?
寫出 Clearn Code ? 
還是交付產品 ?
這樣子的 Source Code 真的是 Clearn Code 嗎？

## 自問自答

#### Q1. 我們該有標準嗎？
A1. 當然要有標準,不過標準之所以為標準,應該有以下幾個特點. 
- 它應該要很簡單, 像是Class與欄位的命名規則
- 它應放諸四海皆準, 不應該輕易被修改
- 它應該可以被自動化的檢測
假設能作到這3點, 這件事應該可以被自動化工具處理掉 . 

#### Q2. 實務上就是很複雜, 所以才需要討論制訂標準啊
A2. 
在實務上遇到很複雜的情況, 大多需要依賴約定成俗方式規範.
這是一種**風格**或**原則** ; 
簡單的分類方法, 
如果無法透過自動化工具作檢測, 
就不應該歸類為**標準**.

*註:有機會再介紹自動化的檢測工具*

#### Q3. **風格**或**原則**跟**標準**有何不同？
A3. 如上所說,標準應該能被自動化,
風格應該是團隊的文化自然形成的產物, 
具體的實作可以透過讓開發者**彼此之間作代碼審核**
或是**結對編程**培養出屬於團隊的風格,
風格要基於標準之上,但是不能違反原則;

以下的原則可以作為參考
- 可以建置並通過測試
- 可讀性
	- self documenting
	- 有用的註解 
- 公開方法要可以被測試
	- 小心使用靜態類別
	- 注意new Instance的時機
	- 重複的代碼應重構
- 保持 SOLID 

初期的可能會發生在「{」要不要換行之類的問題上揪結之類的蠢事,  
如果可以自動化,就把它作成標準吧…  
如果不行的話, 就別揪結了.  

實務上可能遇到各種狀況,  
把Reviewer的權限下放到各個開發者身上,  
或是使用結對編程,  
就讓團隊成員去討論與決定風格.  

以標準為根基,原則為天,  
踩穩腳步,不要超出天空,  
就讓團隊自由發揮吧. 

**最後,持續交付會比每兩周花一個小時開會決定Style的細節好多了. 不是嗎？**

## 其它團隊分享的具體作法

1. 超過一定時間就讓成員擁Merge權限
2. Release權限仍集中控管
3. 錯了再改就好(保持敏捷)
4. 給pair作code review與merge (避免一人思維陷井)
5. 兩個人無法解決時找第三方
6. release 功能 優先於 一致的 coding standard
7. 品質由測試管控而非 reviewer
8. 先有測試才有重構
9. 可讀性 優於 枝微末節的coding standard實踐
10. 善用自動化工具( sonarqube / stylecop )

(fin)

##### 補充 [社群觀點](https://www.facebook.com/groups/616369245163622/permalink/1225873964213144/)

- coding style一般不管的。
- class name／variable name，一定要叫有意義的名字。
- local scope variable，換多少行，indentation，這些是小事
- 一個成熟的developer，隨時會被上司命令這些遠古火星文明（legacy system）去做考古工作
- coding style這些事，就像emacs和vim之戰一樣，戰到skynet出來了也不會戰完的
- 如果要開會去討論coding style，最終很可能讓團隊口服心不服地去跟隨我的Coding Style。
- 在Coding Style這種低層次的小事上用光了團隊成員之間互相容忍的能量，而在更重要的大事上無法好好合作。
- 有很多事是比Coding Style重要的。
	- Object Modeling是否跟business logic一致？
	- 還是Object有這個attribute但是根本沒在用？
	- Code Change是否有做好測試？
	- 系統架構是否合理
	- 有做好High-Avalibility嗎？
	- 有沒有Race Condition？
- 是其是，非其非。真正有道理的，你說了對方便自然會聽下去。
- 「Senior」是代表自己在專業上懂得比別人多，而不是比別人身份高級。