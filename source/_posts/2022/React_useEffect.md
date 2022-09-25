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

useEffect 的第一個參數是一個函數，可以回傳一個 callback function 用來清除 side Effect。

舉例說明:
想像我們有兩個連結用來切換不同的使用者資料，
在切換的過程中會打 api 取得不同使用者(user1 與 user2)的資料,

我們會先點 user1 的連結取 user1 資料(因網速過慢，此時資料還沒回來)，  
再點 user2 的連結取取 user2 資料(user2 的資料也還沒回來)，
這時，user1 資料回來了，畫面會渲染 user1 資料(實際上，這時候我想看 user2 的資料)，
再過一會兒，user2 資料回來了，畫面閃爍，渲染 user2 資料。

[線上範例看這](https://codesandbox.io/s/useeffect-clean-function-wlleou?file=/src/App.tsx),使用網速網路會更明顯。

這是表示當我們點擊 user2 的連結時，其實我們已經拋棄了點擊 user1 的結果(對我們來說已經不需要了)，
這時候 Clean up function 就是用來處理這個不再需要的 side-effect。

我們可以簡單設定一個 toggle 來處理這個問題

```tsx
useEffect(() => {
  let fetching = true;
  fetch(`https://jsonplaceholder.typicode.com/users/${id}`)
    .then((res) => res.json())
    .then((data) => {
      if (fetching) {
        setUser(data);
      }
    });
  return () => {
    fetching = false;
  };
}, [id]);
```

我們也可使用 [AbortController](https://developer.mozilla.org/zh-TW/docs/Web/API/AbortController),  
有關 AbortController 的相關資訊可以參考隨附連結，或是留言給我再作介紹。

```tsx
useEffect(() => {
  const controller = new AbortController();
  const signal = controller.signal;
  fetch(`https://jsonplaceholder.typicode.com/users/${id}`,{ signal })
    .then((res) => res.json())
    .then((data) => {
        setUser(data);
    })
    .catch(err=>{
      if(err.name === "AbortError){
        console.log("cancelled!")
      }else{
        //todo:handle error
      }
    });
  return () => {
    controller.abort();
  };
}, [id]);
```

以上，我們就介紹完了 useEffect 的三個部份(函數、回呼函數與相依數列)與用法。

如果你使用常見的套件 [axios](https://axios-http.com/docs/intro) 應該怎麼作 clean up，  
補充範例如下:

```tsx
useEffect(() => {
  const cancelToken = axios.cancelToken.source();
  axios(`https://jsonplaceholder.typicode.com/users/${id}`,{ cancelToken:cancelToken.token })
    .then((res) =>
        setUser(res.data);
    })
    .catch(err=>{
      if(axios.isCancel(err)){
        console.log("cancelled!")
      }else{
        //todo:handle error
      }
    });
  return () => {
    cancelToken.cancel();
  };
}, [id]);
```

### why render twice with React.StrictMode

請參考[官方文章](https://reactjs.org/docs/strict-mode.html),  
StrictMode 可以幫助開發者即早發現諸如下列的問題:

- Identifying components with unsafe life-cycles
- Warning about legacy string ref API usage
- Warning about deprecated findDOMNode usage
- Detecting unexpected side effects
- Detecting legacy context API
- Ensuring reusable state

## 小結

- React.StrictMode 可以幫助你檢查組件的生命周期(不僅僅 useEffect)
- React.StrictMode 很有用不應該考慮移除它
- useEffect 包含三個部份
  - 第一個參數(function)，處理 side-Effect 的商業邏輯
  - 第一個參數的回傳值(clean-up function)，處理 side-Effect 中斷時的邏輯(拋錯、釋放資源 etc…)
  - 第二個參數表示相依的參數陣列
    - 不傳值將會導致每次渲染都觸發副作用
    - 傳空陣列將會只執行一次
    - 相依的參數需注意 Primitive 與 Non-primitive
    - useMemo 可以協助處理 Primitive 與 Non-primitive

## 參考

- <https://reactjs.org/docs/hooks-reference.html#useeffect>
- <https://www.youtube.com/watch?v=QQYeipc_cik>
- <https://morioh.com/p/a8366f4ec07a?f=603c719d1528b00f7934320a&fbclid=IwAR07igsz8cQ49y1AQYElDBxPk23vqv-9WDYTlbrCf-yBcXhapz9WzAa9lhQ>
- <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax>
- <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters>
- <https://www.zhoulujun.cn/html/webfront/SGML/html5/2022_0530_8824.html>
  (fin)
