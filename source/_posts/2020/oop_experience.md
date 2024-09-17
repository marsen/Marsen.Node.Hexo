---
title: " [閱讀筆記] 物件導向的心得與隨筆 "
date: 2020/07/27 16:42:16
tags:
  - OOP
---

- Object 泛指所有的物件，Instance 是明確指透過 Class 創建的 Object

  - [Anonymous Types C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/anonymous-types)
  - Java ?

- Class 是 Object 的藍圖/設計書 ( Class 是 Object 的抽象 )
  - Object 是 Class 的具像化
  - 通常透過 new Class
  - 一種常見的應用是 Object 裡面只有欄位，用來作為資料的載體
  - 另一種常見的應用是 Object 包含可執行的 Function
  - Static 另作討論
- Abstract Class 是 Class 的藍圖/設計書 ( Abstract Class 是 Class 的抽象 )

  - Class 是 Abstract Class 具像化
    - 通常透過繼承方式
  - Abstract Class 會定義共用的欄位與方法給它的子類別
    - 當某一個方法，實作細節必須由子類別處理時，會宣告成抽象方法 abstract method
    - 子類別繼承後必須實作 abstract method
    - Abstract 無法直接具像成 Object

- Interface 是 Method 的藍圖/設計 ( Interface 是 Method 的抽象 )
  - Method 是 Interface 的具像化
    - 但是 Method 必須生存在 instance 上
    - 所以不會是 Static Object
    - 所以 Interface 必須與 Class 共存，才能在 instance 上實現

(fin)
