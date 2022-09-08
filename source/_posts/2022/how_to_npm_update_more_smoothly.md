---
title: " 如何更無痛更新專案中的 npm 相依套綿"
date: 2022/09/08 14:11:15
tag:
  - Nodejs
---

## 前情提要

純屬個人偏見，**我討厭 Nodejs。**  
有幾個原因，  
首先，使用 javascript 作為開發語言，用弱型別直譯式的語言來開發大型專案是痛苦的。  
不論在 TDD、重構或是除錯上…我始終作不到[極速開發](https://www.facebook.com/91agile/posts/pfbid022QhYJSeWzH4sHq1mBW91Q5LjxNB7iD4oHS6Eks3jtPLdVhLFHkW63CXxua1aEuFhl)那樣流暢的感覺，  
很希望有機會可以跟這樣的高手交流。  
即使改用 TypeScript，儘管看到 [TypeScript 另人驚豔的開發方式](https://www.youtube.com/watch?v=p6dO9u0M7MQ)，這樣的感覺還是揮之不去。

再者，node_modules 是出了名的零碎肥大，網路上甚至為它作了迷音，如下圖  
![node_modules](https://i.imgur.com/Zkhmx8m.jpg)

第三點，專案的 npm 更新是相當麻煩的，也是這篇文章我們要面對的問題，  
通常我們的專案，隨著時間開發，會與越來越多 npm 套件，先不論它會不會變成黑洞影響時空，
我最常遇到的問題是，**我不知道有哪些套件需要被更新 !??**
這些更新有的無關痛養，有的是提昇效能，有的是安全性更新…
不論如何，我的目標是更新更加的流暢，而不要被一些奇怪的問題阻檔。

但實務上，更新之路是異常充滿危機與苦難…

## 實際案例

在這裡我用一個我的學習用專案來進行演練 [Marsen.React.Practice](https://github.com/marsen/Marsen.React.Practice)，  
這個專案是我用來練習 React 相關的開發，相比任何商務專案，相信會小得多。  
但是它相依的套件依然多得驚人，有的與字型相關，有的與 TypeScript 型別相關，
有多語系、有 firebase、有 RTK(React-Tool-Kit)、有 Router、有 Form etc...
隨隨便便也有近 30 幾個相依套件需要管理、更新。

比較好的或是有名的套件，或許會官宣一些更新的原因與細節，  
但是實際上，更多套件的更新你是不會知道的。

你可以這樣作

```terminal
> npm update
```

然後 BOOM !! 就爆炸了

![npm update](https://i.imgur.com/2qJrtOW.png)

你可以看 log、可以解讀這些錯誤訊息、可以梳理其背後的原因，然後用你寶貴的生命去修復它，  
過了幾周或是幾天，新版本的更新又釋出… 那樣的苦痛又要再經歷一次又一次…
或許你會想那就不要更新好了，直到某一天一個致命的漏洞被發佈或是資安審查時強制要求你更新，  
或是因為沒有更新，而用不了新的酷功能，這真是兩難，所有的語言都有類似的問題，但 Nodejs 是我覺得最難處理、最令人髮指的一個。

## 原來有方法

原來有不痛的方法，參考如下
首先執行 `npm outdated`
![npm outdated](https://i.imgur.com/wnPLs20.png)

如上圖，我們可以發現在套件後面會有 Current/Wanted/Latest 三種版號  
分別顯示:目前專案的套件版本/semver 建議的最新版本/Registry 中標注為 latest 的版本  
如圖所示，有時候你 Current 版本會比 Latest 版本還新，不用太過揪結與驚訝，  
更多可以參考[官方文件](https://docs.npmjs.com/cli/v8/commands/npm-outdated)的說明

接下來我們試著來解決這些套件相依，  
我們全域安裝 ncu(npm-check-updates) 這個工具

```terminal
> npm install -g npm-check-updates
```

執行更新

```terminal
> ncu -u
```

魔術般完成了更新，沒有任何痛點。
再次執行 `npm outdated` 將不會看到任何顯示，表示你目前專案的相依套件都在最新的穩定版本
定期執行或是透過 CI/CD 周期性協助你更新並 commit，是不是就會省下了許多時間呢 ?

## 參考

- [How to Update NPM Dependencies](https://www.freecodecamp.org/news/how-to-update-npm-dependencies/)
- [npm-outdated](https://docs.npmjs.com/cli/v8/commands/npm-outdated)
- [91 談極速開發](https://www.facebook.com/91agile/posts/pfbid022QhYJSeWzH4sHq1mBW91Q5LjxNB7iD4oHS6Eks3jtPLdVhLFHkW63CXxua1aEuFhl)
- [Node.js 終結者？青出於藍的 Deno(二)](https://tecky.io/en/blog/Node.js%E7%B5%82%E7%B5%90%E8%80%85-%E9%9D%92%E5%87%BA%E6%96%BC%E8%97%8D%E7%9A%84Deno%28%E4%BA%8C%29/)

(fin)
