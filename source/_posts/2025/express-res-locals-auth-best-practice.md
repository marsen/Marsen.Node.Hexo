---
title: " [學習筆記] Express.js middleware auth 的業界標準(res.locals)"
date: 2025/07/25 11:17:04
---

## 前言

在 Express.js 認證系統中，常見的做法是在每個需要驗證的路由中直接解析 JWT token。  
但有一個更好的實作方式：使用中介軟體將認證結果存放在 `res.locals` 中。  
這不僅是官方建議的做法，也避免了重複解析 token 的效能問題。

## TL;DR

使用 `res.locals` 處理認證是 Express.js 的標準實作：

- 避免在每個路由重複解析 JWT token
- Auth.js、Passport.js 等主流函式庫都採用這種模式
- Express.js 官方文檔明確支持這種用法

## 直接解析 JWT 的缺點

### 重複工作的效能問題

如果每個路由都直接解析 JWT，會造成不必要的重複運算：

```javascript
// ❌ 在每個路由重複解析
app.get('/profile', (req, res) => {
  const token = req.headers.authorization?.split(' ')[1]
  const user = jwt.verify(token, JWT_SECRET) // 重複解析
  res.json({ user })
})

app.get('/orders', (req, res) => {
  const token = req.headers.authorization?.split(' ')[1]
  const user = jwt.verify(token, JWT_SECRET) // 又解析一次
  res.json({ orders: getUserOrders(user.id) })
})
```

### 程式碼重複與維護困難

每個需要認證的路由都要寫類似的 token 驗證邏輯，違反了 DRY 原則，也增加了維護成本。

### 錯誤處理不一致

不同路由可能有不同的 token 驗證錯誤處理方式，造成 API 回應不一致。

## 什麼是 res.locals？

根據 Express.js 官方文檔，`res.locals` 是一個物件，用來存放**請求範圍內的區域變數**，這些變數只在當前請求-回應週期中可用。

```javascript
// Express.js 官方範例
app.use((req, res, next) => {
  res.locals.user = req.user
  res.locals.authenticated = !req.user
  next()
})
```

## 主流函式庫的標準實作

### Auth.js 官方範例

Auth.js 官方文檔明確展示了使用 `res.locals` 的標準模式：

```javascript
import { getSession } from "@auth/express"

export function authSession(req, res, next) {
  res.locals.session = await getSession(req)
  next()
}

app.use(authSession)

// 在路由中使用
app.get("/", (req, res) => {
  const { session } = res.locals
  res.render("index", { user: session?.user })
})
```

### Passport.js 的實作模式

Passport.js 在認證成功時會設置 `req.user` 屬性，許多開發者會將此資訊複製到 `res.locals` 以便在視圖中使用：

```javascript
// 認證中介軟體
app.use((req, res, next) => {
  res.locals.user = req.isAuthenticated() ? req.user : null
  res.locals.isAuthenticated = req.isAuthenticated()
  next()
})
```

## 為什麼這是好實作？

### 1. 官方支持的標準

Express.js 官方文檔說明 `res.locals` 就是用來存放請求級別的資訊，如認證用戶、用戶設定等。這不是 hack 或 workaround，而是設計用途。

### 2. 生態系統共識

主流的認證函式庫都採用類似模式：

- Auth.js 直接在文檔中展示 `res.locals.session`
- Passport.js 社群普遍使用 `res.locals.user`
- 許多教學都展示將認證狀態存放在 `res.locals` 的模式

### 3. 分離關注點

```javascript
// 認證中介軟體 - 只負責認證
function authMiddleware(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1]
  
  if (token) {
    const decoded = jwt.verify(token, JWT_SECRET)
    res.locals.user = decoded
  }
  
  next()
}

// 路由處理器 - 只負責業務邏輯
app.get('/profile', authMiddleware, (req, res) => {
  const { user } = res.locals
  if (!user) {
    return res.status(401).json({ error: 'Unauthorized' })
  }
  res.json({ profile: user })
})
```

### 4. 視圖整合優勢

在使用模板引擎時，`res.locals` 的資料會自動傳遞給視圖：

```javascript
// 中介軟體設定
app.use((req, res, next) => {
  res.locals.currentUser = req.user
  next()
})

// 在 EJS/Pug 模板中直接使用
// <%= currentUser.name %>
```

## 常見的反模式

### 避免的做法

有些開發者認為過度使用 `res.locals` 會讓除錯變困難，但這通常是因為：

1. **濫用中介軟體模式** - 把業務邏輯也放在中介軟體裡
2. **過度耦合** - 多個中介軟體互相依賴 `res.locals` 的順序

### 正確的使用原則

中介軟體應該用於所有 HTTP 請求共通的事項，且不包含業務邏輯：

```javascript
// ✅ 好的做法 - 純粹的認證檢查
function authenticate(req, res, next) {
  const token = getTokenFromRequest(req)
  const user = validateToken(token)
  res.locals.user = user
  next()
}

// ✅ 好的做法 - 授權檢查
function requireAuth(req, res, next) {
  if (!res.locals.user) {
    return res.status(401).json({ error: 'Unauthorized' })
  }
  next()
}

// ❌ 避免的做法 - 在中介軟體中處理業務邏輯
function badMiddleware(req, res, next) {
  const user = res.locals.user
  const orders = getUserOrders(user.id) // 業務邏輯不應該在這裡
  res.locals.orders = orders
  next()
}
```

## 實務上的最佳實作

### 標準認證中介軟體

```javascript
const jwt = require('jsonwebtoken')

function authMiddleware(req, res, next) {
  try {
    const token = req.headers.authorization?.replace('Bearer ', '')
    
    if (token) {
      const decoded = jwt.verify(token, process.env.JWT_SECRET)
      res.locals.user = decoded
      res.locals.isAuthenticated = true
    } else {
      res.locals.user = null
      res.locals.isAuthenticated = false
    }
    
    next()
  } catch (error) {
    res.locals.user = null
    res.locals.isAuthenticated = false
    next()
  }
}

// 授權檢查中介軟體
function requireAuth(req, res, next) {
  if (!res.locals.isAuthenticated) {
    return res.status(401).json({ error: 'Authentication required' })
  }
  next()
}

// 使用方式
app.use(authMiddleware)
app.get('/profile', requireAuth, (req, res) => {
  const { user } = res.locals
  res.json({ user })
})
```

### 與 req 自定義屬性的比較

雖然也可以使用 `req.user` 或 `req.data` 等自定義屬性，但 `res.locals` 有幾個優勢：

1. **語意清晰** - 明確表示這是給視圖用的資料
2. **自動傳遞** - 模板引擎會自動取用 `res.locals` 的資料
3. **標準化** - 社群共識，維護性更好

## 結論

`res.locals` 在認證系統中的使用確實是標準實作：

1. **官方支持** - Express.js 和 Auth.js 官方文檔都展示這種用法
2. **生態系統標準** - Passport.js 等主流函式庫採用相同模式
3. **效能考量** - 避免重複解析 JWT 或查詢資料庫
4. **開發體驗** - 控制器可以直接存取用戶資訊

所以下次有人說使用 `res.locals` 不是好實作時，可以拿出這些官方文檔來證明這確實是被廣泛接受的標準做法。

## 參考資料

- [Express.js Using middleware](https://expressjs.com/en/guide/using-middleware.html)
- [Auth.js Express Integration](https://authjs.dev/reference/express)
- [Passport.js Middleware Documentation](https://www.passportjs.org/concepts/authentication/middleware/)
- [Express.js res.locals API Reference](https://expressjs.com/en/api.html#res.locals)

(fin)
