---
title: "[生活筆記] 2025 T社面試心得 - 一面"
date: 2025/11/28 18:41:01
---

## 前言

Gap Year 期間持續在面試，這次面試的是一家做 Data Engineering 的新創公司 T 社。

一面是 Python 基礎能力測驗，用 Google Colab 進行，大概 1 小時。

結論:通過

## 一面: Python 基礎測驗

面試工具是 Google Colab，面試官給了 10 題 Python 題目，
涵蓋基礎語法、物件導向、函數式程式設計等概念。

原本的題目都是很制式的語法練習題，
但我刻意把它們轉換成「購物車」這個業務情境來解答。

為什麼要這樣做？

因為我想展現的不只是「我會寫 Python 語法」，
而是「我知道什麼時候該用什麼方法」。

### 題目 1: Bubble Sort

**原始題目：**

```python
# Problem 1 - Bubble sort
def bubble_sort(sequence):
    # Write your bubble sort code here.
    pass

assert bubble_sort([5, 1, 3, 2, 4]) == [1, 2, 3, 4, 5]
```

經典排序演算法  
測試: `[5, 1, 3, 2, 4]` → `[1, 2, 3, 4, 5]`

**我的解答：**

```python
def bubble_sort(sequence):
    n = len(sequence)
    for i in range(n):
        for j in range(0, n-i-1):
            if sequence[j] > sequence[j+1]:
                sequence[j], sequence[j+1] = sequence[j+1], sequence[j]
    return sequence
```

### 題目 2: 找第二大值 (O(n) 複雜度)

**原始題目：**

```python
# Problem 2 - Find second largest
def find_second_largest(sequence):
    # Write your algorithm with O(n) time complexity here.
    pass

assert find_second_largest([3, 3, 2, 1]) == 2
assert find_second_largest([3, 3, 3, 3, 3, 2, 2, 1]) == 2
assert find_second_largest([-1, 2, 3, 5, 3, 1, 2, 4]) == 4
```

要求線性時間  
要處理重複值  
測試: `[3, 3, 2, 1]` → `2`

**我的解答：**

```python
def find_second_largest(sequence):
    largest = second_largest = float('-inf')
    
    for num in sequence:
        if num > largest:
            second_largest = largest
            largest = num
        elif num != largest and num > second_largest:
            second_largest = num
    return second_largest
```

### 題目 3: 繼承 (Inheritance)

**原始題目：**

```python
# Problem 3 - Inheritance
# Write some examples with inheritance code here.
```

實作商品類別  
`Product` 和 `DiscountProduct`  
我用購物車的商品折扣來展現繼承和多型

**我的解答：**

```python
# 父類別：商品
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price
    
    def get_price(self):
        return self.price

# 子類別：特價品
class DiscountProduct(Product):
    def __init__(self, name, price, discount):
        super().__init__(name, price)
        self.discount = discount  # 折扣，例如 0.8 代表 8 折
    
    # 覆寫取得價格的方法
    def get_price(self):
        return self.price * self.discount

# 使用範例
products = [
    Product("一般商品A", 1000),
    DiscountProduct("特價商品B", 1000, 0.8)
]

for p in products:
    print(p.name, "價格:", p.get_price())
```

### 題目 4: *args, **kwargs

**原始題目：**

```python
# Problem 4 - *args, **kwargs
# Write some examples with *args, **kwargs here.
```

可變參數  
我用購物車總價計算來展現（商品用 `*args`，運費用 `**kwargs`）

**我的解答：**

```python
# 計算購物車總價
def calculate_cart(*args, **kwargs):
    total = 0
    # args 可以放任意數量的商品物件
    for item in args:
        total += item.get_price()
    
    # kwargs 可以放額外參數，例如運費
    shipping = kwargs.get("shipping", 0)
    total += shipping
    
    return total

# 使用範例
p1 = Product("商品A", 1000)
p2 = DiscountProduct("特價商品B", 1000, 0.8)  # 8折
p3 = DiscountProduct("特價商品C", 500, 0.5)   # 5折

total_price = calculate_cart(p1, p2, p3, shipping=100)
print("購物車總價:", total_price)
```

### 題目 5: Lambda 函式

**原始題目：**

```python
# Problem 5 - lambda
# Write some examples using python lambda here.
```

函數式程式設計  
我用 `map()` + `lambda` 計算折扣總價

**我的解答：**

```python
# 以下使用 lambda 進行計算折扣後價格的加總
products = [
    {"name": "商品A", "price": 1000, "discount": 1},   # 一般商品
    {"name": "特價商品B", "price": 1000, "discount": 0.8},
    {"name": "特價商品C", "price": 500, "discount": 0.5}
]

total_price = sum(list(map(lambda p: p["price"] * p["discount"], products)))
print("總價:", total_price)
```

