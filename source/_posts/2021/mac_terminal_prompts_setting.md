---
title: "[實作筆記] Macbook Terminal 美化與設計"
date: 2021/07/01 23:10:02
tag:
    - 實作筆記
---

## 前情提要

3 年前寫過[如何讓 windows 也有美美命令提示視窗](https://blog.marsen.me/2018/11/25/2018/colorfull_cmder/),  
不過在去年我開始使用 Macbook Pro 了, 當然也想要美美的 Terminal 啦.  
但是這個想法一直在放心裡面沒有實踐, 畢竟只是一個浮誇的東西.  
不過在防疫期間重新看了一遍[高見龍的即將失傳的古老技藝](https://www.youtube.com/playlist?list=PLBd8JGCAcUAH56L2CYF7SmWJYKwHQYUDI),  
除了把 Vim 再熟悉一遍外, 同時也觸動了心中浮誇的那塊.  
實作比 Windows 簡單很多, 在這裡稍作記錄.  

## Overview

- 下載 iTerm2
- 設定 iTerm2
- 安裝 oh-my-zsh
- 安裝 powerlevel10]

## 第一步 下載並安裝 [iTerm2](https://[term2.com/)

### 設定 iTerm2 的外觀  

1. 可以在這裡尋找[iTerm2-Color-Schemes](https://github.com/mbadolato/iTerm2-Color-Schemes)
2. Create the .itemcolors files
3. Peferences > Profiles > Colors > Choose Presets > import

![import .itemcolors](https://i.imgur.com/d9qHicD.png)

Profiles 裡有更多的設定, 字型、顏色  
比如說, 調整啟始視窗大小與背景透明度, 可以前往 Windows 進行設定.
更多的細部設定可以自行摸索.  **記得有些效果需要手動重啟 iTerm**

## 第二步, 安裝 [oh-my-zsh](https://ohmyz.sh/)

```sh
 sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

安裝相當簡單, 更多資訊可以參考 [Github](https://github.com/ohmyzsh/ohmyzsh)
這裡要了解如何對 `~/.zshrc` 進行編輯
## Power1[]

## 參考

- [How to make a beautiful terminal](https://dev.to/techschoolguru/how-to-make-a-beautiful-terminal-j11)

https://github.com/romkatv/powerlevel10k

(fin)
