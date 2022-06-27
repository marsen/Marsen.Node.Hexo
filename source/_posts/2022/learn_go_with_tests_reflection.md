---
title: "[閱讀筆記] Learn Go with tests"
date: 2022/06/27 12:16:20
---

## 前情提要

在學習 Go 的過程中偶然發現的資源，
作者提供了一種有效學習新的語言(Go)的方法---測試。
我很認同這個想法，作者的文章簡單易懂，配合上測試案例，很快就能掌握 Go 語言的幾大特性，
同時你也會了測試，很酷的是，Golang 本身的測試語法就很好用，除了 Mocking 那個部份需引入外部資源外，  
其它就內建其中了，也就是說你不會像 C# 需要面對選擇的障礙(MsTest、NUnit、XUnit)，
同時網路上已經有簡體中文的資源(就像它們的文字一樣有些許殘缺，但對英文不夠好的人也是個福音了)

## 測試心法

### Mocking 是萬惡之源嗎?

通常，如果你的程式需要大量 mocking 這隱含著錯誤的抽象。
而背後代表的意義是你正在作糟糕的設計。

> What people see here is a weakness in TDD but it is actually a strength,  
> more often than not poor test code is a result of bad design or put more nicely,  
> well-designed code is easy to test.

#### 測試的壞味道與方針

如果你的測試變得複雜，或是需要 Mocking 需多依賴，這一種壞味道，  
可能是你的模組負擔太多的事情，試著去切分它。(注:沒切或切太塊，一個模組要運作要 mocking 數 10 個相依)
或是依賴關係變得太細緻，試著將適當的模組作分類(注:切太細，a 依賴 b、b 依賴 c……一路 mocking 到天涯海角)
或是太注重細節，應專注於行為而不是功能的實現(注:太注重細節會變成整合測試，需要完成的功能實現)

```text
- The thing you are testing is having to do too many things (because it has too many dependencies to mock)
  - Break the module apart so it does less
- Its dependencies are too fine-grained
  - Think about how you can consolidate some of these dependencies into one meaningful module
- Your test is too concerned with implementation details
 - Favour testing expected behaviour rather than the implementation
```

### 傳說中 KentBeck 大叔說的過的話

> Make It Work Make It Right Make It Fast

在本書中 `work` 意謂通過測試，`right` 是指重構代碼使意圖明顯好懂，最後 `fast` 才是優化效能。
如果沒有 `work` 與 `right` 之前是無法變 `fast` 的

## 章節 reflection 的重構步驟

