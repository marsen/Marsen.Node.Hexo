---
title: "單元測試與重構記錄(一)"
date: 2017/10/31 00:42:10
tag:
  - Unit Testing
  - Integrated Testing
---

## 前情提要
有幸參與了一個跨國的專案,  
為了快速上線，決定將整套原本在台灣程式碼搬移到跨國專案上,  
上線後再依使用者的需求調整開發功能, 
而在搬移的過程中,有需多模組並未開啟。
…

## 現況
遺留代碼 → 跨國 遇到的問題
1. Copy Paste 最快卻不一定最適合
2. Copy Paste 最快卻不一定改得動
3. Copy Paste 最快但是有的地方沒Copy完

`用明朝的劍，斬清朝的官`

## 實務需求
將本來跨國未開啟的**折扣活動**模組打開,  
簡單的流程大致如下:
購物車 → 取得購物車資料 → 折扣活動 → 計算

實務上,整個流程作了許多事
![](https://i.imgur.com/pM36Joo.jpg)

應該說作了太多事. 
![](https://i.imgur.com/NU0PqCh.jpg)

程式碼有壞味道,確不能修改(重構). 
因為沒有測試保護. 

單一的Process，複雜度過高的方法(12)

`CalculateShoppingCartPromotionDiscountV2Processor.Process()`

![](https://i.imgur.com/qleqGPb.jpg)

### 目標與執行順序
1. 由 PM 或 QA 補足整合測試情境到足夠
    - 由實務上的需求來認定
2. 刪除台灣的測試
3. 解析 `CalculateShoppingCartPromotionDiscountV2Processor` 
4. 補上單元測試
    - Code Coverage(測試覆蓋率)
5. 重構

## 最終的目標是重構

- 心態:[沒有時間，完美的借口](http://www.danielteng.com/2012/09/25/no-time-to-learn-perfect-excuse/)
- 重構前要先作整合測試
- 現有的整合測試的缺陷
    1. 測試項目不符合馬來西亞現狀
    2. 測試項目未處理多語系
    3. 測試項目未處理小數點
    4. 測試項目難以閱讀
    5. 測試項目有重覆的覆蓋範圍
- RD與PM與QA合作

### UAT 讓「人」讀得懂

#### 原本的 UAT (RD)
```gherkin
場景: case37：商品活動+全店活動(有排除)；宅配，運費50元，滿350免運
	．全店活動 / 有排除商品；滿額打折，多階(滿第1階)，跨溫層
	．折扣條件：滿199元，打95折 / 滿299元，打89折 / 滿399元，打84折
	．商品活動；滿件折現，單階，跨溫層
	．折扣條件：滿2件，折45元
	假設 購物車中溫層"Freezer"商品為
		| SalePageId | SaleProductSKUId | Price | Qty |
		| 50         | 50               | 75    | 1   |
		| 27         | 27               | 66    | 2   |
	並且 購物車中溫層"Refrigerator"商品為
		| SalePageId | SaleProductSKUId | Price | Qty |
		| 26         | 26               | 55    | 2   |
	並且 購物車中溫層"Normal"商品為
		| SalePageId | SaleProductSKUId | Price | Qty |
		| 25         | 25               | 2     | 2   |
	並且 活動"1"範圍設定為
		| TargetType | TargetIdList |
		| Shop       | 1            |
	並且 活動目標排除商品頁為
		| PromotionId | TargetExcludeSalePageList |
		| 1           | 50          |
	並且  現折活動"1"的折扣為
		| Id | TypeDef      | TotalPrice | DiscountTypeDef | DiscountRate |
		| 1  | TotalPriceV2 | 199        | DiscountRate    | 0.95         |
		| 2  | TotalPriceV2 | 299        | DiscountRate    | 0.89         |
		| 3  | TotalPriceV2 | 399        | DiscountRate    | 0.84         |
	並且 活動"2"範圍設定為
		| TargetType        | TargetIdList |
		| PromotionSalePage | 0            |
	並且 活動目標商品頁為
		| PromotionId | TargetSalePageList |
		| 2           | 50,25,26,27        |
	並且  現折活動"2"的折扣為
		| Id | TypeDef    | TotalQty | DiscountTypeDef | DiscountPrice |
		| 4  | TotalQtyV2 | 2        | DiscountPrice   | 45            |
	當 計算活動折扣
	那麼 購物車商品折扣後為
		| SalePageId | SaleProductSKUId | PromotionDiscount |
		| 50         | 50               | -12               |
		| 27         | 27               | -25               |
		| 26         | 26               | -19               |
		| 25         | 25               | 0                 |
```

#### 「人」寫的UAT 

```gherkin=
場景: 商品有兩檔活動，全店活動與商品活動；
	．第一檔是全店活動 / 排除商品B；滿額打折，
	．折扣條件：滿10元，打95折 / 滿20元，打89折 / 滿30元，打84折
	．第二檔是指定商品；滿件折現，單階，指定商品A 、商品B
	．折扣條件：滿2件，折3元

當 購物車中的商品為"商品A 與商品B"
		| Title | SalePageId | SaleProductSKUId | Price | Qty |
		| 商品A   | 25         | 25               | 7.45  | 2   |
		| 商品B   | 26         | 26               | 4.45  | 2   |
	
並且 第"1"檔是全店活動 ,排除以下商品
	    | Title | SalePageId |
	    | 商品A   | 26         |

而且 第"1"檔折扣條件是"滿額打折,滿10元，打95折 / 滿20元，打89折 / 滿30元，打84折",如下
    | Id | TypeDef      | TotalPrice | DiscountTypeDef | DiscountRate |
    | 1  | TotalPriceV2 | 10         | DiscountRate    | 0.95         |
    | 2  | TotalPriceV2 | 20         | DiscountRate    | 0.89         |
    | 3  | TotalPriceV2 | 30         | DiscountRate    | 0.84         |

並且 第"2"檔是指定商品,指定商品如下
    | Title | SalePageId |
    | 商品A   | 25         |
    | 商品B   | 26         |
    | 商品C   | 27         |
    | 商品D   | 50         |

而且 第"2"檔折扣條件是"滿件折現,滿2件，折3元",如下
    | Id | TypeDef    | TotalQty | DiscountTypeDef | DiscountPrice |
    | 4  | TotalQtyV2 | 2        | DiscountPrice   | 3             |

當 計算活動折扣

那麼 購物車商品折扣金額及折扣後小計為 
    | Title | SalePageId | Price | Qty | PromotionDiscount | TotalPayment |
    | 商品A   | 25         | 7.45  | 2   | -2.55             | 12.35        |
    | 商品B   | 26         | 4.45  | 2   | -1.11             | 7.79         |
``` 

與 PM 及 QA 討論後 , 寫的 UAT 可閱讀性提高了  
這裡用到了一些小技巧 , 讓 Cucumber 文件的可讀性更高  
而不會有太多的重複方法 , 比如說把描述性的文字當作參數傳遞  
實際上測試不會使用到這些變數 ,但是可以增加可讀性 .


### 刪除台灣測試
因為已經有了跨國所需要的測試 , 
台灣的測試便可以退場了.
實際上也不符合現況, 如多語系、時差與小數點等問題

### 解析 `CalculateShoppingCartPromotionDiscountV2Processor` 

![](https://i.imgur.com/FioG5NG.jpg)
1. 無折扣的情境
2. 新舊相容的情境
3. 排序
4. 計算折扣金額
5. 看見相依
    1. 程式碼中有 new 別的 class 的部份
    2. 程式碼中有使用靜態方法的部份

### 補上單元測試

最簡單的重構,就是將整個方法內的四個邏輯  
拆成四塊個子方法,並為他們加上單元測試.  
修改的過程,如果有紅燈就要修改成綠燈,  
而整個成品要保證整合測試與單元測試都是綠燈. 

此外,重構的過程中如果過到靜態方法,  
或是 new 新物件, 都很有可能是種相依,  
可以透過一些方法作解耦,  
參考之前的文章[單元測試這樣玩就對了](/2017/04/23/unitestwriting/) 

### 重構
最後一步就是大膽的重構了,  
有了測試作保護,  
可以作更大範圍的重構,  
如下圖示,這裡揭露了在台灣原有的繼承結構, 
而紅色的部份是在跨國用不到的類別.
![](https://i.imgur.com/VQ10wY6.jpg)

下一步，待續…

(fin)