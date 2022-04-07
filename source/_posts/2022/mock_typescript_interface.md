---
title: "[實作筆記] 簡單的方法 Mock TypeScript Interface"
date: 2022/04/08 02:20:08
---

## 前情提要

我目前在實作一個 React 的專案，並結合 Storybook 來實現 CDD(component driven development)，  
我很喜歡 CDD 的概念，這裡解決 TDD 對 View 難以趨動開發的痛點，  
但是又維持了 Unit Test 的基本精神，甚至結合 Chromatic 是一個相當有幫助的工具，  
可以作為與 PO/PM/UX/UI/DESIGNER 之間的溝通橋梁。

## 問題

我作了一個簡單的 Header 組件，
這通常是網站或 APP 上方的區塊，通常會提供一些額外的資訊或功能，  
例如: 登入/登出，而這裡我使用 Firebase 作為實作登入/登出的功能，  
作為主要的邏輯判斷，firebase 的 Auth 物件會被傳入

```tsx
Header = (props: { sighOut: () => void; auth?: Auth }) => {
  //...
};
```

這裡我們先不討論 Auth 的邏輯是否應該與 Header 相依(嗯，這裡看起來是個壞味道)，  
而是在我們的 Storybook 撰寫測試案例時，應該如何 mock Auth 物件。  
我的程式都是用 TypeScript 與 tsx 寫得，所以如果不符合型別，IDE 會直接報錯，程式跟本無法執行  

```tsx
const Template: ComponentStory<typeof Header> = () => (
  <Header
    sighOut={() => {
      alert("mocked Signout");
    }}
    auth={mockAuth}
  />
);
```

### 試過的方法

- jest Mock
- ts-mockito
- ts-auto-mock

以上的方法有得有成功，有得失敗，最大的問題是在配置與撰寫其實很麻煩，  
甚至會有同事說為什麼要用 TypeScript 自找麻煩呢 ?

逐一 mock 所有 interface 的屬性，對於一個像 Auth 這樣複雜的介面實作上相當麻煩，  
而且大多 mock 的屬性都用不到

我的想法是，應該是我對 TypeScript 與其生態圈不夠了解，寫起來才會覺得綁手綁腳的，
不能推缷給程式語言，同時 TypeScript 強型別的好處，其實也幫了我不少。
仔細研究後，我找到一個簡單方法如下:

## 解法

不需要 mock Library ， 不需要麻煩的設定或是第三方套件，
最簡單的寫法 `as` 就可以將物件 mock 成為你想要的 Interface

```tsx
const mockAuth = {
  currentUser: { email: "marsen@test.me" },
} as Auth;
```

這樣的寫法除了可以只 mock 必要的屬性外，  
如果你物件中包含了不屬於介面應有的屬性， TypeScript 將會提出警告，非常好用。

(fin)
