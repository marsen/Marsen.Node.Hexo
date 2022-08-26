---
title: " [實作筆記] Storybook CI 使用 Github Actions"
date: 2021/01/19
---

## 前情提要

在[前一篇](https://blog.marsen.me/2021/01/18/2021/storybook_typescript/)文章中，  
我們使用 TypeScript 開發 React Components ，
並使用 Storybook 作為測試的工具。

這篇會介紹如何與 chromatic 作結合，讓 CI/CD 運行時(本文將使用 Github Actions 作為 CI Server)，  
自動部署到 chromatic，同時提供自動化測試與人工審核的功能。

## 環境設置

使用 Github 登入 [Chromatic](https://www.chromatic.com/),  
雖然 Chromatic 也有提供 Bitbucket 與 GitLab 的登入方式,  
但並不確定這些 CI Server 包含 Jenkins、TravisCI 或是 CircleCI 實際上怎麼結合 Storybook,  
以下都以 Github 作介紹,

### 本機環境

安裝 `chromatic`

```terminal
yarn add -D chromatic
```

發佈 Storybook 到 Chromatic 上

```terminal
yarn chromatic --project-token=<project-token>
```

發佈完成你可以得到一個網址 `https://www.chromatic.com/builds?appId=random`
你可以分享網址給同事，對 UI 進行審查.
讓 Pull Request 時，自動執行的設定

### 雲端設定

新增專案後,可以取得 Token

![新增 Project](/images/2021/chromatic_add_project.jpg)
![取得 Token](/images/2021/chromatic_get_token.jpg)

在專案中設定 yaml 檔(Github Actions)  
加上 `.github/workflows/chromatic.yml`

```yaml
# .github/workflows/chromatic.yml
# name of our action
name: "Chromatic Deployment"
# the event that will trigger the action
on:
  # Trigger the workflow on push or pull request,
  # but only for the main branch
  push:
    branches:
      - main
# what the action will do
jobs:
  test:
    # the operating system it will run on
    runs-on: ubuntu-latest
    # the list of steps that the action will go through
    steps:
      - uses: actions/checkout@v2
      - run: cd src/marsen.react && yarn && yarn build && yarn build-storybook
      - uses: chromaui/action@v1
        with:
          projectToken: ${{ secrets.CHROMATIC_PROJECT_TOKEN }}
          token: ${{ secrets.GITHUB_TOKEN }}
          storybookBuildDir: storybook-static
```

### 特殊設定，子專案

如何你和我一樣, 專案是由多個子專案組成,  
那麼預設的 yaml 設定可能就不適合你.  
可以參考這個 [issue](https://github.com/chromaui/chromatic-cli/issues/197),  
其中要特別感謝 [yigityuce](https://github.com/yigityuce) 的 solution,  
我特別 fork 到我的 Github 帳號底下 [Repo](https://github.com/marsen/chromatic-cli)
設定調整如下:

```yaml
# 上略
steps:
  - uses: actions/checkout@v2
  - run: cd src/marsen.react && yarn && yarn build && yarn build-storybook
  - uses: marsen/chromatic-cli@v1
    with:
      workingDir: ./src/marsen.react
      projectToken: ${{ secrets.CHROMATIC_PROJECT_TOKEN }}
# 下略
```

### 驗收

如下圖, 左方會顯示舊版的 UI 畫面, 右方會顯示新版的 UI 畫面,  
如果開啟 Diff 功能(右上角的眼鏡圖示),  
即可以進行差異比對, 有差異的地方將以亮綠色顯示,  
如果認同這次的變更, 選擇右上角的 Accept 反之, 選擇 Deny.  
![驗收](/images/2021/chromatic_acceptance.jpg)

## 參考

- [Chromatic](https://www.chromatic.com/)
- [部署](https://www.learnstorybook.com/intro-to-storybook/react/en/deploy/)
- [玩轉 Storybook: Day 27 Design System for Developers - Review、Test](https://ithelp.ithome.com.tw/articles/10252055)

(fin)
