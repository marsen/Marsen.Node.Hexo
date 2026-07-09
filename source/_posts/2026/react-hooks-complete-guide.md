---
title: "[實作筆記] React Hooks 全套指南：7 個核心 Hook 的定義、範例與陷阱"
date: 2026/05/21 11:52:13
tags:
  - 實作筆記
---

## 前情提要

整理一份 React Hooks 的完整筆記，把日常會用到的 7 個核心 Hook 集中起來：`useState`、`useEffect`、`useReducer`、`useMemo`、`useCallback`、`useRef`、`useContext`。每個 Hook 都附上定義、用途、完整範例、以及實務上踩過的坑。

Hook 是 React 16.8 之後的主流寫法，把 class component 的 state、lifecycle、context 等能力下放到 function component。寫得熟練可以讓元件變得很乾淨；用錯了則會踩進無限 re-render、stale closure、依賴陣列地獄。

文章偏長，建議當作 reference 對著用。

---

## useState：狀態管理的基本款

### 定義

`useState` 是最基礎的 Hook，讓 function component 擁有自己的狀態。呼叫後回傳一個長度為 2 的 tuple：當前狀態值，以及更新該狀態的 setter。

```ts
const [state, setState] = useState<T>(initialValue);
```

每次 component re-render，`state` 都會反映最新的值；呼叫 `setState` 會排程下一次 render。React 內部用「位置」記住每個 useState 的對應狀態，這也是為什麼 Hook 不能寫在 `if`、`for` 裡面 — 一旦呼叫順序變了，狀態就會錯位。

### 用途

- 表單欄位（input value、checkbox checked、select 選項）
- UI 狀態（modal 開關、tab 切換、loading flag）
- 簡單的計數器、toggle、累加器
- 任何只屬於這個元件、不需要跟其他元件共享的狀態

如果狀態邏輯複雜、有多個分支、或要跟其他元件同步，再考慮 `useReducer` 或全域 state 方案。

### 完整範例

一個帶有暫存功能的 Todo 輸入框：

```tsx
import { useState } from 'react';

type Todo = { id: number; text: string; done: boolean };

export function TodoApp() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [input, setInput] = useState('');

  const addTodo = () => {
    if (!input.trim()) return;
    setTodos((prev) => [
      ...prev,
      { id: Date.now(), text: input.trim(), done: false },
    ]);
    setInput('');
  };

  const toggle = (id: number) => {
    setTodos((prev) =>
      prev.map((t) => (t.id === id ? { ...t, done: !t.done } : t))
    );
  };

  const remove = (id: number) => {
    setTodos((prev) => prev.filter((t) => t.id !== id));
  };

  return (
    <div>
      <input
        value={input}
        onChange={(e) => setInput(e.target.value)}
        onKeyDown={(e) => e.key === 'Enter' && addTodo()}
        placeholder="What needs to be done?"
      />
      <button onClick={addTodo}>Add</button>
      <ul>
        {todos.map((t) => (
          <li
            key={t.id}
            style={{ textDecoration: t.done ? 'line-through' : 'none' }}
          >
            <span onClick={() => toggle(t.id)}>{t.text}</span>
            <button onClick={() => remove(t.id)}>x</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

重點觀察：

- `setTodos((prev) => ...)` 使用 functional update，保證拿到最新狀態
- Array / Object 一定要回傳「新的參考」，React 才會偵測到變化
- 把每個獨立狀態切開（`todos`、`input`）比塞成一個物件好維護

### 常見陷阱

**1. 直接 mutate 物件或陣列**

```tsx
// 錯誤：React 不會 re-render
todos.push(newTodo);
setTodos(todos);

// 正確
setTodos((prev) => [...prev, newTodo]);
```

React 用 `Object.is` 比對前後狀態，同一個 reference 會被當作沒變。

**2. 連續 setState 拿到舊值**

```tsx
// 錯誤：count 只會 +1
setCount(count + 1);
setCount(count + 1);

