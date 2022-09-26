---
title: "[實作筆記] Hexo CI/CD 設置"
date: 2022/09/26 15:35:20
tag:
  - CI/CD
  - Hexo
---

## 前情提要

我的 blog 一直以來都是用 markdown 加上 hexo 與 github page 建置的靜態網站。  
為此，我也寫了一系列的文章;但是我卻沒想過優化 blog 撰寫的流程。

我目前寫作的的流程是這樣的:

首先我會用任何工具補捉自已的想法，手機、筆記本、便利貼、Notion etc…  
然後我會用 [hackmd.io](https://hackmd.io/?nav=overview) 來寫草稿。  
草稿完成後，我會在自已電腦的 [Marsen.Node.Hexo Repo](ttps://github.com/marsen/Marsen.Node.Hexo) 上完成文章。  
接著我會先執行 git push 上版，再接著手動進行 blog 的建置(hexo g) 與部署(hexo d)。

這次要優化的部份是 **手動進行 blog 的建置(hexo g) 與部署(hexo d)**。
我希望寫完 blog 後推上 main 就是發佈了。

> 進一步的話，可以考慮用分支作草稿流程管理，但先不過早優化。

## 實作

首先，來了解我們的架構
![hexo CI/CD](https://i.imgur.com/yl4RqLG.png)

如圖，可以被畫分為兩個部份，第一個部份是紅框處，  
在取得 Hexo Source 後，執行 `hexo g` 會產生靜態檔案放在 `public` 資料夾，  
藍框處是第二部份，將 `public` 資料夾部署到 Github Page 的 repository 當中。

就這麼簡單，整個流程只會用到兩個工具 `git` 與 `hexo`，  
~~我們只要透過 docker 準備好安裝這兩個工具的 image 在 CI/CD 上執行即可~~
我找到了一個 Github Action 的 repository -- [HEXO-ACTION](https://github.com/sma11black/hexo-action)
依照以下的 SOP 就可以完成 CI/CD 設定

1. 準備 Deploy Keys 與 Secrets
   執行以下指令

   > $ ssh-keygen -t rsa -C "username@example.com"

   這時我們會取得一組私鑰與公鑰，
   請在 Github Page repository 的 Settings > Deploy keys 中，加入公鑰(記得要勾選 Allow write access)

   > <https://github.com/{your_account}/{your_account}.github.io/settings/keys>

   然後依照 hexo-action 的說明，你必需在 Hexo Source 的 repository 的 Settings > Secrets > actions 中加入私鑰
   有關 ssh-keygen 與不對稱加密的相關知識這裡就不多說明了,  
   簡單的去理解，這組鑰匙是用來對 Github Page Repository 作讀寫，所以公鑰會放在 Github Page Repository 中。
   想嚐試存取 Github Page Repository 的人都會拿到公鑰，但是真得要存取，你得有私鑰才行。
   這個時候，Hexo Source Repository 會將私鑰以 Secrets 的形式存下來，  
   在 Actions 執行時，會透過 `{secrets.DEPLOY_KEY}` 去取得私鑰，小心別弄丟了，不然你要從頭來過。  
   `DEPLOY_KEY` 是 HEXO-ACTION 所規範的 Key Name。

   > 如果你真的很想修改這個 Key Name，可以把 hexo-action fork 回去自已改

2. 配置 Github Workflow
   這步驟就相當簡單了，可以直接參考 HEXO-ACTION 的配置，或是看看我的 [workflow](https://github.com/marsen/Marsen.Node.Hexo/blob/main/.github/workflows/main.yml) 配置

## 後續

一些小錯誤要注意一下

第一點，Hexo Source repository 的 package-lock.json 應該要進版控，
這個檔案本來就應該進版控，不知道為什麼之前被設成 ignore 了，  
剛好 hexo-action 有用到 `npm ci` 語法才發現這個錯誤，相關文章請[參考](https://docs.npmjs.com/cli/v8/commands/npm-ci)

在 Hexo Source 的設定檔 `_config.yml` 中的 `deploy` 區段有兩點要注意，
第一，type 一定要是 `git`，第二，repository 的路徑請設定為 ssh (不要用 https)，
參考以下範例

```yml
deploy:
  type: git
  repository: git@github.com:marsen/marsen.github.io.git
  branche: master
```

如此一來，未來我寫好文章，只要 push 上 Hexo Source，就可以打完收工了，
再也不用額外的手動作業囉，這篇文章剛好就是第一篇，馬上來試試看。

## 心得

相關的技術與知識我都有兩年以上了，  
但是一直沒有發現應該將其結合起來，可以省下許多時間，
讓我想到 91 大說的綜效，我應該要讓想像力再開放一些，儘可能的把知識與資訊變成真正的價值。

## 參考

- <https://blog.marsen.me/tags/Hexo/>
- <https://www.ssh.com/academy/ssh/keygen>
- <https://github.com/sma11black/hexo-action>
- <https://israynotarray.com/nodejs/20211027/1827968017/>

(fin）
