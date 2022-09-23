---
title: "[學習筆記] React useEffect"
date: 2022/09/12 12:20:34
tag:
  - React
---

## 簡介 useEffect

在 RC(React Component) 當中，useEffect 是一個常用 Hook，用來處理一些副作用(side-Effect)。

用 FP(Functional Programming) 的角度來看，RC 只是依照狀態(state)或是參數(prop)來呈現不同的外觀，就像是一個單純的 Pure Function。
FP 一些常見的副作用如下:

- I/O (存取檔案、寫 Log 等)
- 與資料庫互動
- Http 請求
- DOM 操作

而 Http 與 DOM 正好是我們開發 RC 最常接觸到的副作用，
useEffect 就是要來解決這個問題。

### 如何運作

1. RC 在渲染的時候會通知 React
2. React 通知 Browser 渲染
3. Browser 渲染後，執行 Effect

### 使用範例

下面是 VSCode 外掛產生的程式片段

```tsx
useEffect(() => {
  first;

  return () => {
    second;
  };
}, [third]);
```

我們可以看到 useEffect 需要提供兩個參數，一個函數與一個陣列
這裡有三處邏輯，first、second、third,
為了更好說明概念，順序會稍會有點跳躍，請再參考上面程式範例

### 相依(dependencies)

third 就是指與此副作用相依的參數，如果不特別加上這個參數，
每次重新渲染都會觸發 Effect 的第一個參數的函數，
如果只提供一個空陣列，就只會在第一次渲染的時候觸發，這很適合用在初始化的情況。
觸發時機:

- 不傳值，每次渲染時
- []，只有第一次渲染時
- [dep1,dep2,...]

#### 注意的事項與 useMemo

useEffect 在判斷觸發的 dependencies 參數，
會有 Primitive 與 Non-primitive 參數的差別。

Primitive: 比如 number、boolean 與 string
Non-primitive: 比如 object、array
這兩種參數的差異在於 JavaScript 在實作時，記憶體的使用方式

> Primitive 會直接將值記錄在記憶體區塊中，
> Non-primitive，則會另外劃一塊記憶體位置存值，再將這塊記憶體位址存到參數的記憶體位址中。
> 所以當我們比較的是記憶體位址時，即使內部的值都相同也會回傳 false

參考[範例](https://codesandbox.io/s/non-primitive-xmlfnc?file=/src/App.tsx)

```tsx
const [staff, setStaff] = useState({ name: "", toggle: false });
useEffect(() => {
  console.count(`staff updated:${JSON.stringify(staff)}`);
}, [staff]);
//}, [staff.name, staff.toggle]);
```

每次我們點擊 Change Name 按鈕的時候，都會呼叫改變命名的方法，

```tsx
const handleName = () => setStaff((prev) => ({ ...prev, name: name }));
```

但是即時我們的 name 沒有改變，仍然觸發 Effect，  
這是因為 staff 是一個 Non-Primitive 型別的變數。
一種簡單暴力的方法是將物件展開放入 `[]` 之中。

這裡我們無法使用[展開運算符(Spread Operator)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)
因為一般的物件沒有 `Symbol.iterator` 方法

> Only iterable objects, like Array, can be spread in array and function parameters.
> Many objects are not iterable, including all plain objects that lack a Symbol.iterator method:

物件展開放入 `[]` 之中的明顯缺點是，當物件變的太大或複雜的時候，  
你的 `[]` 會變得又臭又長。

解法可以使用 [useMemo](https://reactjs.org/docs/hooks-reference.html#usememo)
`useMemo` 會回傳一個存在記憶體裡的值，  
完整範例請[參考](https://codesandbox.io/s/non-primitive-usememo-pultzn?file=/src/App.tsx:325-535)

```tsx
const memo = useMemo(
  () => ({ name: staff.name, toggle: staff.toggle }),
  [staff.name, staff.toggle]
);

useEffect(() => {
  console.count(`staff updated:${JSON.stringify(memo)}`);
}, [memo]);
```

### 函數

回到我們的程式片段
first 其實就是處理 Effect 函數，我們可以把 Effect 寫在這裡，  
範例中都是寫 console.log 實務上更多會是打 API、fetch etc... 的行為

### Clean up functions

TBW
上一段所說的函數，可以回傳一個 callback function 用來清除 side Effect，
想像我們有兩個 tab 用來切換不同的使用者資料，在切換的過程中會去 fetch 不同使用者的資料,  
假設我們在一個慢速網路的情境底，快速的切換頁，那很有可能會看到頁面的資料閃爍，
原因是我們先點 user1 的 tab 取 user1 資料(此時資料還沒回來)，  
再點 user2 的 tab 取 user2 資料(user2 的資料也還沒回來)，
這時，user1 資料回來了，畫面會渲染 user1 資料(實際上我想看 user2 的資料)，
再過一會兒，user2 資料回來了，畫面閃爍，渲染 user2 資料。

這是很惱人的，TBW

### why render twice with React.StrictMode

TBW

## When and how it runs

- Render Component
-

## 參考

- <https://reactjs.org/docs/hooks-reference.html#useeffect>
- <https://www.youtube.com/watch?v=QQYeipc_cik>
- <https://morioh.com/p/a8366f4ec07a?f=603c719d1528b00f7934320a&fbclid=IwAR07igsz8cQ49y1AQYElDBxPk23vqv-9WDYTlbrCf-yBcXhapz9WzAa9lhQ>
- <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax>
- <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters>
  (fin)
