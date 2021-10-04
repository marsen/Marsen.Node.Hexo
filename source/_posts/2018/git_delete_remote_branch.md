---
title: "[實作筆記] Git 批次刪除分支"
date: 2018/12/12 18:36:31
tag:
    - Git
    - 實作筆記
---

## 要知道的事

- OS : Windows 10

## 易學難精的 Git 與 Flow

在使用 Git 時，分支的策略往往比 Git 本身更複雜。  
Git 在建立分支是成本非常低的一件事情，  
也因此很容易開出一堆分支，  
這與團隊規模和PR的流程有關，可以參考文末的分支策略聯結。  
因為 Git 開分支實在太便宜了，  
我的 Remote Repo 不知不覺中竟然有了破千的分支。  
大量的分支意味著大量的需求，其實是好事，  
但是大部份的分支都已經功成身該退了，  
當我使用一些 GUI 工具，為了顯示這些分支時，  
這樣的數量反而成了阻礙。

## 解法

### 大量刪除遠端分支的方法

Step1. 可以透過正規表示式查詢大量分支。

```shell
 git branch -r | awk -Forigin/ '/\/feature\/BTS14/{print $2}'
```

Step2. 同上的語法，但是後面　pipeline 串接 xargs push 到指定的遠端(這個例子是 origin)

```shell
 git branch -r | awk -Forigin/ '/\/feature\/BTS14/{print $2}'| xargs -I {} git push origin :{}
```

特別看一下 `{} git push origin :{}` ，我們實際上是透過 push 語法刪除分支的。

## 補充

### 2019/05/09

### 大量刪除本地分支的方法

Step1. 可以透過正規表示式查詢大量分支。

```shell
git branch | grep "pattern"
```

Step2. 同上的語法，但是後面　pipeline 串接 xargs push 到指定的遠端(這個例子是 origin)

```shell
git branch | grep "pattern" | xargs git branch -D
```

## 參考

- [Git分支管理策略 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2012/07/git.html)
- [筆記：TBD是三小?---What is Trunk Based Development?](http://nedwu13.blogspot.com/2014/01/tbd-what-is-trunk-based-development.html)
  - 主幹開發主幹發佈/主幹開發分支發佈/分支開發主幹發佈

(fin)