### 題目 6: List Comprehension

**原始題目：**

```python
# Problem 6 - comprehension
# Write some examples using python comprehension here.
```

Python 簡潔語法  
我用篩選折扣後價格 > 500 的商品來展現

**我的解答：**

```python
# 計算特價商品折扣後的價格，只保留大於500的商品
products = [
    {"name": "商品A_一千", "price": 1000, "discount": 1},
    {"name": "商品B_八百", "price": 1000, "discount": 0.8},
    {"name": "商品C_二百五", "price": 500, "discount": 0.5}
]

discounted_over_500 = [p["name"] for p in products if p["price"] * p["discount"] > 500]
print(discounted_over_500)
```

### 題目 7: Decorator

**原始題目：**

```python
# Problem 7 - decorator
# Write some examples using python decorator here.
```

裝飾器模式  
我實作了「限制購物車總折扣不超過 200」的驗證邏輯

**我的解答：**

```python
# 檢查總折扣不能超過 200 decorator
def max_discount_decorator(func):
    def wrapper(cart, *args):
        # 計算目前總折扣
        total_discount = sum(round((1 - p["discount"]) * p["price"]) for p in cart.items)
        # 計算新加入商品折扣
        new_discount = sum(round((1 - p["discount"]) * p["price"]) for p in args)
        if total_discount + new_discount > 200:
            print(f"不能加入商品，總折扣 {total_discount + new_discount} 超過 200")
            return False
        # 執行原方法
        return func(cart, *args)
    return wrapper

# 購物車類別
class Cart:
    def __init__(self):
        self.items = []
    
    @max_discount_decorator
    def add(self, *args):
        self.items.extend(args)
        print(f"加入 {len(args)} 個商品到購物車")

# 使用範例
cart = Cart()

p1 = {"name": "商品A_一千", "price": 1000, "discount": 0.9}  # 折扣額 100
p2 = {"name": "商品B_五百", "price": 500, "discount": 0.8}   # 折扣額 100
p3 = {"name": "商品C_三百", "price": 300, "discount": 0.5}   # 折扣額 150

cart.add(p1, p2)  # 可以加入，折扣總額 200
cart.add(p3)      # 無法加入，折扣總額 200 + 150 = 350 > 200
```

### 題目 8: Generator

**原始題目：**

```python
# Problem 8 - generator
# Write some examples using python generator here.
# Explain the benefit of generators here.
```

生成器  
我用折扣價格計算來展現  
好處是節省記憶體，適合大量資料

**我的解答：**

```python
products = [
    {"name": "商品A", "price": 1000, "discount": 1},
    {"name": "商品B", "price": 800, "discount": 0.8},
    {"name": "商品C", "price": 500, "discount": 0.5}
]

def discounted_price_gen(products):
    for p in products:
        yield {"name": p["name"], "discounted_price": p["price"] * p["discount"]}

for item in discounted_price_gen(products):
    print(item)

# 好處：
# - 節省記憶體：一次只產生一個值，不用把整個序列存在記憶體裡。
# - Lazy evaluation：只有真正取值時才計算，適合大資料或無限序列。
```

### 題目 9: Context Manager

**原始題目：**

```python
# Problem 9 - context manager
# Write some examples using python context manager here.
```

`with` 語句  
用檔案處理展現資源管理

**我的解答：**

```python
with open("test.txt", "w") as f:
    f.write("Hello world")  # 離開區塊會自動關閉檔案
```

### 題目 10: Magic Methods

**原始題目：**

```python
# Problem 10 - magic methods
# Write some examples using python magic methods here.
```

特殊方法 (`__str__`, `__len__`, `__add__`)  
我實作購物車類別來展現

**我的解答：**

```python
class Cart:
    def __init__(self):
        self.items = []
    
    def add(self, item):
        self.items.append(item)
    
    # 可以 print(cart) 顯示資訊
    def __str__(self):
        return f"購物車({len(self.items)} 件商品)"
    
    # len(cart) 會回傳商品數量
    def __len__(self):
        return len(self.items)
    
    # cart1 + cart2 合併購物車
    def __add__(self, other):
        new_cart = Cart()
        new_cart.items = self.items + other.items
        return new_cart

# 使用範例
cart1 = Cart()
cart1.add("商品A")
cart1.add("商品B")

cart2 = Cart()
cart2.add("商品C")

print(cart1)       # 購物車(2 件商品)
print(len(cart2))  # 1 (只有 1 件商品)

cart3 = cart1 + cart2
print(cart3)       # 購物車(3 件商品)
```

## 我的策略

原本題目都是純語法練習，
但我刻意想展現的是:

1. 知道**什麼時候該用什麼方法**
2. 將演算法概念**應用到實際業務**
3. 不是只會背語法，而是理解背後的**適用場景**

(fin)