// 正確：用 functional update
setCount((c) => c + 1);
setCount((c) => c + 1);
```

`setState` 是非同步排程，多次呼叫時不能依賴閉包裡的舊變數。React 18 之後 automatic batching 會把同步 event 裡的多次 setState 合併成一次 render，這個問題會更常見。

**3. 初始值是昂貴運算**

```tsx
// 每次 render 都會跑一次 expensiveInit
const [data, setData] = useState(expensiveInit());

// 正確：lazy initializer 只跑一次
const [data, setData] = useState(() => expensiveInit());
```

初始值傳「函式」而不是「函式呼叫結果」，可以避免重複計算。

**4. 衍生狀態（derived state）**

```tsx
// 反模式：fullName 是 firstName + lastName 的衍生值
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const [fullName, setFullName] = useState('');

useEffect(() => {
  setFullName(`${firstName} ${lastName}`);
}, [firstName, lastName]);

// 正確：直接在 render 期間算
const fullName = `${firstName} ${lastName}`;
```

能直接算出來的不要存進 state，否則就要靠 useEffect 同步，多了一次 render 也容易出 bug。

---

## useEffect：副作用與生命週期

### 定義

`useEffect` 讓 function component 可以執行「副作用」：跟外部世界互動的動作，例如打 API、訂閱事件、設定 timer、操作 DOM。

```ts
useEffect(() => {
  // 副作用程式碼
  return () => {
    // 清理（cleanup）
  };
}, [dep1, dep2]);
```

第二個參數是依賴陣列：

- `undefined`：每次 render 都跑
- `[]`：只在 mount 跑一次
- `[a, b]`：a 或 b 變更時才跑

執行時機：在 DOM 更新完之後、瀏覽器繪製之前（async）。如果需要在繪製前同步執行，用 `useLayoutEffect`。

### 用途

- 打 API 抓資料
- 訂閱 / 取消訂閱 WebSocket、EventListener
- 設定 / 清除 `setTimeout`、`setInterval`
- 同步外部資料來源（localStorage、URL params）
- 跟非 React 函式庫整合（D3、map library、video player 等）

### 完整範例

抓使用者資料 + cleanup race condition：

```tsx
import { useEffect, useState } from 'react';

type User = { id: string; name: string; email: string };

export function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    let cancelled = false;
    setLoading(true);
    setError(null);

    fetch(`/api/users/${userId}`)
      .then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then((data: User) => {
        if (!cancelled) {
          setUser(data);
          setLoading(false);
        }
      })
      .catch((err: Error) => {
        if (!cancelled) {
          setError(err.message);
          setLoading(false);
        }
      });

    return () => {
      cancelled = true;
    };
  }, [userId]);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;
  return <p>Hello, {user?.name}</p>;
}
```

`cancelled` flag 是處理 race condition 的關鍵：當 `userId` 快速切換時，舊請求的回應不能蓋掉新請求的結果。新版專案可以改用 `AbortController`：

```tsx
useEffect(() => {
  const ctrl = new AbortController();
  fetch(`/api/users/${userId}`, { signal: ctrl.signal })
    .then((res) => res.json())
    .then(setUser)
    .catch((err) => {
      if (err.name !== 'AbortError') setError(err.message);
    });
  return () => ctrl.abort();
}, [userId]);
```

訂閱 / 取消訂閱：

```tsx
useEffect(() => {
  const handler = (e: KeyboardEvent) => {
    if (e.key === 'Escape') closeModal();
  };
  window.addEventListener('keydown', handler);
  return () => window.removeEventListener('keydown', handler);
}, []);
```

### 常見陷阱

**1. 依賴漏掉 → stale closure**

```tsx
// 錯誤：count 永遠是 0
useEffect(() => {
  const id = setInterval(() => {
    console.log(count); // 永遠是初始值
  }, 1000);
  return () => clearInterval(id);
}, []);

// 正確：把 count 加進依賴
useEffect(() => {
  const id = setInterval(() => {
    console.log(count);
  }, 1000);
  return () => clearInterval(id);
}, [count]);
```

裝 `eslint-plugin-react-hooks` 的 `exhaustive-deps` 規則，會自動抓出來。

**2. async function 直接當 callback**

```tsx
// 錯誤：useEffect 不接受回傳 Promise
useEffect(async () => {
  const data = await fetch(...);
}, []);

