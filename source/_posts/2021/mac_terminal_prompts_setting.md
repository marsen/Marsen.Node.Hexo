---
title: " [實作筆記] MacBook Terminal 美化與設計"
date: 2021/07/01 23:10:02
tag:
  - 實作筆記
---

## 前情提要

3 年前寫過[如何讓 windows 也有美美命令提示視窗](https://blog.marsen.me/2018/11/25/2018/colorfull_cmder/),  
不過在去年我開始使用 MacBook Pro 了, 當然也想要美美的 Terminal 啦.  
但是這個想法一直在放心裡面沒有實踐, 畢竟只是一個~~浮誇~~的東西.  
不過在防疫期間重新看了一遍 [高見龍的即將失傳的古老技藝](https://www.youtube.com/playlist?list=PLBd8JGCAcUAH56L2CYF7SmWJYKwHQYUDI) 影片,  
除了把 Vim 再熟悉一遍外, 同時也觸動了心中浮誇的那塊.  
實作上比 Windows 簡單很多, 在這裡稍作記錄.

## Overview

- 下載 iTerm2
- 設定 iTerm2
- 安裝 oh-my-zsh
- 安裝 powerlevel10k

## 第一步 下載並安裝 [iTerm2](https://[term2.com/)

### 設定 iTerm2 的外觀

1. 可以在這裡尋找[iTerm2-Color-Schemes](https://github.com/mbadolato/iTerm2-Color-Schemes)
2. Create the `.itemcolors` files
3. Preferences > Profiles > Colors > Choose Presets > import

![import .itemcolors](https://i.imgur.com/d9qHicD.png)

Profiles 裡有更多的設定, 字型、顏色  
比如說, 調整啟始視窗大小與背景透明度, 可以前往 Windows 進行設定.
更多的細部設定可以自行摸索. **記得有些效果需要手動重啟 iTerm**

## 第二步, 安裝 [oh-my-zsh](https://ohmyz.sh/)

```sh
 sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

安裝相當簡單, 更多資訊可以參考 [Github](https://github.com/ohmyzsh/ohmyzsh)
這裡要了解如何對 `~/.zshrc` 進行編輯, 在後面安裝我們會需要編輯這個檔案,  
可以確認一下檔案內容

```sh
vim ~/.zshrc
```

### 安裝 Powerlevel10k

前往 [Powerlevel10k](https://github.com/romkatv/powerlevel10k)
我們使用 oh-my-zsh 的方式安裝

```sh
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

安裝後編輯 `~/.zshrc` 設定參如下:

`ZSH_THEME="powerlevel10k/powerlevel10k"`

之後重啟 iTerm2 將會有一連串設定問題, 依照喜歡的設定即可,
可以參考這個[影片](https://www.youtube.com/watch?v=JnJm4gRrWN8&t=326s),  
如果設定完後不喜歡, 可以執行 `p10k configure` 重新設定

```sh
    This is Powerlevel10k configuration wizard. It will ask you a few questions and
                                 configure your prompt.

                    Does this look like a diamond (rotated square)?
                      reference: https://graphemica.com/%E2%97%86

                                     --->    <---

(y)  Yes.

(n)  No.

(q)  Quit and do nothing.

Choice [ynq]:
```

## 其它

這裡記錄一下我會使用的設定

### `~/.zshrc`

```rc
 ZSH_THEME="powerlevel10k/powerlevel10k" # 設定使用 powerlevel10k 主題
```

安裝完 oh-my-zsh  後如果有資料夾問題, 會出現以下訊息

```sh
[oh-my-zsh] For safety, we will not load completions from these directories until
[oh-my-zsh] you fix their permissions and ownership and restart zsh.
[oh-my-zsh] See the above list for directories with group or other writability.

[oh-my-zsh] To fix your permissions you can do so by disabling
[oh-my-zsh] the write permission of "group" and "others" and making sure that the
[oh-my-zsh] owner of these directories is either root or your current user.
[oh-my-zsh] The following command may help:
[oh-my-zsh]     compaudit | xargs chmod g-w,o-w

[oh-my-zsh] If the above didn't help or you want to skip the verification of
[oh-my-zsh] insecure directories you can set the variable ZSH_DISABLE_COMPFIX to
[oh-my-zsh] "true" before oh-my-zsh is sourced in your zshrc file.
```

一般來說可以執行 `compaudit | xargs chmod g-w,o-w` 指令再重啟 iTerm2 即可以排除此問題.  
如果無法排除, 又覺得每次出現這個訊息很煩人的話, 請至 `~/.zshrc` 設定以下資訊

```rc
ZSH_DISABLE_COMPFIX="true"
```

### 活用 Terminal 的左右空間

在一般的終端機上, 隨著深入資料夾結構之中,  
左側顯示路徑是會越來越長, 可以透過設定讓其顯示縮短, 更加的簡潔
編輯 `~/.p10k.zsh` 以下參數

- POWERLEVEL9K_SHORTEN_DIR_LENGTH = 1
- POWERLEVEL9K_DIR_MAX_LENGTH = 0

而右側的留白空間, 反而很適合放入一些有用的參數
編輯 `~/.p10k.zsh` 以下參數

- POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS (右側的顯示內容)

未來如果有要調整相關的設定, 找`.p10k.zsh`與`.zshrc`的相關設定就對啦

附上成果圖
![my iTerm2](https://i.imgur.com/E4lSCit.png)

### 20211114 補充

但是在一些工具預設會用原本的 Terminal (終端機)開啟,  
比如說 [git-fork](https://git-fork.com/), 就會開啟原始的終端機,  
如果沒有更換字型, 就會看到一些不正常顯示的`?`符號, 可以在  
終端機 > 偏好設定 > 描述檔 > 文字 > 字體 作更改  
我使用的字體是: _MesloLGS NF_  
另外，git-fork 可以在

- Preference > Integration > Terminal Client 調整開啟的終端機為 iTerm2

### 20211114 補充 2

我已經將 .vim 資料夾入版控,  
未來只要在

1. 在 ~ 目錄下 Clone [.vim Repo](https://github.com/marsen/.vim) 即可
2. 在 ~ 目錄下設定連結

   > ln ./.vim/.vimrc .

### 20211114 補充 3

讓命令呈現 highlight 語法,  
使用 [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md)
這個設定可以帶來的好處是, 視覺上可以第一時間知道你有沒有打錯命令

#### Oh-my-zsh 的安裝方法

Clone this repository in oh-my-zsh's plugins directory:

```shell
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

編輯 `~/.zshrc` 開啟外掛

> plugins=( [plugins...] zsh-syntax-highlighting)

example:

> plugins=( git zsh-syntax-highlighting)

最後重啟終端機即可

### 20211116 補充

如何讓錯誤訊息 Highlight,

iTerm > Help > Trigger 或是  
iTerm > Preferences > Advanced > Trigger > Edit

以下是我的設定:

> (?i:._error._) // Yellow on Red  
> (?i:._(warning|warn)._) // Yellow  
> (?i:._FATAL._) // Red

## 參考

- [How to make a beautiful terminal](https://dev.to/techschoolguru/how-to-make-a-beautiful-terminal-j11)
- [Oh my zsh](https://github.com/ohmyzsh/ohmyzsh)
- [Powerlevel10k](https://github.com/romkatv/powerlevel10k)
- [DAY 22 使用 Vim 外掛](https://www.youtube.com/watch?v=aOfeDgu0SQA)
- [DAY 23 好用的 Vim 外掛介紹](https://www.youtube.com/watch?v=xSHOf6cFcrk)

(fin)
