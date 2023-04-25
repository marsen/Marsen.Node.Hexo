---
title: " [實作筆記] 自定義 Hexo Helper"
date: 2022/05/10 10:17:05
tag:
  - 實作筆記
  - Hexo
---

## 前情提要

我的 Blog 目前是使用 Hexo 搭建的,
使用的主題是 `landscape` 。  
隨著時光推移, Hexo 漸漸從人們的目光淡出,  
主功能、文件、翻譯或是外掛、主題等…, 更新速度變慢很多

我想改寫主題當中的某個輔助函數（Helper), 稍微記錄一下作法。

## 實作

首先是檔案位置，一個是在根目錄底下建立 `scripts` 資料夾，  
或者你正在使用主題(Theme)底下。  
以我的例子來說，我會放在 `themes/landscape/scripts` 底下

參考下面的圖示

```terminal
├── scripts
│   ├── index.js
│   └── your_helper_here.js
├── themes
│   └── the_theme_you_are_using
│       └── scripts
│           ├── index.js
│           └── your_helper_here.js
```

下一步，你需要建立 `index.js`,並在其入註冊你的函數,

```javascript
//hexo.extend.helper.register(name, function)
hexo.extend.helper.register("helper_name", function () {
  return "<div>your helper function </div>";
});
```

通常 helper 不會是一個簡單的函數，且會有 UI 相關的程式，  
有為了方便管理，我會拆檔，再用 `require` 來引入函數

```javascript
hexo.extend.helper.register("helper_name", require("./helper_file_name"));
```

最後，在 \*.ejs 檔案中使用 helper

```javascript
<%- helper_name() %>
```

## 參考

- [Hexo 輔助函數](https://hexo.bootcss.com/api/helper.html)
- [詳解 Hexo 輔助函數](https://blog.haysc.tech/custom-helper-hexo/)
- [將 Hexo 的 Markdown 渲染引擎換成 markdown-it](https://titangene.github.io/article/hexo-markdown-it.html)

(fin)
