---
title: "[實作筆記] Git 批次刪除遠端分支"
date: 2018/12/12 18:36:31
tag:
    - Git
---

## 易學難精的 Git 與 Flow

在使用 Git 時，分支的策略往往比 Git 本身更複雜。  
Git 在建立分支是成本非常低的一件事情，  
也因此很容易開出一堆分支，  
這與團隊規模和PR的流程有關，但是因為 Git 開分支實在太便宜了，  
導致我的 Remote Repo 不知不覺中竟然有了破千的分支。  
中間換過專案、團隊，甚至是 Issue System。  
整個 Web 開發部門大約 50 個 RD，兩個 System 的編號都破3萬，  
這還不包含有部份的「黑需求」，扯這麼多只是想說明軟體開發的模糊性有多高;  
多容易迷失在需求大海之中。  

回到正題，大量的需求意味著大量的開發，同時也代表著大量的分支  
至少在我的 Remote 上是這樣，大概有破千的分支，
雖然不影響我作業，但是九成以上的分支都已功成身退了，
所以我想刪掉這些分支，但是一支一支刪就太慢了。

## 解法
找了一下方法，如下:
查詢：先看一下你要刪掉什麼分支，可以透過正規表示式查詢大量分支。
```shell
 git branch -r | awk -Forigin/ '/\/feature\/BTS14/{print $2}'
```
刪除：同上的語法，但是後面　pipeline 串接 xargs push 到指定的遠端(這個例子是 origin)
```
 git branch -r | awk -Forigin/ '/\/feature\/BTS14/{print $2}'| xargs -I {} git push origin :{}
```
特別看一下 `{} git push origin :{}` ，我們實際上是透過 push 語法刪除分支的。

(fin)