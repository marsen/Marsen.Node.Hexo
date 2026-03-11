---
title: "[實作筆記] Next.js Hydration 與隨機性：從問題到正確架構"
date: 2026/02/18 02:24:28
tags:
  - 實作筆記
---

## 前情提要

最近在開發 Next.js 專案時，有 A/B 測試的需求，結果遇到一個奇怪的錯誤：

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

問題在於 `Math.random()` 在 Server 和 Client 各自執行一次：

```text
Server  跑 Math.random() → 0.3 → 選 Layout A → 產生 HTML
Client  跑 Math.random() → 0.7 → 選 Layout B → 跟 Server 的 HTML 對不上 → 爆炸
```

## 初步解法：繞開 SSR，在前端處理

要解決 mismatch，核心思路是讓 `Math.random()` 只在 Client 端跑一次。有兩種做法：

### 做法一：dynamic({ ssr: false })

直接告訴 Next.js 這個 component 跳過 SSR：

```typescript
// ABComponent.tsx
'use client'
export default function ABComponent() {
  const variant = Math.random() < 0.5 ? 'a' : 'b'
  return <Layout variant={variant} />
}

// page.tsx
const ABComponent = dynamic(() => import('./ABComponent'), { ssr: false })
```

component 完全不在 Server 端跑，`Math.random()` 只在瀏覽器執行，沒有 mismatch。

### 做法二：useState + useEffect

讓 component 在 Server 和 Client 第一次都 render `null`，mount 後才隨機：

```typescript
const [variant, setVariant] = useState<string | null>(null)

useEffect(() => {
  setVariant(Math.random() < 0.5 ? 'a' : 'b')
}, [])

if (!variant) return null
```

兩邊第一次渲染結果一致，hydration 成功，mount 後才決定變體。

### 哪個比較好？

**做法一（dynamic）比較好。** 程式碼更簡單，意圖也更清楚：「這個 component 不需要 Server Side Render」。

做法二的 `useState + useEffect` 除了更繁瑣，還可能被部分嚴格的 lint rule 擋住，lint auto-fix 之後反而重新引入 hydration bug，修 A 破 B。

不過兩種做法有一個共同的致命問題：

**同一個用戶每次重整都重新隨機，這次看 A、下次看 B，A/B 測試數據完全沒意義。**

## 正確做法：Middleware 決定，Server Side 處理

讓 Server 在第一次請求時決定變體，存進 cookie，之後每次讀同一個值。

```text
Middleware 執行 Math.random()，結果存進 cookie
    ↓
Server Component 讀 cookie 取得變體值
    ↓
以 prop 傳給 Client Component
    ↓
Server render 和 Client hydration 讀到同一個 prop → 沒有 mismatch
```

### middleware.ts

```typescript
export function middleware(request: NextRequest) {
  const response = NextResponse.next()

  if (!request.cookies.get('ab-variant')) {
    const variant = Math.random() < 0.5 ? 'a' : 'b'
    response.cookies.set('ab-variant', variant, { httpOnly: true })
  }

  return response
}
```

### page.tsx（Server Component）

```typescript
import { cookies } from 'next/headers'

export default function Page() {
  const variant = cookies().get('ab-variant')?.value ?? 'a'
  return <Layout variant={variant} />
}
```

同一個用戶拿到同一個 cookie，每次看到同一個版本，A/B 測試數據才有意義。

## 小結

這次踩坑之後整理出一個原則：

**隨機、時間、用戶身份這類「非確定性資料」，應該從 Server 端注入，Client 只負責呈現。**

| 資料類型 | 不建議做法 | 建議做法 |
| --- | --- | --- |
| 隨機值（A/B 測試） | Client `useState + Math.random()` | Middleware 寫 cookie，Server 讀取傳入 |
| 當前時間 | Client `Date.now()` | Server props 傳入 |
| 用戶身份 | Client 讀 localStorage | Server 讀 session cookie |

下次再給我選的話，我會選用在 Server Side 決定隨機性，而不在前端。

(fin)
