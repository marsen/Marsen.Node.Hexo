---
title: " [學習筆記] 為前端框架 Component 建立 Props Typing 的幾種方式，以 React 為例"
date: 2024/02/24 20:11:01
---

## 前言

在開發 React 應用程式時(其它框架其實也適用)，  
使用 TypeScript 來強制定義 props 是一個常見的做法。  
然而，有許多不同的方法可以實現這一目標。這篇文章討論了三種主要的方法。

## 比較

### Inline Object Literals

在這個方法中，我們直接在函式的參數位置定義了 props 的型別，如下所示：  

```typescript
const Wrapper = (props: {
  children?: ReactNode;
}) => {
  return <div>{props.children}</div>;
};
```

優點：快速簡潔。  
缺點：不太適合長期維護。  

另一種解構的寫法會有更少的字數，  
它會有一個 `{}:{}`特殊寫法，前者表示傳入的參數，後者表示要被解構的物件與型別  
這樣的寫法最主要在 component 當中要使用傳入參數時可以省略 `prop.` 的語法，  
但是理解上會不會更困難(對於不熟悉的開發者而言)就見團隊見智了…  

```typescript
 
const Wrapper = ({
  children,
}: {
  children?: ReactNode;
}) => {
  return <div>{children}</div>;
};
```


### Type Aliases

在這個方法中，我們把 props 的型別定義抽取到一個類型別名中：  

```typescript
export type WrapperProps = {
  children?: ReactNode;
};

const Wrapper = (props: WrapperProps) => {
  return <div>{props.children}</div>;
};
```

優點：可以在其他文件中重用。  
缺點：在大型代碼庫中可能會讓 TypeScript 變慢。  

### Interfaces

在這個方法中，我們使用介面來定義 props：  

```typescript
export interface WrapperProps {
  children?: ReactNode;
}

const Wrapper = (props: WrapperProps) => {
  return <div>{props.children}</div>;
};
```

優點：性能較好，在大型代碼庫中表現較好。  
缺點：需要較多的代碼。  

## 小結

儘量使用 interface 會有較好的效能，同時可以共用這些代碼並幫助理解。  

## 參考

- <https://www.totaltypescript.com/react-props-typescript>
- [Destructuring assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

(fin)
