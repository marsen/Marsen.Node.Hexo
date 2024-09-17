---
title: " [學習筆記] JavaScript function 幾種不同的寫法"
date: 2022/01/13 11:59:54
---

## 回傳純值的函數

```javascript
//return value
function foo1(a, b) {
  return a + b;
}
// arrow function
let foo2 = (a, b) => {
  return a + b;
};
// skip { } and return
let foo3 = (a, b) => a + b;

console.log("foo1(1,2) is", foo1(1, 2));
console.log("foo2(1,2) is", foo2(1, 2));
console.log("foo3(1,2) is", foo3(1, 2));
```

## 回傳物件的函數

```javascript
//return object

function foo4(a, b) {
  return { sum: a + b };
}
// arrow function
let foo5 = (a, b) => {
  return { sum: a + b };
};
// add () so you can skip { } and return
let foo6 = (a, b) => ({ sum: a + b });

console.log("foo4(1,2) is", foo4(1, 2));
console.log("foo5(1,2) is", foo5(1, 2));
console.log("foo6(1,2) is", foo6(1, 2));
```

## 科里(Curry)化函數

```javascript
//return function, execute foo7(1,2)()
let foo7 = (a, b) => () => a + b;
console.log("foo7(1,2) is", foo7(1, 2)());
//curry
let foo8 = (a) => (b) => a + b;
console.log("foo8(1,2) is", foo8(1)(2));
```

(fin)
