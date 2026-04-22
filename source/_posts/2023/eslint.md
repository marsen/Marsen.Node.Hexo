---
title: " [實作筆記] 建立團隊可用的 ESLint(一)"
date: 2023/03/12 18:10:41
tags:
  - 實作筆記
  - ESLint
  - TypeScript
  - React
---

## 前情提要

在寫 JavaScript / TypeScript 的時候，常見的問題有三種：語法錯誤、風格不一致、潛在的 bug。  
ESLint 可以在開發期間即時抓出這些問題，並且強制團隊遵守同一套規範。

這篇的目標：在一個 React + TypeScript 專案裡，建立一套團隊可用的 ESLint 設定。  
具體來說要達成以下幾件事：

- 涵蓋 React 與 TSX 的語法檢查
- 套用現成的 style guide，不從零開始訂規則
- 讓任何人拉下專案就自動受到約束
- 之後還能整合進 CI/CD

選擇 **Airbnb** style guide——業界主流、規則嚴格，站在巨人的肩膀上，不用從零開始訂規範。

## 安裝與初始化

### 步驟 1：安裝 ESLint

```bash
npm install eslint --save-dev
```

### 步驟 2：初始化設定

```bash
npx eslint --init
```

init 過程是互動式問答，以下是我的選擇：

```text
✔ How would you like to use ESLint?
  ❯ To check syntax, find problems, and enforce code style

✔ What type of modules does your project use?
  ❯ JavaScript modules (import/export)

✔ Which framework does your project use?
  ❯ React

✔ Does your project use TypeScript?
  Yes

✔ Where does your code run?
  ✔ Browser

✔ How would you like to define a style for your project?
  ❯ Use a popular style guide

✔ Which style guide do you want to follow?
  ❯ Airbnb

✔ What format do you want your config file to be in?
  ❯ JavaScript

✔ Would you like to install them now? Yes
```

init 過程中選 Airbnb 後，它不會自動處理 TypeScript 支援，需要額外安裝：

```bash
npm install --save-dev eslint-config-airbnb-typescript \
  @typescript-eslint/eslint-plugin \
  @typescript-eslint/parser \
  eslint-plugin-import \
  eslint-plugin-jsx-a11y \
  eslint-plugin-react \
  eslint-plugin-react-hooks
```

完成後產生的 `.eslintrc.js` 要調整成這樣：

```js
module.exports = {
  env: {
    browser: true,
    es2021: true,
  },
  extends: [
    'airbnb',
    'airbnb-typescript',
    'airbnb/hooks',
  ],
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module',
    project: './tsconfig.json',
  },
  plugins: ['react'],
  rules: {},
}
```

### 步驟 3：調整規則

Airbnb 的規則偏嚴，第一次跑通常會噴出大量警告。建議先用 `--fix` 自動修正能修的：

```bash
npx eslint . --fix
```

剩下無法自動修正的，在 `rules` 裡逐一覆蓋：

```js
rules: {
  'no-console': 'warn',       // console.log 只警告不報錯
  'react/prop-types': 'off',  // 用 TypeScript 就不需要 prop-types
}
```

> 如果覺得 Airbnb 太嚴、導入阻力太大，也可以改用 [eslint-config-standard-with-typescript](https://github.com/standard/eslint-config-standard-with-typescript)，規則較寬鬆，不需要分號。

## 驗證設定是否生效

在 `package.json` 加入 script：

```json
"scripts": {
  "lint": "eslint src --ext .ts,.tsx",
  "lint:fix": "eslint src --ext .ts,.tsx --fix"
}
```

執行 `npm run lint`，確認規則有跑起來。

## 參考

- [eslint-config-airbnb-typescript](https://www.npmjs.com/package/eslint-config-airbnb-typescript)
- [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)
- [eslint-config-standard-with-typescript](https://github.com/standard/eslint-config-standard-with-typescript)（替代方案）

(fin)
