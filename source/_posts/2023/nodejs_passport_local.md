---
title: " [實作筆記] passport 與 passport-local 原始碼分析"
date: 2023/02/08 04:41:13
tags:
  - 實作筆記
---

## 前言

最近接觸的專案有一部份是 express 與登入機制有相關
使用的套件是 [Passport](https://www.passportjs.org/)
Passport 是這樣介紹自已的

> Simple, unobtrusive authentication for Node.js
> Passport is authentication middleware for Node.js. Extremely flexible and modular,  
> Passport can be unobtrusively dropped in to any Express-based web application.  
> A comprehensive set of strategies support authentication using a username and password, Facebook, Twitter, and more.

這裡我們不作使用教學，官方都有而且相當簡單。
`passport.authenticate` 本質上就是一個 middleware。
而 passport.use 會用來註冊各種 Strategy。
比如說 facebook login、google login 等…
更多可見[這裡](https://www.passportjs.org/packages/)

## 談談 passport-local

開發的過程中，有小朋友反應，如果他不輸入帳密，
頁面就會直接轉導，無法看到錯誤訊(有關 req.flash 的部份，本文不會談到)。

```javascript
passport.authenticate("local", {
  successRedirect: "/",
  failureRedirect: "/users/login",
  failureFlash: true,
});
```

當然我們很清楚有設定 `failureRedirect`  
不過讓我們不得其解的是，我們寫的整個 `LocalStrategy` 並沒有被執行，連 log 都沒有

這裡要翻一下[原碼](https://github.com/jaredhanson/passport-local/blob/master/lib/strategy.js)，  
這個作者的專案我覺得很棒，大多數都有測項，
除了文件外，你可以直接看測項理解它的想法。
在 Strategy.prototype.authenticate 可以發現以下程式

```javascript
if (!username || !password) {
  return this.fail(
    { message: options.badRequestMessage || "Missing credentials" },
    400
  );
}
```

簡單的說，它預設回傳一個 400 錯誤，而我們又設定了一個錯誤轉導。  
使用這類的第三方套件就怕這類的非預期行為，  
好在這個套件的文件與測項開源且清楚，我們才能快速定位問題。

## 參考

- [[筆記] 透過 Passport.js 實作驗證機制](https://medium.com/%E9%BA%A5%E5%85%8B%E7%9A%84%E5%8D%8A%E8%B7%AF%E5%87%BA%E5%AE%B6%E7%AD%86%E8%A8%98/%E7%AD%86%E8%A8%98-%E9%80%8F%E9%81%8E-passport-js-%E5%AF%A6%E4%BD%9C%E9%A9%97%E8%AD%89%E6%A9%9F%E5%88%B6-11cf478f421e)
- [Express 使用中介軟體](https://expressjs.com/zh-tw/guide/using-middleware.html)

(fin)
