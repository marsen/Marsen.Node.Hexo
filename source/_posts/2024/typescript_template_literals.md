---
title: " [學習筆記] 探索 TypeScript 的 Template Literals "
date: 2024/09/30 15:35:00
tags:
  - 學習筆記
---
## 前言

什麼是 Template Literals 類型？  
在 TypeScript 中，Template Literals 類型是 ES6 引入的一個強大的功能。  
Template literals ，主要用來加強字串的操作，它提供了更靈活和易讀的字串插值方式，  
這不僅提高了代碼的可讀性，結合上 TypeScript **還可以增加了類型安全性**。  
本篇提供大量範例與適用場景以供測試，有任何問題歡迎提出。  

## 基礎應用(JS/TS　都適用)

### 字串插值 (String Interpolation)

傳統的字串拼接需要用 + 號，使用 Template Literals 可以讓拼接更簡單。

```js
const name = "Lin";
const age = 30;

// 傳統拼接
const message = "My name is " + name + " and I am " + age + " years old.";

// Template Literals 
const message2 = `My name is ${name} and I am ${age} years old.`;

console.log(message2);
```

### 多行字串 (Multi-line Strings)

Template literals 支援多行字串，可以避免使用 `\n` 或其他換行符號。

```js
const poem = `
  Roses are red,
  Violets are blue,
  Sugar is sweet,
  And so are you.
`;

console.log(poem);
```

### 內嵌表達式 (Embedded Expressions)

可以在字串中插入任意的 JavaScript 表達式，例如函數調用、運算等。

```js
const a = 5;
const b = 10;

console.log(`The sum of ${a} and ${b} is ${a + b}`);
```

### 條件判斷 (Conditional Statements)

結合三元運算符或簡單的 if 判斷，可以在字串中靈活地處理條件。

```js
const loggedIn = true;
const message = `You are ${loggedIn ? "logged in" : "not logged in"}.`;

console.log(message);
```

### 標記模板字串 (Tagged Templates)

標記模板允許你在插值之前處理字串，可以用於處理國際化、多語系或安全操作（如避免 XSS 攻擊）。

```js
function sanitizeHTML(literals, ...values) {
  let result = "";
  literals.forEach((literal, i) => {
    const value = values[i] ? String(values[i]).replace(/</g, "&lt;").replace(/>/g, "&gt;") : '';
    result += literal + value;
  });
  return result;
}

const userInput = "<script>alert('Hacked!')</script>";
// call function use Tagged Template Literal
const safeOutput = sanitizeHTML`<div>${userInput}</div>`;

console.log(safeOutput);  // <div>&lt;script&gt;alert('Hacked!')&lt;/script&gt;</div>
```

### 動態生成 HTML 或模版字串

當需要動態生成 HTML 或動態內容時，使用 Template Literals 可以讓結構更清晰。

```js
const list = ["Apple", "Banana", "Cherry"];

const html = `
  <ul>
    ${list.map(item => `<li>${item}</li>`).join('')}
  </ul>
`;

console.log(html);
```

## 進階應用 with Type

### 動態生成 URL 路徑

在構建 API 或動態路徑時，Template Literals 結合 TypeScript 類型檢查可以確保參數類型正確，避免拼接錯誤。

```ts
type Endpoint = "/users" | "/posts" | "/comments";

function createURL(endpoint: Endpoint, id: number): string {
  return `https://api.example.com${endpoint}/${id}`;
}

const url = createURL("/users", 123);
console.log(url); // https://api.example.com/users/123

// 若傳入不存在的路徑，TypeScript 會報錯
createURL("/invalid", 123); // Error: Argument of type '"invalid"' is not assignable to parameter of type 'Endpoint'.
```

可以作開發中的路由檢查。

```ts
type HTTPMethod = "GET" | "POST" | "PUT" | "DELETE";
type Route = `/api/${string}`;

function request(method: HTTPMethod, route: Route): void {
  console.log(`Sending ${method} request to ${route}`);
}

request("GET", "/api/users");  // 正確
request("POST", "/api/posts"); // 正確

// 這會觸發類型檢查錯誤
request("GET", "/invalidRoute"); // Error: Argument of type '"/invalidRoute"' is not assignable to parameter of type 'Route'.
```

相同的概念也可以來動態生成 SQL 查詢語句，同時確保參數的安全性與類型正確性。

```ts
type TableName = "users" | "posts" | "comments";

function selectFromTable(table: TableName, id: number): string {
  return `SELECT * FROM ${table} WHERE id = ${id}`;
}

const query = selectFromTable("users", 1);
console.log(query); // SELECT * FROM users WHERE id = 1

// 傳入錯誤的表名時，會被 TypeScript 類型檢查發現
selectFromTable("invalidTable", 1); // Error: Argument of type '"invalidTable"' is not assignable to parameter of type 'TableName'.
```

### 字串聯合類型構造器

一個常見的模式是將Template Literals 類型與聯合類型結合，這樣可以生成所有可能的組合。  
例如，假設我們有一組顏色和相應的色調：

```typescript
type ColorShade = 100 | 200 | 300 | 400 | 500 | 600 | 700 | 800 | 900;
type Color = "red" | "blue" | "green";
```

我們可以創建一個顏色調色板，代表所有可能的顏色和色調的組合：

```typescript
type ColorPalette = `${Color}-${ColorShade}`;
let color: ColorPalette = "red-500"; // 正確
let badColor: ColorPalette = "red"; // 錯誤
```

這樣，我們就得到了 27 種可能的組合（3 種顏色乘以 9 種色調）。  
又或者，在樣式生成工具或 CSS-in-JS 的場景中，Template Literals 可以結合類型系統來強化樣式生成工具的正確性。

```ts
type CSSUnit = "px" | "em" | "rem";
type CSSProperty = "margin" | "padding" | "font-size";

