---
title: "[實作筆記] terminal 設定 alias"
date: 2021/07/09 10:02:43
tag:
    - 實作筆記
---

## 前情提要

在換了[新電腦](https://blog.marsen.me/2020/03/10/2020/macbook_ssh_add_and_git_fork/)後, 越來越常使用 terminal,  
在學習了 Vim 之後, 常常不開任何的 IDE 直接在 terminal 的編輯檔案,  
指令的使用頻率也更高了, 所以試著設定一下 `alias` 來提昇開發的效率.  
什麼是 `alias` ? 它可以幫助我們使用較短指令, 達到與完整指令一樣的效果,  
當我們常用的指令縮短了, 就可以加開我們開發的速度.  

## 設定檔案

`.bashrc` 或是 `.zshrc`, 我使用 iTerm 所以要在 `.zshrc` 裡設定.  
目前我最常使用的為 git,此外 hexo (一種靜態網站的發佈策略, 我用來設定我的 blog),  
設定如下:

### git

```shell
alias gac="git add . && git commit -m" # + commit message
alias gi="git init && gac 'Initial commit'"
alias gp="git push" # + remote & branch names
alias gl="git pull" # + remote & branch names
alias gs="git status"
## Pushing/pulling to origin remote
alias gpo="git push origin" # + branch name
alias glo="git pull origin" # + branch name
## Pushing/pulling to origin remote, master branch
alias gpom="git push origin master"
alias glom="git pull origin master"
## Branches
alias gb="git branch" # + branch name
alias gc="git checkout" # + branch name
alias gcb="git checkout -b" # + branch name
```

### Hexo

```config
alias hxg="hexo g"
alias hxs="hexo s"
alias hxd="hexo d"
```

## 參考

-[Git aliases for lazy developers](https://bitsofco.de/git-aliases-for-lazy-developers/)

(fin)
