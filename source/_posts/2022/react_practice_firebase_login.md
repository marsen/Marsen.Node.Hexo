---
title: " [實作筆記] React Practice (一) Firebase 與 React Login"
date: 2022/01/27 17:57:59
tags:
  - 實作筆記
  - React
  - Firebase
---

## 前情提要

本文不會介紹 [Firebase](https://firebase.google.com/) 與 [React](https://reactjs.org/).
依照這篇文章完成後，你會

- 透過 [CRA](https://create-react-app.dev/) 建立一個專案
- 透過 [MUI](https://mui.com/) 建立符合 [Material Design](https://material.io/design) 的登入畫面
- 建立 Firsbase 專案並且透過 Firsbase 完成登入登出的功能
- 建立登入後才能瀏覽的頁面，並且實作登入／登出功能，登出後將無法瀏覽

## Firebase 設定

### 建立 Firebase 專案

這裡的操作非常簡單，可以直接參考下面的影片。

{%youtube 6juww5Lmvgo %}

詳細的步驟如下

1. 進入 [firebase console](https://console.firebase.google.com/)
2. 選擇建立專案(Add Project)，並且為你的專案取一個名字
3. 進入剛剛建立的專案後，選擇建立應用程式(Add App)，  
   在這裡我們選擇 Web 類型的應用程式  
   建立完成後我們會得到一個範例檔如下, 部份機敏資料先上遮罩

```typescript
// Import the functions you need from the SDKs you need
import { initializeApp } from "firebase/app";
import { getAnalytics } from "firebase/analytics";
// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Your web app's Firebase configuration
// For Firebase JS SDK v7.20.0 and later, measurementId is optional
const firebaseConfig = {
  apiKey: "A****************",
  authDomain: "******.firebaseapp.com",
  projectId: "******",
  storageBucket: "******.appspot.com",
  messagingSenderId: "******",
  appId: "1:******:web:******",
  measurementId: "G-*********",
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);
const analytics = getAnalytics(app);
```

#### 抽出環境變數

在 React 當中的機敏資料,建議的作法是使用環境變數,  
我們可以建立一個`.env.local`提供給開發測試用,  
而 Production 環境請直接設定在環境變數中, 相關資料不會進版控.  
以下幾點注意事項

- 必需使用 `REACT_APP_*` 開頭才可讓 REACT APP 使用 ex:`REACT_APP_APIKEY`
- 一般的建議是全大寫
- 環境變數的更新不適用 Hot Reload，請重啟環境

### 啟用 Firebase 認証(Authentication)

1. 首先要建立 Sign-in method > 原生供應商 > 電子郵件/密碼
2. 建立第一個使用者 Users > Add User

## React

### Create React Application

我們透過 TypeScript 建立 React 專案

```shell
npx create-react-app my-app --template typescript
```

我們可以試著啟動專案看一下，

```shell
cd my-app
```

```shell
npm i
```

```shell
npm start
```

### 安裝相關的套件

#### emotion 相關

```shell
npm i @emotion/react @emotion/styled
```

#### material 相關

```shell
npm i @mui/material @mui/icons-material
```

#### firebase 相關

```shell
npm i firebase react-firebase-hooks
```

#### react-router-dom 相關

```shell
npm i react-router-dom@6
```

### 建立登入頁面

#### 登入頁面與路由

在 src 中建立 `pages` 資料夾，  
之後我們再建立 `Login.tsx` 檔案如下

```jsx
import React from "react";

export const Login = () => {
  return <>Login Page</>;
};
```

接下來我們要透過 react-router-dom 來建立路由
開啟 `index.tsx` 如下

```jsx=
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();

```

修改如下

```jsx=
import React from "react";
import ReactDOM from "react-dom";
import "./index.css";
import App from "./App";
import reportWebVitals from "./reportWebVitals";
import { BrowserRouter, Route, Routes } from "react-router-dom";
import { Login } from "./pages/Login";

ReactDOM.render(
  <React.StrictMode>
    <BrowserRouter>
      <Routes>
        <Route path="" element={<App />} />
        <Route path="login" element={<Login />} />
      </Routes>
    </BrowserRouter>
  </React.StrictMode>,
  document.getElementById("root")
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();
```

重新啟動專案，在瀏覽器輸入不同的網址將會看到不同的畫面，如此我們完成了頁面的路由。

#### 快速建立符合 MUI 的登入頁面

我們可以透過 MUI 提供的 [Template](https://mui.com/getting-started/templates/) 來建立 [Login](https://mui.com/getting-started/templates/sign-in-side/) 頁面  
實作細節如下:

- 參考 [Source Code](https://github.com/mui-org/material-ui/tree/master/docs/src/pages/getting-started/templates/sign-in-side) 建立 `src/components/SigninSide` 組件
- 修改 `src/pages/Login.tsx` 如下:

```jsx=
import SignInSide from "../components/SignInSide";

export const Login = () => {
  return <SignInSide />;
};
```

重新瀏覽登入畫面，就可以看到一個美觀的登入頁 .

#### 實作登入功能

首先請花點時間看一下 [Firebase 帳號密碼登入的文件](https://firebase.google.com/docs/auth/web/password-auth#sign_in_a_user_with_an_email_address_and_password)。
接下來我們將依照文件的解釋修改我們的 SigninSide 組件。
打開 SigninSide 組件，找到`handleSubmit` 函數如下：

```jsx=
  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    const data = new FormData(event.currentTarget);
    // eslint-disable-next-line no-console
    console.log({
      email: data.get("email"),
      password: data.get("password"),
    });
  };
```

修改如下

```jsx=
  import { getAuth, signInWithEmailAndPassword } from "firebase/auth";
  import { app } from "../firebase-config";
  ....
  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    const data = new FormData(event.currentTarget);
    const auth = getAuth(app);
    signInWithEmailAndPassword(
      auth,
      data.get("email")!.toString(),
      data.get("password")!.toString()
    )
      .then((userCredential) => {
        // Signed in
        const user = userCredential.user;
        console.log("user", userCredential.user);
        // ...
      })
      .catch((error) => {
        const errorCode = error.code;
        const errorMessage = error.message;
        console.log("error", error);
      });
  };
```

> 題外話，**data.get("email")!.toString()**
> 當中的 `!.` 運算子可以[參考 TypeScript Non-null assertion operator 的說明](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-0.html#non-null-assertion-operator)

用之前在 firebase 建立的帳密登入，登入成功的話可以在 Console 看到類似以下的資訊

```console
UserImpl {providerId: 'firebase', emailVerified: false, isAnonymous:
false, tenantId: null, providerData: Array(1), …}
```

#### 實作登入後進頁面的跳轉

這裡要使用 `react-dom-router` 提供的 hook `useNavigate`,  
注意 React Hook 的使用限制

- [只在最上層呼叫 Hook---不要在迴圈、條件式或是巢狀的 function 內呼叫 Hook。](https://reactjs.org/docs/hooks-rules.html#only-call-hooks-at-the-top-level)
- [只在 React Function 中呼叫 Hook---別在一般的 JavaScript function 中呼叫 Hook。](https://reactjs.org/docs/hooks-rules.html#only-call-hooks-from-react-functions)

CRA 建立專案時同時會幫我們安裝檢查用的 Lint 所以不用太擔心。
同時我們把 `getAuth(app)` 也提取到 `SignInSide` Component 之外

```jsx=
////getAuth 可以不用寫在 handleSubmit 中
const auth = getAuth(app);

export default function SignInSide() {
  ////遵循 React Hook 的規則在最上層呼叫 Hook
  const navigate = useNavigate();//取得 navigate
  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    const data = new FormData(event.currentTarget);
    signInWithEmailAndPassword(
      auth,
      data.get("email")!.toString(),
      data.get("password")!.toString()
    )
      .then((userCredential) => {
        // Signed in
        // console.log("user", userCredential.user);
        navigate("/");// navigate 到首頁
```

### 實作登出的功能

調整 `App.tsx` 如下，移除不相關的程式，建立一個 Button 並準備好 onClick 事件

```jsx
import { Button } from "@mui/material";
import "./App.css";
function App() {
  return (
    <div className="App">
      <Button variant="outlined" onClick={() => {}}>
        LOGOUT
      </Button>
    </div>
  );
}

export default App;
```

#### 路由設定，只有登入者才能看的頁面

透過 getAuth().currentUser 取得目前的登入者資料, 如果沒有登入者轉導到 Login 頁面,

```jsx=
import { getAuth } from "firebase/auth";
....
<React.StrictMode>
  <BrowserRouter>
    <Routes>
      <Route
      path=""
      element={getAuth().currentUser ? <App /> : <Navigate to="/login" /> }
      />
      <Route path="login" element={<Login />} />
    </Routes>
  </BrowserRouter>
</React.StrictMode>
```

不過這樣的作法會有問題，
getAuth().currentUser 是非同步取得的資訊，
所以你很有可能都會拿到 null 值，
在官方的建議作法是透過 [onAuthStateChanged](https://firebase.google.com/docs/auth/web/manage-users#get_a_users_profile) 註冊 Observer, `index.tsx` 會類似下面這樣

```jsx=
const auth = getAuth();

onAuthStateChanged(auth, (user) => {
  ReactDOM.render(
    <React.StrictMode>
      <BrowserRouter>
        <Routes>
          <Route path="" element={user ? <App /> : <Navigate to="login" />} />
          <Route path="login" element={<Login />} />
        </Routes>
      </BrowserRouter>
    </React.StrictMode>,
    document.getElementById("root")
  );

  // If you want to start measuring performance in your app, pass a function
  // to log results (for example: reportWebVitals(console.log))
  // or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
  reportWebVitals();
});
```

## Todo

- Firebase Email link(passwordless sign-in)
- Firebase Host Project
- PageNotFound
- 管理 Routes
- more...

(つづく)