function applyStyle(property: CSSProperty, value: number, unit: CSSUnit): string {
  return `${property}: ${value}${unit};`;
}

console.log(applyStyle("margin", 10, "px"));  // margin: 10px;
console.log(applyStyle("font-size", 1.5, "em")); // font-size: 1.5em;

// 若單位或屬性錯誤，會觸發類型檢查錯誤
applyStyle("background", 10, "px"); // Error: Argument of type '"background"' is not assignable to parameter of type 'CSSProperty'.
```

### 另一個例子

在構建大型系統時，常常需要動態生成變量名稱或類型，  
這時可以使用 Template Literal Types 來幫助構造更具彈性的類型系統。

```ts
type Eventa = "click" | "hover" | "focus";
type ElementID = "button" | "input" | "link";

// 動態生成事件處理函數名稱 3x3 種型別檢查
type EventHandlerName<E extends Eventa, T extends ElementID> = `${T}On${Capitalize<E>}Handler`;
const buttonOnClickHandler: EventHandlerName<"click", "button"> = "buttonOnClickHandler";
const buttonOnHoverHandler: EventHandlerName<"hover", "button"> = "buttonOnHoverHandler";
const buttonOnFocusHandler: EventHandlerName<"focus", "button"> = "buttonOnFocusHandler";
const inputOnClickHandler: EventHandlerName<"click", "input"> = "inputOnClickHandler";
const inputOnHoverHandler: EventHandlerName<"hover", "input"> = "inputOnHoverHandler";
const inputOnFocusHandler: EventHandlerName<"focus", "input"> = "inputOnFocusHandler";
const linkOnClickHandler: EventHandlerName<"click", "link"> = "linkOnClickHandler";
const linkOnHoverHandler: EventHandlerName<"hover", "link"> = "linkOnHoverHandler";
const linkOnFocusHandler: EventHandlerName<"focus", "link"> = "linkOnFocusHandler";

const invalidHandler: EventHandlerName<"click", "button"> = "buttonOnHoverHandler"; // Error: Type '"buttonOnHoverHandler"' is not assignable to type '"buttonOnClickHandler"'.
```

### 參考網路上的例子

Template Literals 類型允許我們在 TypeScript 中插入其他類型到字符串類型中。  
例如，假設我們想要定義一個表示 PNG 文件的類型：

```typescript
type Excel = `${string}.xlsx`;
```

這樣，當我們為變量指定 Excel 類型時，它必須以 .xlsx 結尾：

```typescript
let new_excel: Excel = "my-image.xlsx"; // ✅正確
let old_excel: Excel = "my-image.xls"; // ❌錯誤
```

當字符串不符合定義時，TypeScript 會顯示錯誤提示，這有助於減少潛在的錯誤。  
我們可以確保字符串符合特定的前綴或中間包含特定子字符串。  
例如，若要確保路由以 `/` 開頭，我們可以這樣定義：

```typescript
type Route = `/${string}`;
const myRoute: Route = "/home"; // ✅正確
const badRoute: Route = "home"; // ❌錯誤
```

同樣的，如果我們需要確保字符串包含 ?，以便視為查詢字符串，我們可以這樣定義：

```typescript
type QueryString = `${string}?${string}`;
const myQueryString: QueryString = "search?query=hello"; // 正確
const badQueryString: QueryString = "search"; // 錯誤
```

此外，TypeScript 還提供了一些內建的實用類型來轉換字符串類型，例如 Uppercase 和 Lowercase，可以將字符串轉換為大寫或小寫：

```typescript
type UppercaseHello = Uppercase<"hello">; // "HELLO"
type LowercaseHELLO = Lowercase<"HELLO">; // "hello"
```

還有 Capitalize 和 Uncapitalize 這兩個實用類型，可以用來改變字符串的首字母大小寫：

```typescript
type CapitalizeMatt = Capitalize<"matt">; // "Matt"
type UncapitalizePHD = Uncapitalize<"PHD">; // "pHD"
```

這些功能展示了 TypeScript 類型系統的靈活性。

## 結語

Template Literals 類型是一個非常有用的特性，能夠幫助開發者在 TypeScript 中更精確地控制字符串類型。  
從定義特定格式的文件名到生成複雜的組合類型，它為代碼提供了更高的可讀性和安全性。  
如果你在開發過程中需要處理字符串模式，Template Literals 類型無疑是值得考慮的工具。

## 參考

- [Template literals (Template strings)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)
- [JavaScript 有趣的冷知識：tagged template literals](https://medium.com/onedegree-tech-blog/javascript-%E6%9C%89%E8%B6%A3%E7%9A%84%E5%86%B7%E7%9F%A5%E8%AD%98-tagged-template-literals-5ca9db71f066)

(fin)
