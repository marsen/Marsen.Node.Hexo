---
title: "[實作筆記] Hexo CI 自動執行 ncu -u 更新相依套件"
date: 2022/09/28 19:18:35
tags:
  - 實作筆記
  - CI/CD
  - Hexo
---

## 前情提要

參考[前篇](https://blog.marsen.me/2022/09/26/2022/Hexo_CICD/)，
我針對了我 Blog 文章的發佈流程作了改善，  
當中我提到其中並沒有用到多麼了不起的技術或新知，  
而是整合現有的技術，讓流程更加的自動化。

這次我又想到一個自動化的流程，
之前提到的 [npm-check-updates](https://blog.marsen.me/2022/09/08/2022/how_to_npm_update_more_smoothly/) 工具，
雖然可以很輕鬆的更新 npm 套件，但是什麼時候執行就是一個問題了，  
可以的話，我希望讓 CI/CD 代勞。

## 實作

立刻來捲起袖子吧，  
首先回顧一下手動作業的流程。

### 前置條件

全域安裝 npm-check-updates

### 流程

1. 拉取專案 `git pull`
2. 執行 `ncu -u`
3. 如果有異動的話 `git commit -m "some message here"`
4. 推上遠端 repo `git push`

### 自動化

看看上面的流程，我們知道其實我們只需要一個安裝好 node、git 與 ncu 的環境，

![必要條件](https://i.imgur.com/GAPHpsr.png)

接著就依照流程執行命令即可，這裡我覺得正可以展現 Github Workflow 與 Actions 的強大之處，  
你的問題，就是大眾會遇到的問題，而且大部份的情況都有對應的 Actions，  
我們所需要作的，就是組合這些步驟，並且放入自已的邏輯。
直接看 workflow 範例

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v2.4.2
  - name: ncu-upgrade
    run: |
      npm i -g npm-check-updates
      ncu -u
  - name: Commit & Push changes
    uses: actions-js/push@v1.3
    with:
      github_token: ${{ secrets.GITHUB_TOKEN }}
```

觀注在 steps 的部份，
透過 `actions/checkout` 取得最新的 remote repository

```yaml
- name: Checkout
  uses: actions/checkout@v2.4.2
```

接下來，執行一些語法，
安裝 `npm i -g npm-check-updates`  
再執行 `ncu -u`

最後，透過 `action-js/push` 將異動推回 remote repository

```yaml
- name: Commit & Push changes
  uses: action-js/push@v1.3
```

下一個問題是，要怎麼取得 git repo 的權限 ?
登登!! 我們可以使用 GITHUB_TOKEN 就輕易取得權限,
有關 GITHUB_TOKEN 的介紹可以參考下方的連結。

## 參考

- <https://blog.marsen.me/2022/09/26/2022/Hexo_CICD/>
- <https://blog.marsen.me/2022/09/08/2022/how_to_npm_update_more_smoothly/>
- [GitHub Actions: GITHUB_TOKEN Explained](https://www.youtube.com/watch?v=jEK07KPEjnY)
- [GitHub Action for GitHub Commit & Push](https://github.com/marketplace/actions/github-commit-push)
- [Checkout V3](https://github.com/marketplace/actions/checkout)

(fin)
