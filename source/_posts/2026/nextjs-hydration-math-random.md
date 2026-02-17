---
title: "[實作筆記] 簡單了解 Next.js Hydration"
date: 2026/02/18 02:24:28
---


## 前情提要

最近在開發 Next.js 專案時，有 A B 測試的需求，結果遇到一個奇怪的錯誤：

```text
Error: Hydration failed because the initial UI does not match what was rendered on the server.
```

追了一下，發現是 `Math.random()` 惹的禍。

## 問題根源

Next.js 的頁面渲染分兩個步驟：

第一步，Server 產生 HTML，送到瀏覽器，使用者馬上看到畫面，但還不能互動。

第二步，Client 載入 JavaScript，把事件綁上去，畫面變成可互動的。

第二步就叫 **Hydration**（注水）— 把靜態 HTML 注入互動能力，像幫乾燥的海綿注水。

Hydration 的核心規則：**Server 產生的 HTML 和 Client 產生的 HTML 必須一模一樣。**

Client 不是重新渲染，而是「接手」Server 的 HTML。如果兩邊不一樣，React 就會報 hydration error。

問題在於 `Math.random()` 在 Server 和 Client 各自執行一次：

```text
Server  跑 Math.random() → 0.3 → 選 Layout A → 產生 HTML
Client  跑 Math.random() → 0.7 → 選 Layout B → 跟 Server 的 HTML 對不上 → 💥
```

兩邊各自產生隨機數，結果不同，HTML 就不同，hydration 就炸了。

## 解法

最簡單的修法：用 `useState` + `useEffect`，讓隨機選擇只發生在 Client 端。

```typescript
const [variant, setVariant] = useState<string | null>(null);

useEffect(() => {
  setVariant(Math.random() < 0.5 ? "a" : "b");
}, []);

if (!variant) return null;
```

整個流程是這樣的：

**Server 渲染：** `variant = null` → return null → 產生空 HTML

**Client Hydration：** `variant = null` → return null → 跟 Server 一致，hydration 成功

**Client 掛載後（useEffect 執行）：** `Math.random()` → 0.3 → `setVariant("a")` → 重新渲染 → 顯示 Layout A

關鍵在於：Server 和 Client 第一次渲染都是 `null`，保持一致。隨機選擇延遲到 Client 掛載後才發生，完全繞開了 hydration 的限制。

## 小結

任何在 Server 和 Client 執行結果可能不同的程式碼，都不能直接放在 render 階段：

- `Math.random()`
- `Date.now()`
- `window`、`localStorage` 等 browser-only API

如果可以在前端執行，這些都要移到 `useEffect` 裡面，讓它只在 Client 掛載後執行。

如果一定要在 Server 端決定，解法是把隨機結果當成 props 傳下來，而不是讓 Client 自己再跑一次。

`useState` 的初始值設為 `null`，讓 Server 和 Client 第一次渲染時保持一致，這是解決 hydration mismatch 的標準思路。

(fin)
