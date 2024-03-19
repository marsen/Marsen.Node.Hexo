---
title: " [學習筆記] TypeScript 的 Using 與 Symbol.dispose"
date: 2024/03/19 13:13:45
---

## 介紹

TypeScript 在 5.2 引入了一個新關鍵字 using - 可用於在離開作用域時使用 Symbol.dispose 函數處理任何內容。

看一下官方的說明

> TypeScript 5.2 introduces support for the forthcoming Explicit Resource Management feature in ECMAScript,  
> which aims to address the need for “cleaning up” after creating an object.  
> This feature allows developers to perform necessary actions such as closing network connections,  
> deleting temporary files, or releasing memory.

`using` 與 `await` 的目的在於釋放資源上會非常有用。

在舉例子之前先看一下新的全域 Symbol:`Symbol.dispose` 與 `Symbol.asyncDispose`,  
任何將函數分配給 Symbol.dispose 的物件都將被視為「資源」（“具有特定存留期的物件”），並且可以與 using 關鍵字一起使用。

舉例來說:

```typescript
{
  const getResource = () => {
    return {
      [Symbol.dispose]: () => {
        console.log('Hooray!')
      }
    }
  }

  using resource = getResource();
} // 'Hooray!' logged to console
```

當程式離開 resource 所在的 scope 後，就會觸發 `Symbol.dispose` 的 function

## 應用

比較常見的場景在於存取 DB、File System 等…  
我們現在的作法需要用 `try...finally` 進行處理

```typescript
export function processFile(path: string) {
    const file = fs.openSync(path, "w+");

    try {
        // use file...

        if (someCondition()) {
            // do some more work...
            return;
        }
    }
    finally {
        // Close the file and delete it.
        fs.closeSync(file);
        fs.unlinkSync(path);
    }
}
```

而改用 `using` 後可以變得如此簡單

```typescript
export function processFile(path: string) {
    using file = new TempFile(path);

    // use file...

    if (someCondition()) {
        // do some more work...
        return;
    }
}
```

## 參考

- [using Declarations and Explicit Resource Management](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-2.html)
- [What is the “using” keyword in Typescript?](https://medium.com/@InspireTech/what-is-the-using-keyword-in-typescript-2a20738599e3)
- [TypeScript 5.2's New Keyword: 'using'](https://www.totaltypescript.com/typescript-5-2-new-keyword-using)

(fin)