// 正確：包一層 IIFE 或 named function
useEffect(() => {
  (async () => {
    const data = await fetch(...);
  })();
}, []);
```

`useEffect` 預期 callback 回傳「cleanup function 或 undefined」，async function 回傳 Promise 會破壞 cleanup 機制。

**3. 無限 re-render**

```tsx
// 錯誤：每次 render 都換新 reference，effect 永遠跑
useEffect(() => {
  setData({ value: 1 });
}, [{ filter: 'a' }]);

// 正確：依賴用 primitive 或穩定的 reference
useEffect(() => {
  setData({ value: 1 });
}, ['a']);
```

物件 / 陣列 / 函式作為依賴時，要保證 reference 穩定，否則每次 render 都會觸發。

**4. 在 effect 裡同步更新 state 卻沒守好邊界條件**

```tsx
// 錯誤：infinite loop
useEffect(() => {
  setCount(count + 1);
}, [count]);
```

每次 count 變動 → effect 觸發 → 再改 count → 再觸發。永遠停不下來。

**5. Strict Mode 下 effect 跑兩次**

React 18 開發模式下，`<StrictMode>` 會故意把 effect mount → unmount → mount 跑兩次，逼你寫好 cleanup。如果你的 effect 沒寫 cleanup，可能會看到 API 被打兩次、訂閱被建立兩次。這是設計而非 bug，正式環境只跑一次。

---

## useReducer：狀態邏輯複雜時的選擇

### 定義

`useReducer` 是 `useState` 的進階版。當狀態邏輯有多個分支、多個 action 類型、或多個欄位互相牽動時，用 reducer 把「狀態變更邏輯」集中管理。

```ts
const [state, dispatch] = useReducer(reducer, initialState);
```

`reducer(state, action) => newState` 是 pure function，接收當前狀態與 action，回傳新狀態。

### 用途

- 表單狀態管理（多欄位 + 驗證）
- 步驟流程（wizard、checkout flow）
- 局部複雜狀態（一個元件內，但 useState 已經太亂）
- 需要明確記錄 state transition 的場景
- 想做單元測試的狀態邏輯（reducer 是 pure function，測試非常好寫）

如果狀態還要跨元件共享，再搭配 `useContext` 變成輕量級的 Redux。

### 完整範例

購物車：

```tsx
import { useReducer } from 'react';

type Item = { id: string; name: string; price: number; qty: number };

type State = {
  items: Item[];
  discount: number;
};

type Action =
  | { type: 'ADD'; item: Omit<Item, 'qty'> }
  | { type: 'REMOVE'; id: string }
  | { type: 'CHANGE_QTY'; id: string; qty: number }
  | { type: 'APPLY_DISCOUNT'; rate: number }
  | { type: 'CLEAR' };

const initialState: State = { items: [], discount: 0 };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'ADD': {
      const existing = state.items.find((i) => i.id === action.item.id);
      if (existing) {
        return {
          ...state,
          items: state.items.map((i) =>
            i.id === action.item.id ? { ...i, qty: i.qty + 1 } : i
          ),
        };
      }
      return {
        ...state,
        items: [...state.items, { ...action.item, qty: 1 }],
      };
    }
    case 'REMOVE':
      return {
        ...state,
        items: state.items.filter((i) => i.id !== action.id),
      };
    case 'CHANGE_QTY':
      return {
        ...state,
        items: state.items.map((i) =>
          i.id === action.id ? { ...i, qty: Math.max(1, action.qty) } : i
        ),
      };
    case 'APPLY_DISCOUNT':
      return { ...state, discount: action.rate };
    case 'CLEAR':
      return initialState;
    default:
      return state;
  }
}

