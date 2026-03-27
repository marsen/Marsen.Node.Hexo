---
title: "[實作筆記] 在 Next.js 安全讀取 window：從 useEffect 到 useSyncExternalStore"
date: 2026/03/27 18:52:58
tags:
  - 實作筆記
---

## 前情提要

在做韓文注音學習工具時，需要偵測瀏覽器是否支援 `speechSynthesis`，然後顯示提示訊息。

```ts
const [speechSupported, setSpeechSupported] = useState(true);

useEffect(() => {
  setSpeechSupported("speechSynthesis" in window);
}, []);
```

寫完 CI 報錯：

```text
react-hooks/set-state-in-effect: Avoid calling setState() directly within an effect
```

第一反應是加 `eslint-disable`，但這樣太逃避。研究了一下，發現這個問題有更正確的解法。

---

## 為什麼不能直接讀 `window`

Next.js 的頁面先在 **Server** render 一次，再到 **Client** hydration。

Server 端沒有 `window`，所以這樣寫會直接炸：

```ts
// ❌ Server 端沒有 window
const supported = "speechSynthesis" in window;
```

`useEffect` 只在瀏覽器執行，所以在裡面讀 `window` 是安全的——這就是第一版寫法的出發點。

---

## useEffect + setState 的問題

```ts
const [speechSupported, setSpeechSupported] = useState(true);

useEffect(() => {
  setSpeechSupported("speechSynthesis" in window); // ← lint 報這裡
}, []);
```

這個寫法會造成兩次 render：

```text
第一次 render：speechSupported = true（初始值）
useEffect 執行：setSpeechSupported(真實值)
第二次 render：speechSupported = 真實值
```

ESLint 規則的意思是：「你在 `useEffect` 裡面直接 `setState`，造成 cascading render，能不能用更好的方式？」

它不是說這樣會壞掉，而是說有更乾淨的做法。

---

## 三種解法

### 解法一：eslint-disable（不推薦）

```ts
useEffect(() => {
  // eslint-disable-next-line react-hooks/set-state-in-effect
  setSpeechSupported("speechSynthesis" in window);
}, []);
```

問題沒有解決，只是把警告蓋掉。

---

### 解法二：useState lazy initializer

```ts
const [speechSupported] = useState(
  typeof window !== "undefined" ? "speechSynthesis" in window : true
);
```

Server 端 `typeof window === "undefined"` 為 `true`，所以初始值是 `true`。
Hydration 時 React **不重新執行** lazy initializer，直接沿用，所以不會 mismatch。

缺點：如果使用者的瀏覽器真的不支援 `speechSynthesis`，這個值永遠是 `true`，訊息永遠顯示不了。只適合「假設支援、不在乎誤判」的場景。

---

### 解法三：useSyncExternalStore（推薦）

React 18 為 Server/Client 差異設計了這個 hook：

```ts
import { useSyncExternalStore } from "react";

const speechSupported = useSyncExternalStore(
  () => () => {},                       // subscribe（不訂閱任何東西，no-op）
  () => "speechSynthesis" in window,    // Client 端的值
  () => true,                           // Server 端的值
);
```

三個參數分別對應：

| 參數 | 意義 |
|---|---|
| `subscribe` | 訂閱外部資料變更，不需要就給 no-op |
| `getSnapshot` | 瀏覽器端回傳什麼值 |
| `getServerSnapshot` | 伺服器端回傳什麼值 |

**流程：**

```text
Server render → 呼叫 getServerSnapshot → 回傳 true
Client hydration → 呼叫 getSnapshot → 回傳真實值
若兩者不同 → React 知道這是 client-only 差異，自動補一次 render，不報 hydration error
```

不需要 `useState`，不需要 `useEffect`，ESLint 不報錯，能正確偵測「真的不支援」的瀏覽器。

---

## 三種解法比較

| | `useEffect` + `setState` | `useState` lazy | `useSyncExternalStore` |
|---|---|---|---|
| Server 安全 | ✅ | ✅ | ✅ |
| Hydration 安全 | ✅ | ✅ | ✅ |
| 多餘 render | ❌ 有 | ✅ 無 | ✅ React 自動處理 |
| ESLint 通過 | ❌ | ✅ | ✅ |
| 能偵測真的不支援 | ✅ | ❌ | ✅ |

---

## 小結

`useSyncExternalStore` 名字聽起來像是給 Redux 這種外部 store 用的，但它的第三個參數 `getServerSnapshot` 就是為了 SSR/Client 差異設計的——哪怕你根本沒有要訂閱任何東西。

遇到「要讀 `window` 但又要 SSR 安全」的情境，先考慮這個 hook，不要直接跳到 `useEffect` + `setState`。

(fin)
