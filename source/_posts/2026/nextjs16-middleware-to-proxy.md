---
title: "[實作筆記] Next.js 16 — middleware 改名為 proxy 了"
date: 2026/06/16 14:28:30
tags:
  - 實作筆記
---

## 前情提要

升到 Next.js 16 之後，dev server 一直噴這個警告：

```text
⚠ The "middleware" file convention is deprecated.
  Please use "proxy" instead.
```

查了一下，原來 Next.js 16 把 `middleware` 整個改名了，記錄一下到底換了什麼。

## 為什麼改

Next.js 官方說，`middleware` 這個名字太模糊，
實際上它做的事情是**網路邊界的路由代理**，
改叫 `proxy` 更能表達它的職責。

## 改了哪些東西

| 項目 | 舊（deprecated） | 新 |
| --- | --- | --- |
| 檔名 | `middleware.ts` | `proxy.ts` |
| export 函式名 | `export function middleware` | `export function proxy` |
| config export | 不變 | 不變（`matcher` 一樣） |
| runtime | edge / nodejs | **只支援 nodejs** |

`config` 和 `matcher` 不用動，只有檔名和 function 名稱要換。

## 最重要的限制

新的 `proxy` **只跑 Node.js runtime，不支援 edge**。[官方文件](https://nextjs.org/docs/app/api-reference/file-conventions/proxy#runtime)說得很清楚（截至 Next.js 16 / 2026-06）：

> The `runtime` config option is not available in Proxy files. Setting the `runtime` config option in Proxy will throw an error.

不是暫時限制，是設計決策。官方的方向是讓開發者不依賴 middleware/proxy：

> Next.js is moving forward to provide better APIs with better ergonomics so that developers can achieve their goals without Middleware.

所以如果你目前的 `middleware.ts` 有設 `export const runtime = 'edge'`，或用了依賴 edge runtime 的套件，**先不要遷移**——繼續用 `middleware.ts` 也能跑，只是會有 deprecation 警告，等官方提供替代方案再說。

## 怎麼遷移

### 方式一：官方 codemod（推薦）

```bash
npx @next/codemod@latest middleware-to-proxy .
```

自動幫你改檔名和 function 名稱。

### 方式二：手動

```bash
mv src/middleware.ts src/proxy.ts
```

然後把 function 名改掉：

```ts
// 原本
export async function middleware(request: NextRequest) { ... }

// 改成
export async function proxy(request: NextRequest) { ... }
```

`NextRequest`、`NextResponse`、`config`、`matcher` 全部不用動。

## 小結

Next.js 16 把 `middleware` 改名叫 `proxy`，概念一樣，名字更準確。
遷移很簡單，用 codemod 一行搞定；
唯一要注意的是 edge runtime 目前（2026-06）不支援，有用到的先等等。

(fin)
