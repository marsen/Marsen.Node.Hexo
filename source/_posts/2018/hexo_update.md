---
title: "[實作筆記]Hexo Update 與 npm 修復漏洞"
date: 2018/10/20 11:41:05
tag:
  - Hexo
  - nodejs
---

好久沒有作 hexo 更新了，  
本身這個 Blog 就是透過 [hexo 與 Github 建置](https://blog.marsen.me/2016/08/28/how_to_use_github_page/)的 。
眨眼也過了兩年，文筆仍然很差;  
說真的有很大的[資訊焦慮](https://www.darencademy.com/article/view/id/16485)，文章的內容也不夠充實，  
很多觀點無法忠實的記錄下來，更無法連貫成線;  
自已還要多方努力與學習。

如前面所說，這個 Blog 是透過 Github 作存放的空間，  
雖然 [Github 最近被 Microsoft 收購](https://news.microsoft.com/2018/06/04/microsoft-to-acquire-github-for-7-5-billion/)了;  
不過它仍是開發人員最常使用的代碼托管服務與社群，  
而且微軟近年轉型相同成功 –— 未來會怎麼樣我們可以觀察後再下定論。  

Github 一直以來都有一個很棒的功能，  
[GitHub's security alerts for vulnerable dependencies](https://help.github.com/articles/about-security-alerts-for-vulnerable-dependencies/)

它能夠對你的 Repo 提供安全警告，  
如下圖，這是對我的個人 Blog 所使用的框架 Hexo 的相依性安全警告，  
這封信說明了 lodash 這個套件的版本存在著高風險的資安問題，   
建議升級套件以解決這個問題。  
![](https://i.imgur.com/HYBh5vv.jpg)


### 解決方式

使用 [`npm update`](https://docs.npmjs.com/cli/update) , 這個命令會更新專案的 package 到最新版本  


```sh
npm update
```

可能會看到類似以下的結果，同時提示你可以使用 [`npm audit fix`](https://docs.npmjs.com/cli/audit) 更新有風險的 package 以修複這些漏洞  


```sh
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.4 (node_modules\nunjucks\node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.4: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.1.3 (node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.1.3: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})

+ hexo@3.8.0
added 126 packages from 59 contributors, removed 80 packages, updated 21 packages, moved 14 packages and audited 3143 packages in 25.064s
found 68 vulnerabilities (21 low, 38 moderate, 9 high)
run `npm audit fix` to fix them, or `npm audit` for details
```

(fin)