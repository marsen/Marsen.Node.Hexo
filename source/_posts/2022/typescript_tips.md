---
title: "[活動筆記]線上活動 TypeScript tips and Tricks with Matt"
date: 2022/05/15 10:00:57
---

## 前情提要

YouTube 頻道[Visual Studio Code](https://www.youtube.com/channel/UCs5Y5_7XK8HLDX0SLNwkd3w)  
前陣子推出了由大神 Matt 所介紹有關 TypeScript 奇技淫巧[影片](https://www.youtube.com/watch?v=hBk4nV7q6-w)  
相當燒腦，特別作此記錄。

## 5:06 [generic-table-component](https://aka.ms/typescript/1)

難度:★★☆☆☆

這題需要有一些 React 的基礎概念，  
首先我們有一些 React Component(以下簡稱 RC)，  
並且用 interface 建立相對應 props。

問題是如果要讓我們的 RC 更泛用應該怎麼作，  
在這個例子中，我們建立了一個 Table RC(以下簡稱 Table) ，並且在 props 中傳入一些參數，  
這個參數的型別，直接與 Table 產生了耦合，我們可用泛型(Generic)處理。

### Before

```javascript
interface TableProps {
  items: { id: string }[];
  renderItem: (item: { id: string }) => React.ReactNode;
}

export const Table = (props: TableProps) => {
  return null;
};
```

### After

```tsx
interface TableProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
}

export const Table = <T,>(props: TableProps<T>) => {
  return null;
};
```

這裡是很簡單的泛型觀念，如果有寫過具備泛型的語言(ex:C#)應該不難理解，  
另外需要注意的一個有關 React 的小小 Tricky  
注意到 `<T,>` 這裡有一個小小的逗號，其實是不必要的，  
但是在 React 的開發環境之中，角括號`<>`會被視為未封閉的 RC;  
封閉的例子，ex:`<myComponent></myComponent>` 或是 `<myComponent />`  
未了必免編輯器的警告，需要加上一個逗號`,`

另外這裡在寫 React Component 時，使用了 arrow function，  
我們也可以換成一般的 function 寫法,這種寫法就可以省略 `<T,>` 這樣 tricky 的寫法

```tsx
export const Table = function <T>(props: TableProps<T>) {
  return null;
};
```

泛型不難理解，但是考量到需要具備的 React 知識，難度我給他兩顆星

## 11:19 [get-deep-value](https://aka.ms/typescript/2)

難度:★★★☆☆

這題要動態的取出物件深層的屬性的型別，  
具體好處是可以讓編輯型別檢查

看一下影片中的範例 `getDeepValue` 是一個函數，可以用來取物件深層屬性的值  
這裡我們看到 `obj:any` Matt 說道可以視作一個壞味道(不知道他對 unknown 的看法)

```javascript
export const getDeepValue = (obj: any, firstKey: string, secondKey: string) => {
  return obj[firstKey][secondKey];
};
```

看一下原始範例如下

```javascript
const obj = {
  foo: {
    a: true,
    b: 2,
  },
  bar: {
    c: "12",
    d: 18,
  },
};
```

### 我的作法

如果比較直觀的寫法可以考慮寫個 ObjectType,

```typescript
// this one is NOT from the sample

const obj: ObjType = {
  foo: {
    a: true,
    b: 2,
  },
  bar: {
    c: "12",
    d: 18,
  },
};

type FirstKeyType = "foo" | "bar";
type SecondKeyType = "a" | "b" | "c" | "d";
type ObjType = {
  [key in FirstKeyType]: { a: boolean; b: number } | { c: string; d: number };
};
export const getDeepValue = (obj: ObjType //...
```

註:這裡有用到 [index-signatures](https://www.typescriptlang.org/docs/handbook/2/objects.html#index-signatures) 的概念  
看看官方文件的說法:

```text
Sometimes you don’t know all the names of a type’s properties ahead of time, but you do know the shape of the values.

In those cases you can use an index signature to describe the types of possible values
```

```javascript
const value = getDeepValue(obj, "foo", "a");
```

### Matt 在影片中的作法

```typescript
export const getDeepValue = <
  TObj,
  TFirstKey extends keyof TObj,
  TSecondKey extends keyof TObj[TFirstKey]
>(
  obj: TObj,
  firstKey: TFirstKey,
  secondKey: TSecondKey
) => {
  return obj[firstKey][secondKey];
};
```

[keyof 關鍵字可以直接參考官方文件](https://www.typescriptlang.org/docs/handbook/2/keyof-types.html#handbook-content)

### 比較兩者的作法

我們加上了 Type，就是要為程式碼加上保護  
我跟 Matt 的作法，下面的程式碼都會提出警告

```typescript
var error_value = getDeepValue(obj, "error", "wrong");
```

但是無法避免以下的錯誤

```typescript
var error_value = getDeepValue(obj, "foo", "c");
```

`obj.foo` 是不存在 `c` 屬性，這樣我們就需要動態判斷型別
看看 Matt 使用 `keyof` 作法。

```typescript
export const getDeepValue = <TObj, TFirstKey extends keyof TObj>(
  obj: TObj,
  firstKey: TFirstKey,
  secondKey: keyof TObj[TFirstKey]
) => {
  return obj[firstKey][secondKey];
};
```

這裡有點不可思議，首先是 ts 的動態型別推導，  
在呼叫方法 getDeepValue 的第 1 個參數會動態推導出 TObj 的型別  
大概等價以下的型別

```typescript
type TObj = {
  foo: {
    a: boolean;
    b: number;
  };
  bar: {
    c: string;
    d: number;
  };
};
```

`keyof TObj` 的值會是　`foo | bar`  
而 TFirstKey 的型別，在第 2 個參數傳入之時決定是 `foo` 或 `bar`，  
不是上述兩種的參數傳入時，編輯會拋出錯誤警告，非常好用。  
下一個燒腦的部份，第 3 個參數 `secondKey: keyof TObj[TFirstKey]`，  
透過類似陣列取值的手法，我們可以在傳入第 2 個參數時決定第 3 個參數的值，並且讓編輯器提供保護。  
例如，第 2 個參數為 `foo` 時，第 3 個參數會限定只能傳入　`"a"|"b"`

這裡需要思考一下，並不難理解，所以給 3 顆星。

如果能應用得好的話，對於開發 ts Library 會相當有幫助  
不過我覺得自已無法用得很得心應手，比如說，如果物件深度不固定，應該如處理？  
又或者如果不用 `keyof` 的技巧，對一個已知的物件，我能不能作到類似的效果(編輯器動態檢查)?

## 28:54 [conditional-types](https://aka.ms/typescript/3)

難度:★★★☆☆

這其實是 TypeScript 一個基本的概念與技巧，  
可以在[官方文件](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)看到更多資訊

一個標準的寫法如下

```typescript
  SomeType extends OtherType ? TrueType : FalseType;
```

記得這裡最終只會回傳 TrueType 或是 FalseType，而 SomeType 只是作為條件檢查。
當然也可以作多重的條件檢查

```typescript
  SomeType extends OtherType ?
    TrueType :
    SomeType extends OtherConditionType?
    OtherThing :
    FalseType;
```

### 35:20 更進一步

我們看一下怎麼應用 Condition Type 這個技巧?
參考[error-messages-in-ts](https://aka.ms/typescript/9)這個教案  
在 deepEqualCompare 這個方法之中，陣列比較需要例外處理，
因為陣列比較永遠會回傳 false
原本的作法，會在程式的 runtime 作型別檢查並拋出例外

```typescript
export const deepEqualCompare = <Arg>(a: Arg, b: Arg): boolean => {
  // runtime handle
  if (Array.isArray(a) || Array.isArray(b)) {
    throw new Error("You cannot compare two arrays using deepEqualCompare");
  }
  return a === b;
};
```

而透過 condition type，可以讓編輯器在開發時直接檢查，看起來更加的優雅

```typescript
export const deepEqualCompare = <Arg>(
  a: Arg extends any[] ? "not allow array" : Arg,
  b: Arg extends any[] ? "not allow array" : Arg
): boolean => {
  return a === b;
};
```

這應該算是 TypeScript 的基本工，也有在官方文件上有很清楚的講解，
對我來說，第一次看到仍然相當驚訝，我給他三顆星
另外，我有兩個延申問題，

1. 程式碼中有 any 算不算壞味道? 我是不是應該用 `unknown[]` 取代 `any[]`
2. `Arg extends any[] ? "not allow array" : Arg` 明顯出現了重複，我有沒有什麼技巧可以消除這樣的重複?

## 其它

整個影片還有許多燒腦的部分，
Q&A 或是最後的挑戰都有相當多有趣的東西的可挖掘，
有機會我再補上。

## 參考

- [Type-Challenges](https://github.com/type-challenges/type-challenges/tree/main/questions)
- Matt's Tips & Tricks
  - [generic-table-component](https://aka.ms/typescript/1)
  - [get-deep-value](https://aka.ms/typescript/2)
  - [conditional-types](https://aka.ms/typescript/3)
  - [loose-autocomplete](https://aka.ms/typescript/4)
  - [compose](https://aka.ms/typescript/5)
  - [conditional-function-args](https://aka.ms/typescript/6)
  - [distributive-conditional-types](https://aka.ms/typescript/7)
  - [object-values](https://aka.ms/typescript/8)
  - [error-messages-in-ts](https://aka.ms/typescript/9)
  - [search-params-decoding](https://aka.ms/typescript/10)
- [Today in TypeScript: the keyof operator](https://dev.to/samwightt/today-in-typescript-the-keyof-operator-445b)
- [zod](https://www.npmjs.com/package/zod)
- [ts-toolbelt](https://www.npmjs.com/package/ts-toolbelt)
- [Matt 目前的專案 stately](https://stately.ai/)
- [Matt 的推特](https://twitter.com/mpocock1)

(fin)
