---
title: "[翻譯] react-router 的三種渲染方法(component、render、children)"
date: 2021/10/12 11:24:46
---

## 前情提要

本文內容大量參考此[系列文章](https://dev.to/raaynaldo/react-router-three-route-rendering-methods-component-render-and-children-2eng), 僅作記錄之使用。
在學習 React 的過程中, 我們需要處理瀏覽器的網址(URL)與頁面之間的關係,  
目前(2021 年)主流的作法，就是使用 react-router 這個 library,  
其中有三個類似的方法

## Rendering Method

### component

使用方法:

```jsx
<Route path="/" component={Home} />
//Same as
<Route path="/" >
  <Home />
</Route>
```

這個方法的缺點是並沒有提供傳遞 `props` 的 API

```jsx
const Home = (props) => {
  console.log(props);
  return <div>Home</div>;
};
```

### render

Render 這方法要求你使用一個傳入一個回傳 component 的方法,  
我們可以透過方法參數傳遞 `props`

```jsx
  <Route
  path="/contact"
  render={(routeProps) => {
    return <Contact name={name} address={address} {...routeProps} />;
  }}
  />
```

### children

基本上使用方式與 `render` 並無二致,
最大的差異在染渲邏輯，`children` 在路由不匹配的時候, 仍然會顯示,
以下例子在使用者輸入 `/` 會顯示 `Portfolio` 與 `Contact`

```jsx
  <Route path="/" exact component={Home} />
  <Route path="/about" render={() => <About></About>} />
  <Route path="/portfolio" children={() => <Portfolio></Portfolio>} />
  <Route path="/contact" children={() => <Contact></Contact>} />
```

## 參考

- [Rendering Methods (component, render, and children)react-router: Three Route Rendering Methods (component, render, and children)](https://dev.to/raaynaldo/react-router-three-route-rendering-methods-component-render-and-children-2eng)

(fin)