參考章節 reflection，當完成 slice 的 test case 時候，  
程式已經變得相當[噁心](https://quii.gitbook.io/learn-go-with-tests/go-fundamentals/reflection#write-enough-code-to-make-it-pass-6)，

```golang
func walk(x interface{}, fn func(input string)) {
 val := getValue(x)

 if val.Kind() == reflect.Slice {
  for i := 0; i < val.Len(); i++ {
   walk(val.Index(i).Interface(), fn)
  }
  return
 }

 for i := 0; i < val.NumField(); i++ {
  field := val.Field(i)

  switch field.Kind() {
  case reflect.String:
   fn(field.String())
  case reflect.Struct:
   walk(field.Interface(), fn)
  }
 }
}
```

我來探討一下重構的思路，不然書中的重構步驟(對我來說)太快了，無法掌握變化的過程。
首先，我們有一個前提是每個步驟完成都要跑測試，並全數通過才行。
除了 Production Code 我們不會異動測試的任何程式碼。

### 消除 return

這一步應該不難理解，我們用 switch 語法取代 if return 的寫法  
這樣讓我們的程式更有整體性，而不是被 if return 切分成兩個區塊
但是這樣又產生了新的壞味道，巢狀 switch
如下:

```golang
func walk(x interface{}, fn func(input string)) {
 val := getValue(x)
 switch val.Kind() {
 case reflect.Slice:
  for i := 0; i < val.Len(); i++ {
   walk(val.Index(i).Interface(), fn)
  }
 default:
  for i := 0; i < val.NumField(); i++ {

   field := val.Field(i)
   switch field.Kind() {
   case reflect.String:
    fn(field.String())
   case reflect.Struct:
    walk(field.Interface(), fn)
   }
  }
 }
}
```

### 巢狀 switch

先來看一下這個巢狀 switch 的條件判斷為何?  
我們可以發現兩個 switch 最終都是在對 `.Kind()` 作判斷，
這帶來了可能性，
我們可以把內層 switch 的處理往上提昇
下層使用遞迴呼叫 `walk(...` 如果內層的 case 都被提昇至上層，
那麼內層的 switch 就可以被剝離

### 巢狀 switch : `case reflect.String`

先把 `case reflect.String:` 往上層提昇
內層保留 case ，但改呼叫遞迴，執行測試，全部通過

```golang
func walk(x interface{}, fn func(input string)) {
 val := getValue(x)
 switch val.Kind() {
 case reflect.Slice:
  for i := 0; i < val.Len(); i++ {
   walk(val.Index(i).Interface(), fn)
  }
 case reflect.String:
  fn(val.String())
 default:
  for i := 0; i < val.NumField(); i++ {

   field := val.Field(i)
   switch field.Kind() {
   case reflect.String:
    walk(field.Interface(), fn)
   case reflect.Struct:
    walk(field.Interface(), fn)
   }
  }
 }
}
```

### 巢狀 switch 消除重複

在本篇我只對內層的 switch 進行重構，  
下面我只展示內層的 switch。
兩個問題，

1. case 重複（這是我們刻意製造出來的）→ 所以要消重複
2. 消除重複後，參數 field 就變得有點多餘，我們可以用 inline 的手法消除

```golang
   field := val.Field(i)
   switch field.Kind() {
   case reflect.String:
    walk(field.Interface(), fn)
   case reflect.Struct:
    walk(field.Interface(), fn)
   }
```

測試通過後，我們的內層 switch 就會變成這樣

```golang
   switch val.Field(i).Kind() {
   case reflect.String, reflect.Struct:
    walk(val.Field(i).Interface(), fn)
   }
```

#### 巢狀 switch default case

這個時候我們看整體的程式碼，會發現一個怪異的現象，  
外層的 default 值會直接視作存在多個 Field 進行遞迴拆解 `for i := 0; i < val.NumField(); i++ {`  
內層的 switch 語法只有一個 case 同時處理 `reflect.Struct` 與 `reflect.String` 兩種條件，
以邏輯來說，外層的 default 只會處理 `reflect.Struct` 其它的資料型態都不處理，  
而 `reflect.String` 在同層的 switch 其它條件被處理掉了  
所以我們可以把 default 的區段改寫如下，執行測試，通過

```golang
 case reflect.Struct:
  for i := 0; i < val.NumField(); i++ {
   switch val.Field(i).Kind() {
   //case reflect.String, reflect.Struct://這個寫法也可以
   default://這個寫法比較有交換率的等價概念
    walk(val.Field(i).Interface(), fn)
   }
  }
```

這時候內層的 switch 就變得相當的多餘，可以整個拿掉。  
執行測試，通過。
最後的程式重構就會與書上的一致，十分雅緻

```golang
func walk(x interface{}, fn func(input string)) {
 val := getValue(x)
 switch val.Kind() {
 case reflect.String:
  fn(val.String())
 case reflect.Slice:
  for i := 0; i < val.Len(); i++ {
   walk(val.Index(i).Interface(), fn)
  }
 case reflect.Struct:
  for i := 0; i < val.NumField(); i++ {
   walk(val.Field(i).Interface(), fn)
  }
 }
}
```

## 參考

- <https://github.com/quii/learn-go-with-tests>
- <https://studygolang.gitbook.io/learn-go-with-tests/>

(fin)
