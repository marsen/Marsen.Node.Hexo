---
title: " [實作筆記] MacBook Terminal 美化與設計"
date: 2024/03/29 16:48:10
tags:
  - 實作筆記
---

## 2024更新

重寫此文，部份資訊仍有用途，故舊文不刪

## 前情提要

參考[前文](https://blog.marsen.me/2021/07/01/2021/mac_terminal_prompts_setting/)，受同事啟發，改用 zim 取代 oh-my-zsh，
用更高效與精煉的方式設定 terminal 環境，
看看下面的小故事與 [Zim 的官網](https://zimfw.sh/#install)，
這就我選擇更換的原因。

### 小故事

可以[參考此文](https://www.reddit.com/r/linuxadmin/comments/rhg7wx/zsh_frameworks/?rdt=39568)，  

當前流行的 Zsh 框架有三個主要選項：Oh My Zsh、Prezto 和 Zim。  
Oh My Zsh 是最受歡迎的，但因其過於龐大、混亂和性能問題而受到批評。  
Prezto 是對其的重大改進，但未被合併回 Oh My Zsh。  
而 Zim 是一個從頭重寫的框架，非常快速、高效，並將許多最佳想法結合在一起。  
Zim 還提供了各種主題和模塊，並且易於安裝和管理。  
這些框架在定製 Zsh 提示符、增加功能和優化性能方面提供了不同的選擇。

## Overview

- 下載與設定 iTerm2
- 安裝 zim
- 安裝 powerlevel10k

## 第一步 下載並安裝 [iTerm2](https://iterm2.com/downloads.html)

### 設定 iTerm2 的外觀

1. 從網站 iTerm2-Color-Schemes (<https://github.com/mbadolato/iTerm2-Color-Schemes>) 下載你喜歡的配色方案，例如 "DimmedMonokai.itermcolors"。
2. 開啟 iTerm2，點擊菜單欄的 "iTerm2"，選擇 "Preferences"。
3. 在偏好設定視窗中，選擇 "Profiles" 選項卡，然後選擇你要更改配色方案的會話配置文件。
4. 在 "Colors" 選項卡下，點擊 "Color Presets" 按鈕，選擇 "Import..."。
5. 找到剛才下載的配色方案檔案，例如 "DimmedMonokai.itermcolors"，點擊 "Open"。
6. 選擇 "DimmedMonokai" 作為你的 iTerm2 配色方案。

![import .itemcolors](https://i.imgur.com/d9qHicD.png)

Profiles 裡有更多的設定, 字型、顏色  
比如說, 調整啟始視窗大小與背景透明度, 可以前往 Windows 進行設定.
更多的細部設定可以自行摸索. **記得有些效果需要手動重啟 iTerm**

## 第二步, 安裝 [zim](https://zimfw.sh)

```sh
 curl -fsSL https://raw.githubusercontent.com/zimfw/install/master/install.zsh | zsh
```

即可完成安裝，並預設定一些相當實用的模組，可以[參考](https://zimfw.sh/docs/modules/)

### 設定

一般來說不需要這些額外的設定，我的情況是同時需要移除 `oh-my-zsh` 才會有額外的項目需要進行
進入 ~/.zshrc　修改

一、刪除 `oh-my-zsh` 的區塊  
常用的一些工具，都被整合在 `zim` 之中了
例如　zsh-syntax-highlighting　原本在 zshrc 的設定如下
現在都可以全數刪除

```shell
# Install zsh-syntax-highlighting if it's not installed
if [ ! -d ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting ]; then
  git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
fi

# Load zsh-syntax-highlighting plugin
plugins+=(zsh-syntax-highlighting)

# Set name of the theme to load --- if set to "random", it will
# load a random theme each time oh-my-zsh is loaded, in which case,
# to know which specific one was loaded, run: echo $RANDOM_THEME
# See https://github.com/ohmyzsh/ohmyzsh/wiki/Themes
ZSH_THEME="powerlevel10k/powerlevel10k"
```

由於`zim`本身就有包含 zsh-syntax-highlighting，  
所以其它相關的設定不會有問題。  
例如：

```shell
ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE='fg=177'
ZSH_HIGHLIGHT_HIGHLIGHTERS=(main brackets)
typeset -A ZSH_HIGHLIGHT_STYLES
ZSH_HIGHLIGHT_STYLES[comment]='fg=242'
```

### Prompt 設定

Prompt 主要影響 terminal 的外觀，  
一般來說社群主流會推薦的 Powerlevel10k 後面會提供實作步驟，  
但是 zim 已經提供足夠的 [theme](https://zimfw.sh/docs/themes/)作選擇，  
下面選用 minimal (zim 提供的輕量版主題)作範例 　
我們可以修改 `~/.zimrc` 即可完成設定  

```shell
# A heavily reduced, ASCII-only version of the Spaceship and Starship prompts.
zmodule minimal 
```

### 安裝 Powerlevel10k

不過 zim 一樣可以使用 Powerlevel10k
前往 [Powerlevel10k](https://github.com/romkatv/powerlevel10k)
我們使用 zim 的方式安裝

修改 `~/.zimrc`

```shell
zmodule romkatv/powerlevel10k --use degit
```

後執行　`zimfw install`

之後重啟 iTerm2 將會有一連串設定問題, 依照喜歡的設定即可,
可以參考這個[影片](https://www.youtube.com/watch?v=JnJm4gRrWN8&t=326s),  
如果設定完後不喜歡, 可以執行 `p10k configure` 重新設定

## 參考

- <https://blog.marsen.me/2021/07/01/2021/mac_terminal_prompts_setting/>
- <https://www.reddit.com/r/linuxadmin/comments/rhg7wx/zsh_frameworks/?rdt=39568>
- [How to make a beautiful terminal](https://dev.to/techschoolguru/how-to-make-a-beautiful-terminal-j11)
- [Powerlevel10k](https://github.com/romkatv/powerlevel10k)

(fin)
