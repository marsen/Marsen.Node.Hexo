---
title: "將 Express 網站整合 TypeScript 開發"
date: 2017/08/16 16:26:14
tags:
  #- TypeScript
  - GulpJs
  - Nodejs
---

## 原因與目的

- 目前我使用 express 作為網站開發
- 我是個 .NET 工程師,習慣用強型別語言作開發 (C#)
- 網路上找的文章
  - [TypeScript 2 + Express + Node.js](http://brianflove.com/2016/11/08/typescript-2-express-node/)
  - [TypeScript + Express + Node.js](http://brianflove.com/2016/03/29/typescript-express-node-js/)
- 這件事情理論上是可行的,而且又有人作過,又能讓我以自已比較習慣的方式作開發

## 技術問題

- grunt/gulp/webpack
  - 選擇 gulp 希望能切換到 webpack
- TypeScript

## 初期目標

- 把所有 js 改成 ts
- 相同指令即可完成編譯與開啟站台
- 可部署到正式環境

## 構想

~~原本我是想把整個專案重新編譯至另外一個資料夾中,~~  
~~再由該資料夾設為起始專案執行~~
在建立 typescript 資料夾，只編譯相關的 ts 檔;
至於哪些是**相關的 ts 檔**?

1. `app.ts`(編譯為`app.js`)
   在根目錄作為所有 request 入口的起始檔案,用來解析 request 需要對應的 router
   這裡可能會有一些共用的商務/系統邏輯或是錯誤處理
2. 所有的 router
   這是最接近使用者的 business logic code,影響返回的頁面與呈現的資料,
   通常這裡主要的目的是組合來自不同的 service 的資訊,再返回給 view 層,
   不過有時候也會處理一些顯示邏輯.
3. 所有的 service
   這層擁最主要的商務邏輯,大多會依功能性作區分(ex:授權、會員、購物車等...)
   也有專門的 service 提供共用的方法及模組,
   並透過 repository 取得/更新資料

4. 所有的 repository
   這層最主要的功能是直接與資料庫作存取
5. 其它
   例如:Interface, Class, Enum 或是一些框架所需要額外的方法.

## 實作

定義好需要修改的範圍後,我建立了一個 typescript 資料夾
裡面會建立相對應的`router`,`service`與`repository`資料夾
與一個`app.ts`檔案.

一開始將所有 js 檔案依照相對的位置,照抄複製放入對應位置,並將副檔名改為`.ts`
接下我將利用 gulp 幫執行相關的編譯行為.
我們可以預期編譯產生的`.js`檔可以執行,因為 Typescript 是 Javascript 的 Super Set

### gulp

- 安裝 gulp
  `npm install gulp -g`
  `npm install gulp --save`
  `npm install gulp-typescript --save`

- 安裝相關模組
  `npm install @types/node --save-dev`
  `npm install --save @types/express`
  `npm install --save @types/morgan`
  `npm install --save @types/cookie-parser`
  `npm install --save @types/httperr`
  實際安裝哪些模組與專案所需要的有關,通常只要在 npm 搜尋`npm @types module_name`
  就可以找得到,不過有時候也會有找不到情況
  這個時候很苦惱了,我們想要 ts 的強型別即時除錯,但是自已刻又太自虐,
  我的想法是只有自已寫的`router`,`service`與`repository`有需要即時除錯.
  之後實務上有遇到再回來補充.

- 設定 gulp file

```bash
var gulp = require('gulp');
var tsc = require('gulp-typescript');

gulp.task('app', function() {
        return gulp.src(['typescript/app.ts','typescript/**/*.ts'])
        .pipe(tsc({
                target: "es2017",                          /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', or 'ESNEXT'. */
                module: "es2015",                     /* Specify module code generation: 'commonjs', 'amd', 'system', 'umd', 'es2015', or 'ESNext'. */
                lib: ["es2015","dom"],
                noImplicitAny: false,
            }))
        .pipe(gulp.dest('./'));
});
```

我們不需要`tsconfig.json`這個檔案,直接可以寫成 json 物件
另外記得設定 src(ts 所在的位置)與 dest(輸出 ts 的位置)

- 設定 package.json

```json
  "scripts": {
    "start": "node ./bin/www",
    "dev": "supervisor ./bin/www",
    "test": "node_modules/.bin/nightwatch",
    "e2e": "npm-run-all --parallel start test",
    "run": "node ./bin/www",
    "ts": "gulp app"
  }
```

- 執行 `npm run ts`
- 檢查輸出的 js,再說一次因為 Typescript 是 Javascript 的 Super Set
  所以可以預期會產生相同的檔案內容(副檔名變成.js)
- 運行網站確定功能正常

### 修改 ts 檔

雖然 js 檔已正常產生,但是其實這一切都是假的!
原因是我們的 ts 檔其實仍然在寫 js
![修改ts檔](https://i.imgur.com/5aCuXSy.jpg)
在執行 gulp 的過程當中,應該可以看到一些提示訊息.
由於有各種情況,就不一一說明了.
我在這此改版的疑難就附在下面,如果有遇到相同的問題,可以作為參考.
如果有不同問題,也可以留言討論喔(雖然我覺得是自行 google 會快一些) XD

### 疑難

- [typescript getting error TS2304: cannot find name 'require'](https://stackoverflow.com/questions/31173738/typescript-getting-error-ts2304-cannot-find-name-require)
- [Express and Typescript - Error.stack and Error.status properties do not exist](https://stackoverflow.com/questions/28793098/express-and-typescript-error-stack-and-error-status-properties-do-not-exist)
- [Error TS2693: 'Promise' only refers to a type, but is being used as a value here.](https://github.com/Microsoft/vscode/issues/21968)

## 參考

1. [npm scripts 使用指南](http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html)
2. [typescript getting error TS2304: cannot find name ' require'
   ](https://stackoverflow.com/questions/31173738/typescript-getting-error-ts2304-cannot-find-name-require)
3. [Express and Typescript - Error.stack and Error.status properties do not exist](https://stackoverflow.com/questions/28793098/express-and-typescript-error-stack-and-error-status-properties-do-not-exist)
4. [gulp-typescript](https://github.com/ivogabe/gulp-typescript)
5. [Promise static method give error](https://github.com/Microsoft/vscode/issues/21968)

(fin)
