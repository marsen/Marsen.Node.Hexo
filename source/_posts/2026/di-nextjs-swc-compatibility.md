---
title: "[實作筆記] 為什麼 InversifyJS 在 Next.js 不能用？SWC 相容性問題與 awilix 解法"
date: 2026/06/17 04:26:03
tags:
  - 實作筆記
---

## 前情提要

在做電商平台時，遇到一個問題：要同時支援多家金流（TapPay、ECPay、藍新），

所以我設計了多個不同的 PaymentService 來實作同一個 interface。

這算是 DI 的經典場景，自然就想到引入 DI container 來管理。

調查一輪之後，才發現 Next.js 的編譯器對這件事有一個很重要的限制。

## Next.js 預設用 SWC 編譯

SWC 是用 Rust 寫的 JavaScript/TypeScript 編譯器，Next.js 從 v12 開始改用它作為預設編譯器。

快很多，這是事實。但問題來了。

## InversifyJS 和 tsyringe 為什麼不能用

InversifyJS 和 tsyringe 是目前最主流的兩個 TypeScript DI framework，

兩個都靠裝飾器（decorator）做依賴注入：

```ts
@injectable()
class TapPayPaymentAdapter implements PaymentService {
  constructor(@inject('Logger') private logger: LoggerService) {}
}
```

這個模式需要兩個東西：

1. **`emitDecoratorMetadata`**：TypeScript 編譯時把型別資訊保留下來
2. **`reflect-metadata`**：在執行期讀取那些型別資訊

問題在於：**SWC 對 `emitDecoratorMetadata` 的支援不完整**。

SWC 的目標是快，不是完整複製 tsc 的行為。

`reflect-metadata` 依賴的 metadata 在 SWC 編譯後不保證存在，跑起來會出錯或行為異常。

要硬用 InversifyJS，得把 Next.js 的編譯器換回 tsc，放棄 SWC 的效能優勢。代價太高。

## awilix 為什麼可以

awilix 完全不用裝飾器，也不依賴 `reflect-metadata`。

它靠的是**命名慣例**：建構子參數的名稱對應 container 裡的 key。

```ts
// container 裡這樣註冊
container.register({
  loggerService: asClass(PinoLoggerService),
  tapPayService: asClass(TapPayPaymentAdapter),
  ecpayService: asClass(EcpayPaymentAdapter),
})

// adapter 建構子這樣寫
class TapPayPaymentAdapter {
  constructor({ loggerService }: { loggerService: LoggerService }) {}
}
```

名稱對上，awilix 自動注入。不需要編譯器幫你保留任何型別資訊，SWC 完全相容。

## 三家比較

| | InversifyJS | tsyringe | awilix |
| --- | --- | --- | --- |
| 週下載量 | 1.5M | 600K | 400K |
| GitHub Stars | 12K | 6K | 4.2K |
| 裝飾器 | 需要 | 需要 | 不需要 |
| reflect-metadata | 需要 | 需要 | 不需要 |
| Next.js / SWC | ❌ | ❌ | ✅ |
| 學習曲線 | 高 | 中 | 低 |
| Token/Symbol 保護 | ✅ | ✅ | ❌ 參數名即 key |
| Minification 安全 | ✅ | ✅ | ⚠️ 需額外設定 |
| 型別保護時機 | 編譯期 | 編譯期 | 執行期 |

下載量 InversifyJS 最高，但 Next.js 專案用不了。

awilix 能跑，代價是命名耦合與 minification 的隱患，小專案先忽略。

## 業界趨勢

2026 的調查顯示，小型 Node.js 專案的趨勢反而是**遠離 DI container**，

回到手寫的 module-level singleton 或手動 dependency passing。

這不是說 DI container 不好，而是：

- 依賴圖簡單時，手寫 container 清楚又好讀
- 依賴圖複雜到手寫開始讓人痛了，再引入才是真正有感的投資

我的考量是**早期建立團隊開發共識與原則**：

在 AI 時代，這樣技術學習門檻不高，早期引入就變成慣例，讓 AI 用乾淨架構去開發，才不會一沱。

## 小結

Next.js 用 SWC，SWC 不完整支援 reflect-metadata，所以 InversifyJS 和 tsyringe 不能用。

Next.js 專案要引入 DI container，awilix 是目前唯一合理的選擇。

(fin)
