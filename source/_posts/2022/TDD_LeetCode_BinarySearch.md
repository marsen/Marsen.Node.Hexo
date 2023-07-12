---
title: "[實作筆記] TDD 二元搜尋樹(Binary Search)"
date: 2022/10/22 18:27:27
tags:
  - 實作筆記
---

## 前情提要

最近想再面試，發現大家都很流行刷題，好像沒有刷個 100、200 題，就不能夠面試了。
我剛開始從事軟體工作之初，刷題並不是一個很主流的面試條件，不過也是會準備一些考古題，  
關於這樣的面試方式，是不是真的能找到公司要的人才，或許有機會未來，再寫一篇來聊聊。

總之，我很喜歡 TDD 的開發方式，試著在刷題的過程之中，順便練手一下 TDD

## 二元搜尋樹(Binary Search)

演算法題庫的第一題，就是二元搜尋樹(Binary Search)，
概念上很簡單，就是將數列排列後，由中位數去比較大小，
如果目標比中位數大，那麼就往右側的樹，再作一次搜尋
如果目標比中位數小，那麼就往左側的樹，再作一次搜尋
直到找目標，或是抵達樹葉，而仍未找到目標便回傳 -1

![二元搜尋樹(Binary Search Tree)](https://i.imgur.com/mC5fNMA.png)

這樣的作法，不考慮極端狀況下，會比遍歷整個樹快一倍

## 測試案例

### 目標 5，空陣列，回傳-1

這個案例用來建立方法簽章，
用最簡單的方法實作

```csharp
public int Search(int[] nums, int target) {
  return -1;
}
```

### 目標 5，[5]，回傳 0

這個案例用來實作判斷 target 是否在 nums 之中，
先 hard code 通過測試

```csharp
public int Search(int[] nums, int target) {
  if (nums.Length > 0)
    return 0;
  return -1;
}
```

接著我們要修改 hard code 的部份，首先實作中位數的比較，
同時代出左右 index 的概念

```csharp
  if (nums.Length > 0)
  {
      int left = 0;
      int right = nums.Length - 1;
      int mid = (right - left) / 2;
      if (nums[mid] == target)
      {
          return mid;
      }
  }
```

重構，利用左右 index 的觀念，改寫進入搜尋的條件式

```csharp
  int left = 0;
  int right = nums.Length - 1;
  if (right - left >= 0)
  {
      int mid = (right - left) / 2;
      if (nums[mid] == target)
      {
          returN mid;
      }
  }
```

### 目標 5，[5,6]，回傳 0

我們的樹開始長出葉子，僅管通過測試，
我們也需要透過這個案例，產生一個 while 迴圈，

```csharp
  int left = 0;
  int right = nums.Length - 1;
  while (right >= left)
  {
      int mid = (right - left) / 2;
      if (nums[mid] == target)
      {
          returN mid;
      }
  }
```

### 目標 5，[5,6,7]，回傳 0

### 目標 5，[3,4,5]，回傳 2

有了 mid 的概念，我們可以以 mid 的左右 假想為不同的子樹，
用這兩個案例逼出左、右樹邊界重設條件，  
注意 mid 同時也是邊界重要條件之一，  
為此，我們需要調整 mid 的邏輯，
完整的程式如下:

```csharp
  int left = 0;
  int right = nums.Length - 1;
  while (right >= left)
  {
      int mid = (right - left) / 2 + left;
      if (nums[mid] == target)
      {
          returN mid;
      }
      if (nums[mid] < target)
      {
          left = mid + 1;
      }

      if (nums[mid] > target)
      {
          right = mid - 1;
      }
  }
```

## 小結

二元搜尋樹是屬於簡單的題型，  
也是資工學習演算法的初級題目，  
轉為 TDD 的過程也不困難，掌握幾個原則，
一、先設計再實作
二、用測試案例趨動實作
三、果斷拋棄無法趨動實作的測試案例(案例設計不當的壞味道)
四、適當的重構，以符合當初的設計

(fin)