export function Cart() {
  const [state, dispatch] = useReducer(reducer, initialState);
  const subtotal = state.items.reduce((s, i) => s + i.price * i.qty, 0);
  const total = subtotal * (1 - state.discount);

  return (
    <div>
      <ul>
        {state.items.map((i) => (
          <li key={i.id}>
            {i.name} × {i.qty} = ${i.price * i.qty}
            <button onClick={() => dispatch({ type: 'REMOVE', id: i.id })}>
              Remove
            </button>
          </li>
        ))}
      </ul>
      <p>Subtotal: ${subtotal}</p>
      <p>Total: ${total.toFixed(2)}</p>
      <button onClick={() => dispatch({ type: 'CLEAR' })}>Clear</button>
    </div>
  );
}
```

優點：

- 所有狀態變更邏輯集中在 reducer，元件只負責 dispatch
- 容易測試：reducer 是 pure function，丟個 state + action 就能斷言結果
- Action 名稱明確，看 log 就能追蹤狀態變化
- 用 discriminated union 定義 action，TypeScript 會幫你檢查 payload

### 常見陷阱

**1. 在 reducer 裡 mutate state**

```ts
// 錯誤
case 'ADD':
  state.items.push(action.item);
  return state;

// 正確
case 'ADD':
  return { ...state, items: [...state.items, action.item] };
```

Reducer 必須是 pure function。直接 mutate 會讓 React 比對失敗、debug 困難、time-travel 工具（如 Redux DevTools）失效。

**2. 把副作用放進 reducer**

```ts
// 錯誤
case 'SAVE':
  fetch('/api/save', { ... });
  return state;
```

Reducer 必須是 pure function，不能有 side effect。API 呼叫應該放在元件裡的 event handler 或 useEffect，等成功後再 dispatch。

**3. Action 沒有用 discriminated union**

```ts
// 不好
type Action = { type: string; payload?: any };

// 好
type Action =
  | { type: 'ADD'; item: Item }
  | { type: 'REMOVE'; id: string };
```

少了型別保護，dispatch 時 typo 不會被發現。

**4. State 結構過深**

```ts
state = { user: { profile: { settings: { theme: 'dark' } } } };
```

更新時要層層展開：`{ ...state, user: { ...state.user, profile: { ... } } }`。考慮拆成多個 reducer，或用 Immer 簡化：

```ts
import { produce } from 'immer';

const reducer = (state: State, action: Action) =>
  produce(state, (draft) => {
    switch (action.type) {
      case 'TOGGLE_THEME':
        draft.user.profile.settings.theme = 'dark';
        break;
    }
  });
```

---

## useMemo：計算結果的快取

### 定義

`useMemo` 快取「計算結果」。只有當依賴陣列變化時，才重新計算；否則直接回傳上一次的結果。

```ts
const value = useMemo(() => expensiveCompute(a, b), [a, b]);
```

注意：`useMemo` 不是「保證」快取，React 在某些情況（如記憶體壓力）會丟棄快取重新計算。所以**不能依賴它做副作用**。

### 用途

- 昂貴的計算（大量資料 filter / sort / aggregate）
- 產生 stable reference 給子元件（避免 props 變更）
- 衍生狀態（derived state）

很多人用 `useMemo` 把所有東西都包起來，這是反模式。簡單的計算（加減乘除、字串組合）反而會因為快取的 overhead 變慢。

### 完整範例

搜尋 + 排序大型清單：

```tsx
import { useMemo, useState } from 'react';

type Product = { id: string; name: string; price: number; tags: string[] };

