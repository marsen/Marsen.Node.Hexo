---
title: '[KATA]用typescript作一個簡易的todolist(一) - 前置作業'
tag: 
- kata
- typescript
- bootstrap
- npm
- jquery
- jquery-ui
---
## 目標
1. 使用 typescript 開發
2. 顯示/新增/刪除 todolist 

## 功能分析
1. 只是練習,故不開發server side的程式
2. 暫時存在 cookie 上
3. 用 bootstrap 作簡單的樣式

## 實作記錄

### UI : 使用 Bootstrap _沒有必要重新打造輪子,能用的就拿來用_
- 找到一個TODOLIST的[樣版](http://bootsnipp.com/index.php/snippets/featured/todo-example),內含 HTML 、 CSS 與 JS ,功能完整.
- 取用[樣版](http://bootsnipp.com/index.php/snippets/featured/todo-example)的HTML .     
- 引用 bootstrap 3.3.5 CDN上的 css .
- 建立一個 todo.css 直接引用[樣版](http://bootsnipp.com/index.php/snippets/featured/todo-example)的 css 並加入頁面參考.


### 開發環境 
1. 安裝 typescript  
`npm install typescript --save`
2. 安裝 gulp  
`npm install gulp`
`npm install --global gulp` 
3. 安裝 gulp-typescript  
`npm install gulp-typescript`
4. 建立gulpfile.js 

        var gulp = require('gulp');
        var tsc = require('gulp-typescript');
        gulp.task('default', function() {     
            return gulp.src('public/javascripts/**/*.ts')
            .pipe(tsc())
            .pipe(gulp.dest('public/javascripts/'));
        });


### Typescript

1. 關於 typescript 的定義檔, 以前有 tsd 與 typings兩種管理工具,現在可以更簡便的合併到 npm 作管理 .

2. 透過 [TypeSearch](http://microsoft.github.io/TypeSearch/) 可以找到 [bootstrap](https://www.npmjs.com/package/@types/bootstrap) 的 typescript 定義檔.

3. 執行 `npm install --save @types/bootstrap` 安裝 bootstrap (目前的版本是 Bootstrap 3.3.5) , 因為相依於 jquery 所以也會一併安裝

4. 安裝 jquery-ui 的定義檔  
`npm install --save @types/jqueryui`

5. 新增檔案 `todo.ts` ,將 [樣版](http://bootsnipp.com/index.php/snippets/featured/todo-example) 的 javascript 複製貼上 .   
   _*註:因為 typescript 是 javascript 的 superset , 完全可以相容原生 javascript, 如果有任何錯後, TypeScript將會提示你_

6. todo.ts 引用 jquery 、jquery-ui 與 bootstrap 的 typescript 定義檔.  
        
        /// <reference path="../../../node_modules/@types/jquery/index.d.ts"/>  
        /// <reference path="../../../node_modules/@types/bootstrap/index.d.ts"/>  
        /// <reference path="../../../node_modules/@types/jqueryui/index.d.ts"/>

7. 頁面載入對應的js檔,記得放置順序 jquery 要在最前面,並將放在`<\body>`之後  
        
        <script src="http://code.jquery.com/jquery-2.2.4.min.js"   integrity="sha256-BbhdlvQf/xTY9gja0Dq3HiwQF8LaCRTXxZKRutelT44="   crossorigin="anonymous"></script>
        <script src="//maxcdn.bootstrapcdn.com/bootstrap/3.3.5/js/bootstrap.min.js"></script>
        <script src="http://code.jquery.com/ui/1.12.1/jquery-ui.min.js"   integrity="sha256-VazP97ZCwtekAsvgPBSUwPFKdrwD3unUfSGVYrahUqU="   crossorigin="anonymous"></script>

8. 執行 `gulp` ,會產生todo.js

7. 頁面載入js檔,記得要放在相依的js( jquery 、 bootstap 、 jqueryui)之後.
        
        <script type="text/javascript" src="/javascripts/kata/todo.js"></script>


#### 截至目前為止,僅僅是在作複製樣版,同時一併處理一些基本 gulp 與 typescript 相關的配置
#### 接下來才要開始寫 typescript 

(待續)

#### 參考

1. [關於 TypeScript 2.0 之後的模組定義檔 ( Declaration Files ) ( *.d.ts )](http://blog.miniasp.com/post/2016/08/22/TypeScript-Future-Declaration-Files.aspx)
2. [bootsnipp](http://bootsnipp.com/)
3. [TYPESCRIPT + EXPRESS + NODE.JS](http://com-brianflove.appspot.com/2016/03/29/typescript-express-node-js/)