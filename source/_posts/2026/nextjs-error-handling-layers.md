---
title: "[實作筆記] Next.js 錯誤處理的三層：middleware、try-catch、instrumentation.ts"
date: 2026/06/18 02:22:10
tags:
  - 實作筆記
---

## 前情提要

在串接 ECPay 金流時，一個 API Route 裡用了 `redirect()`（from `next/navigation`），

結果 ECPay 的 callback 打進來之後整個 dev server 直接掛掉。

通常我會設計成主要的商業邏輯（包含錯誤的商業邏輯）走流程處理，

無法處理的由最終防線（通常會在 middleware 的後面）接住錯誤

主要的商業邏輯，儘可能少一點的 try-catch(原則上不用)

但 Next 還要考慮到 Edge Runtime, 借這個機會深入了解一下 Next 的錯誤處理機制

## Next.js 請求的三層

```text
HTTP Request
     │
     ▼
┌─────────────────────────────────────┐
│  middleware（Edge Runtime）          │  ← auth guard、redirect
│                                     │
│  出錯 → onRequestError 觸發         │  ← instrumentation 接得住
│       → 但 Pino logger 在這裡失效   │  ⚠️ Edge Runtime 限制
│       → Next.js 回 500             │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  API Route / Server Component       │
│  （Node.js，完整環境）               │
│                                     │
│  ┌─────────────────────────────┐    │
│  │  業務層 try-catch            │    │  ← 有業務理由才放
│  │  （ECPay→0|FAIL、OAuth→     │    │
│  │   /login?error=...）        │    │
│  └──────────────┬──────────────┘    │
│                 │ 未被接住           │
│                 ▼                   │
│  ┌─────────────────────────────┐    │
│  │  instrumentation.ts         │    │  ← 最終防線，全域自動生效
│  │  onRequestError             │    │
│  │  Node.js → Pino logger ✅   │    │
│  │  → Next.js 回 500           │    │
│  └─────────────────────────────┘    │
└─────────────────────────────────────┘
               │
               ▼
         HTTP Response
```

每一層的職責不一樣，不能混用。

## 第一層：middleware

Next.js 的 middleware， 跑在 **Edge Runtime**——這是一個故意閹割過的 JS 執行環境，

只有 Web 標準 API（`fetch`、`Request`、`Response`），沒有 Node.js 的東西。

所以 middleware 能做的事很有限：

```ts
// ✅ 可以：讀 cookie、做 redirect、檢查 JWT
// ❌ 不行：連資料庫、用 Pino logger、用大多數 npm 套件
```

這不是 Vercel 的限制，是 Next.js 的設計決策——middleware 在每個請求最前面跑，

設計目標是輕量、快，所以鎖死在 Edge Runtime。

**結論：middleware 只放輕量的守衛邏輯（auth check、redirect），不是錯誤處理的地方。**

## 第二層：業務層 try-catch

API Route 跑在完整 Node.js 上，可以做任何事。

但 try-catch 的原則是不使用，除非**有業務理由**。

什麼叫業務理由？

```ts
// ✅ ECPay webhook 協定要求：出錯必須回 0|FAIL，不能回 500
try {
  // ... 處理邏輯
  return new Response('1|OK', { status: 200 })
} catch (err) {
  logger.error('ECPay webhook error', { error: err })
  return new Response('0|FAIL', { status: 200 })
}

// ✅ OAuth 失敗：應該導到 /login?error=... 而不是白畫面
try {
  // ... OAuth 流程
} catch {
  return NextResponse.redirect(new URL('/login?error=server_error', request.url))
}

// ❌ 只是怕 crash，沒有業務意義
try {
  // ...
} catch (err) {
  return new Response('error', { status: 500 }) // 跟 Next.js 預設行為一樣，多此一舉
}
```

另外注意：在 API Route Handler 裡不能用 `redirect()` from `next/navigation`，那是給 Server Component / Server Action 用的。API Route 要用 `Response.redirect()`：

```ts
// ❌ API Route 裡用這個會讓 server crash
import { redirect } from 'next/navigation'
redirect('/some-page')

// ✅ 正確做法
return Response.redirect(`${baseUrl}/some-page`, 302)
```

## 第三層：instrumentation.ts

Next.js 15 起提供 `onRequestError`，是真正的全域最終防線。任何沒被 try-catch 接住的 exception 都會進來。

```ts
// src/instrumentation.ts
import type { Instrumentation } from 'next'

export const onRequestError: Instrumentation.onRequestError = async (
  err,
  request,
  context,
) => {
  const { getLoggerService } = await import('@/infrastructure/di/container')
  getLoggerService().error('Unhandled request error', {
    error: err.message,
    digest: err.digest,
    path: request.path,
    method: request.method,
    routeType: context.routeType,
    routePath: context.routePath,
  })
}
```

這個 hook 的好處：

- **全域自動生效**，新加的 route 不需要記得處理
- **API Route、Server Component、Server Action** 全部覆蓋
- 用 dynamic import 避免初始化順序問題

它不會改變 response（user 還是收到 500），但你起碼有 log 可以查根因。

## 陷阱：middleware 錯誤接得住，但 logger 失效

`onRequestError` 在 Edge Runtime 和 Node.js 都會觸發，包含 middleware 的錯誤（`context.routeType === 'proxy'`）。

但 Pino 是 Node.js 專用的，在 Edge Runtime 下會直接失敗。

要完整處理，需要根據 runtime 分開：

```ts
export const onRequestError: Instrumentation.onRequestError = async (
  err,
  request,
  context,
) => {
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    // Node.js：用 Pino 正常記 log
    const { getLoggerService } = await import('@/infrastructure/di/container')
    getLoggerService().error('Unhandled request error', {
      error: err.message,
      path: request.path,
      routeType: context.routeType,
    })
  } else {
    // Edge Runtime（middleware 錯誤）：只能用 console 或送外部服務
    console.error('[edge] Unhandled middleware error', err.message, request.path)
  }
}
```

這個專案的 middleware 只做 auth guard，邏輯薄、出錯機率低，目前先用 `console.error` 兜底即可。

## 三層的職責總結

| 層次 | 位置 | 做什麼 | 限制 |
| ------ | ------ | -------- | ------ |
| middleware | 請求最前面 | auth guard、redirect | Edge Runtime，無 DB/Logger |
| try-catch | 各 route 內 | 業務錯誤對應 | 只放有業務理由的 |
| instrumentation.ts | 全域最後 | 接住所有漏網錯誤、記 log | 無法改 response；Edge 下需另處理 |

## 參考

- [Next.js 官方文件：instrumentation.js — onRequestError](https://nextjs.org/docs/app/api-reference/file-conventions/instrumentation#onrequesterror-optional)

## 小結

不要為了「怕 crash」而到處加 try-catch，那只是把問題藏起來。

正確的順序是：先把 `instrumentation.ts` 設起來，確保所有未處理的錯誤都有 log，然後只在真的有業務需求的地方加 try-catch。

這樣新加的 route 自動有防護，不需要靠人記得。

(fin)