export function ProductList({ products }: { products: Product[] }) {
  const [query, setQuery] = useState('');
  const [sortKey, setSortKey] = useState<'name' | 'price'>('name');

  const filtered = useMemo(() => {
    const q = query.toLowerCase().trim();
    if (!q) return products;
    return products.filter(
      (p) =>
        p.name.toLowerCase().includes(q) ||
        p.tags.some((t) => t.toLowerCase().includes(q))
    );
  }, [products, query]);

  const sorted = useMemo(() => {
    return [...filtered].sort((a, b) => {
      if (sortKey === 'price') return a.price - b.price;
      return a.name.localeCompare(b.name);
    });
  }, [filtered, sortKey]);

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      <select
        value={sortKey}
        onChange={(e) => setSortKey(e.target.value as 'name' | 'price')}
      >
        <option value="name">Name</option>
        <option value="price">Price</option>
      </select>
      <p>{sorted.length} results</p>
      <ul>
        {sorted.map((p) => (
          <li key={p.id}>
            {p.name} - ${p.price}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

`products` 有上萬筆時，每次按鍵都重新 filter + sort 會卡頓。`useMemo` 確保只在 `query` 或 `sortKey` 改變時才重算。

### 常見陷阱

**1. 過度使用**

```tsx
// 多此一舉：a + b 比 useMemo 的 overhead 還便宜
const sum = useMemo(() => a + b, [a, b]);
```

React 官方文件直接寫過：default 不要包 useMemo，profiler 量到瓶頸再加。

**2. 依賴陣列漏掉**

```tsx
// 錯誤：用到 filter，但沒列進依賴
const result = useMemo(() => data.filter(filter), [data]);
```

跟 useEffect 一樣，少了依賴就會拿到 stale value。一樣靠 `exhaustive-deps` lint。

**3. 期待 useMemo 保證執行**

```tsx
// 錯誤：把副作用放進 useMemo
useMemo(() => {
  fetch('/api/log'); // 不保證執行次數
}, [trigger]);
```

副作用要用 useEffect。useMemo 是「可丟棄的快取」，不是 lifecycle hook。

**4. 把整個元件樹包進 useMemo**

```tsx
// 通常沒意義
const jsx = useMemo(() => <SomeComponent {...props} />, [...props]);
```

JSX 已經夠便宜了，包 useMemo 通常沒有意義。要 memo 整個元件改用 `React.memo`。

**5. 依賴是物件 / 陣列**

```tsx
// 錯誤：每次都新 reference
const result = useMemo(() => compute(opts), [{ flag: true }]);
```

依賴用 primitive 比較穩定。如果一定要傳物件，先用 useMemo 把物件包起來再傳下去。

---

## useCallback：函式的快取

### 定義

`useCallback` 是 `useMemo` 的特例，專門快取「函式」。等價於 `useMemo(() => fn, deps)`。

```ts
const memoizedFn = useCallback((args) => { ... }, [deps]);
```

每次 render 都會建立新的函式 reference。如果這個函式要傳給 `React.memo` 的子元件，或當作 useEffect 的依賴，就需要 useCallback 來保持 reference 穩定。

### 用途

- 傳給 `React.memo` 包過的子元件當 prop
- 當作其他 hook（useEffect / useMemo）的依賴
- 傳給 custom hook，避免內部 effect 反覆觸發

如果沒有上述情況，函式重新建立其實沒成本，不必特別包。

### 完整範例

避免子元件不必要的 re-render：

```tsx
import { memo, useCallback, useState } from 'react';

type Props = {
  label: string;
  onClick: () => void;
};

const Button = memo(({ label, onClick }: Props) => {
  console.log(`render: ${label}`);
  return <button onClick={onClick}>{label}</button>;
});

export function Counter() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  // 沒有 useCallback：每次 setName 觸發 re-render，increment 都是新 reference，Button 也會 re-render
  // 有 useCallback：reference 穩定，Button 不會跟著 re-render
  const increment = useCallback(() => {
    setCount((c) => c + 1);
  }, []);

  return (
    <div>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <p>Count: {count}</p>
      <Button label="Add" onClick={increment} />
    </div>
  );
}
```

關鍵：

- `Button` 用 `React.memo` 包起來，props 沒變就不 re-render
- `increment` 用 `useCallback`，reference 不變
- 即使 `name` 變更觸發 `Counter` re-render，`Button` 也不會跟著重畫

當作 useEffect 依賴：

```tsx
function SearchBox({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState('');

  useEffect(() => {
    const timer = setTimeout(() => onSearch(query), 300);
    return () => clearTimeout(timer);
  }, [query, onSearch]);

  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}

// 呼叫端
function Parent() {
  const handleSearch = useCallback((q: string) => {
    fetch(`/api/search?q=${q}`);
  }, []); // 沒 useCallback 的話，每次 Parent re-render 都會觸發 SearchBox 的 effect
  return <SearchBox onSearch={handleSearch} />;
}
```

### 常見陷阱

**1. 子元件沒有 memo，包 useCallback 沒用**

```tsx
// 沒用：Child 沒有 memo，每次 Counter render 都會 re-render Child
const Child = ({ onClick }) => <button onClick={onClick}>x</button>;
const handler = useCallback(() => {}, []);
```

useCallback 要搭配 `React.memo`、`useMemo`、或 effect 的依賴陣列才有意義。單獨用就是純粹的 overhead。

**2. 依賴陣列漏掉 → 函式抓到舊變數**

```tsx
// 錯誤：increment 永遠用初始的 step
const increment = useCallback(() => {
  setCount((c) => c + step);
}, []); // 漏掉 step
```

跟 useEffect / useMemo 同樣的問題。lint 規則統一處理。

**3. 過度包裝**

```tsx
// 沒必要
const onClick = useCallback(() => console.log('hi'), []);
return <button onClick={onClick}>x</button>;
```

`<button>` 是原生元素，不會 re-render 的成本問題。直接寫 inline function 即可。

**4. 認為 useCallback 會避免 render**

useCallback **不會**避免父元件的 re-render。它只是讓「傳給子元件的函式 reference 維持不變」，配合 `React.memo` 才能真正避免子元件的 re-render。

**5. 對 inline JSX prop 包 useCallback 也救不了**

```tsx
<Child onClick={handler} style={{ color: 'red' }} />
```

即使 `handler` 穩定，`style` 每次都是新物件，`React.memo` 還是擋不住。要連 prop 物件都用 useMemo 包。

---

## useRef：跨 render 的可變容器

### 定義

`useRef` 回傳一個物件 `{ current: T }`，這個物件在整個元件 lifetime 內持續存在，不會因為 re-render 重建。修改 `ref.current` 不會觸發 re-render。

```ts
const ref = useRef<T>(initialValue);
```

兩種主要用途：

1. 存取 DOM 元素
2. 存可變的值，但不需要觸發 re-render

### 用途

- 取得 DOM 節點（focus、scroll、measure 尺寸）
- 整合非 React 函式庫（Chart.js、Leaflet、video.js）
- 存 timer ID、AbortController、interval handle
- 記錄前一次 props 或 state（usePrevious 的實作）
- 在 callback 中讀取最新值（避免 stale closure）

### 完整範例

自動 focus + 計算停留時間：

```tsx
import { useEffect, useRef, useState } from 'react';

export function SearchBox() {
  const inputRef = useRef<HTMLInputElement>(null);
  const enterTimeRef = useRef<number>(Date.now());
  const [query, setQuery] = useState('');

  useEffect(() => {
    inputRef.current?.focus();
    enterTimeRef.current = Date.now();

    return () => {
      const duration = Date.now() - enterTimeRef.current;
      console.log(`Stayed for ${duration}ms`);
    };
  }, []);

  return (
    <input
      ref={inputRef}
      value={query}
      onChange={(e) => setQuery(e.target.value)}
    />
  );
}
```

存 interval handle 並 cleanup：

```tsx
import { useEffect, useRef, useState } from 'react';

export function Timer() {
  const [seconds, setSeconds] = useState(0);
  const intervalRef = useRef<ReturnType<typeof setInterval> | null>(null);

  const start = () => {
    if (intervalRef.current) return;
    intervalRef.current = setInterval(() => {
      setSeconds((s) => s + 1);
    }, 1000);
  };

  const stop = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  };

  useEffect(() => stop, []);

  return (
    <div>
      <p>{seconds}s</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </div>
  );
}
```

實作 `usePrevious`：

```tsx
function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T>();
  useEffect(() => {
    ref.current = value;
  }, [value]);
  return ref.current;
}
```

### 常見陷阱

**1. 期待修改 ref 會 re-render**

```tsx
// 錯誤：點按鈕沒反應
const countRef = useRef(0);
return (
  <div>
    <p>{countRef.current}</p>
    <button onClick={() => countRef.current++}>Add</button>
  </div>
);
```

`ref.current` 改了但 React 不知道。需要 re-render 的值用 `useState`。

**2. 在 render 過程中讀寫 ref**

```tsx
// 錯誤：render 期間 mutate，破壞 React 18 的 concurrent mode 假設
function Component() {
  ref.current = something;
  return ...;
}
```

ref 的讀寫應該在 event handler、useEffect、或 cleanup 裡進行。render 過程要保持 pure。

**3. DOM ref 在 render 期間取不到**

```tsx
// 錯誤：第一次 render 時 ref.current 還是 null
function Component() {
  const ref = useRef<HTMLDivElement>(null);
  const width = ref.current?.offsetWidth; // null
  return <div ref={ref}>...</div>;
}

// 正確：useEffect 裡才能讀
useEffect(() => {
  const width = ref.current?.offsetWidth;
}, []);
```

DOM ref 在元件 mount 後才會被填值。

**4. 把所有東西都塞進 ref**

ref 不是 state 的替代品。需要驅動 UI 的值用 useState；只是內部記憶體的值才放 ref。混在一起用很容易讓元件行為變得難以追蹤。

**5. 對 functional component 的 ref 沒包 forwardRef**

```tsx
// 錯誤：functional component 無法接收 ref
const Input = ({ ... }) => <input ... />;
<Input ref={ref} />; // ref 是 null

// 正確
const Input = forwardRef<HTMLInputElement>((props, ref) => (
  <input ref={ref} {...props} />
));
```

React 19 之後 `ref` 可以直接當 prop 傳，不用 forwardRef；舊版本還是需要。

---

## useContext：跨層級的狀態傳遞

### 定義

`useContext` 從最近的 `Context.Provider` 讀取資料。配合 `React.createContext` 使用，可以避開「props drilling」（一層一層往下傳 props）。

```ts
const value = useContext(MyContext);
```

只要元件包在某個 Provider 底下，無論深度多深，都能直接拿到值。

### 用途

- 主題切換（dark / light mode）
- 多語系（i18n）
- 登入使用者資訊
- 全域設定、feature flag
- 輕量級的全域狀態（搭配 useReducer）

如果狀態變動非常頻繁，所有訂閱的元件都會 re-render，需要考慮拆分 context 或用 Zustand / Jotai 之類的細粒度方案。

### 完整範例

主題系統 + 全域使用者：

```tsx
import { createContext, useContext, useState, useMemo, ReactNode } from 'react';

type Theme = 'light' | 'dark';
type ThemeContextValue = {
  theme: Theme;
  toggle: () => void;
};

const ThemeContext = createContext<ThemeContextValue | null>(null);

export function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<Theme>('light');

  const value = useMemo<ThemeContextValue>(
    () => ({
      theme,
      toggle: () => setTheme((t) => (t === 'light' ? 'dark' : 'light')),
    }),
    [theme]
  );

  return (
    <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>
  );
}

export function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error('useTheme must be used inside ThemeProvider');
  return ctx;
}

// 使用
function Header() {
  const { theme, toggle } = useTheme();
  return (
    <header className={theme}>
      <button onClick={toggle}>Toggle Theme ({theme})</button>
    </header>
  );
}

function App() {
  return (
    <ThemeProvider>
      <Header />
      {/* 其他元件 */}
    </ThemeProvider>
  );
}
```

關鍵：

- Context value 用物件包起來，方便擴充
- 用 `useMemo` 穩定 reference，避免 Provider 每次 render 都產生新物件
- 用 custom hook（`useTheme`）包裝 `useContext`，避免每次都寫 null check
- Provider 放在最上層，整個 app 都能讀到

搭配 useReducer 變成輕量級 store：

```tsx
const StoreContext = createContext<{
  state: State;
  dispatch: React.Dispatch<Action>;
} | null>(null);

export function StoreProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(reducer, initialState);
  const value = useMemo(() => ({ state, dispatch }), [state]);
  return <StoreContext.Provider value={value}>{children}</StoreContext.Provider>;
}
```

### 常見陷阱

**1. value 每次都是新物件 → 全部訂閱者 re-render**

```tsx
// 錯誤：每次 render value 都是新物件
<ThemeContext.Provider value={{ theme, toggle }}>

// 改進：用 useMemo 穩定 reference
const value = useMemo(() => ({ theme, toggle }), [theme]);
<ThemeContext.Provider value={value}>
```

Context 變更時，所有用 `useContext` 訂閱的元件都會 re-render。如果 value 每次都是新物件，等於整個 app 一直在 re-render。

**2. 拿不到值卻沒 default 處理**

```tsx
const ctx = useContext(ThemeContext); // 可能是 null
ctx.theme; // 💥 runtime crash
```

兩種解法：

- 給 default value：`createContext({ theme: 'light', toggle: () => {} })`
- 用 custom hook 統一檢查（如上面的範例）

**3. 把所有 state 都塞同一個 context**

```tsx
// 反模式：user / cart / theme 都擠在一起
<AppContext.Provider value={{ user, cart, theme, ... }}>
```

只要任一欄位變動，全部訂閱者都重畫。應該按變動頻率與用途拆成多個 context。

**4. 把 context 當 Redux 用**

對中小型 app 沒問題，但複雜場景下少了 middleware、time-travel、selector，debug 會比較吃力。狀態邏輯超過一定規模，還是考慮 Zustand / Redux Toolkit。

**5. 在 Provider 外面用 hook**

```tsx
function App() {
  return (
    <div>
      <Header /> {/* useTheme 在這呼叫會拿不到值 */}
      <ThemeProvider>...</ThemeProvider>
    </div>
  );
}
```

Provider 一定要包在使用者外層。在 monorepo 或測試環境忘記包 Provider 是很常見的坑。

---

## 小結

7 個 hook 的快速摘要：

| Hook | 用途 | 何時用 |
|---|---|---|
| `useState` | 元件內部狀態 | 表單、UI flag、簡單狀態 |
| `useEffect` | 副作用 | API 呼叫、訂閱、DOM 操作 |
| `useReducer` | 複雜狀態邏輯 | 多欄位表單、wizard、購物車 |
| `useMemo` | 快取計算結果 | 昂貴運算、穩定 reference |
| `useCallback` | 快取函式 | 配合 React.memo、useEffect 依賴 |
| `useRef` | 可變容器 | DOM 存取、timer ID、不觸發 re-render 的值 |
| `useContext` | 跨層級狀態 | 主題、i18n、全域設定 |

幾個跨 hook 的共通原則：

- **依賴陣列要誠實**：缺東西就會拿到 stale value，多餘的會導致不必要 re-run。裝 `eslint-plugin-react-hooks` 強制執行
- **預設不要 memo**：useMemo / useCallback / React.memo 都不是免費，先用 profiler 找瓶頸再加
- **更新 state 用 functional update**：`setX(prev => ...)` 比 `setX(x + 1)` 安全
- **副作用只放 useEffect**：reducer 不能有副作用，useMemo 也不能
- **Hook 只能在 top level 呼叫**：不能放在 if、for、callback 裡。React 用呼叫順序對應狀態
- **不確定就讀官方文件**：React 文件 (react.dev) 寫得非常好，比大部分二手教學清楚

熟練之後，會發現 hook 之間是組合性的：custom hook 把這些 building block 包成業務邏輯，例如 `useDebounce`、`usePrevious`、`useLocalStorage`、`useFetch`。掌握 7 個內建 hook 是寫好 custom hook 的基礎。

新版的 React 19 之後又多了 `use`、`useActionState`、`useOptimistic`、`useFormStatus` 等新 hook，但底層心智模型還是相通的：把元件視為「state → UI」的純函式，副作用放外面、衍生值現算、Hook 順序固定。掌握了這些原則，新 hook 學起來就很快。

(fin)
