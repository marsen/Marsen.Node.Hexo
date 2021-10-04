---
title: "[學習筆記] Linux 語法學習筆記 二"
date: 2018/04/05 15:53:23
tag:
  - Unix
  - Linux
  - Shell
---


[上一篇](https://blog.marsen.me/2018/03/26/2018/linux_command_1/)我們學會一些基本 linux command,  
接下來我們將介紹更多的命令並組合它們到 shell script.  
讓 script 幫助我們完成一些事, 就像魔法一般, 開始囉.  

## 補充指令

### `vim`

#### 開啟 vim 編輯器

### `echo`

#### 印出文字

> $ echo "text"
text

#### 印出變數 `echo $*`

#### 印出PID (Process ID) `echo $$`

### `set`

#### 設定變數

>$ set good morning marsen

補充:使用 `echo` 印出變數,**從1開始**  
`$*` 指所有變數

>$ echo $1
>good
>$ echo $2
>morning
>$ echo $3
>marsen
>$ echo $*
>good morning marsen

#### 進階使用 backticks 執行 `cat` Command

>$ cat > testfile
hello world
sh-4.4$ set `cat testfile`
sh-4.4$ echo $*
hello world

## 範例

### Hello World

1. 建立檔案

  > $ cat > helloworld.sh

2. 編輯檔案

  > $ vim helloworld.sh

  ```shell
  #say hello
  echo "hello world"
  ```

3. 執行檔案

> $ sh helloworld.sh
> hello world

### 變數 variable

1. 大小寫有分
2. 使用 `read` 讀取 input 到變數中
3. 使用 `$`+變數名呼叫變數

sample:

```shell
# this is a shell sample
echo "who are you?"
read name
echo "Hi, $name nice to see you."
```

executed:

>$ sh whoareyou.sh
who are you?
Mark
Hi, Mark nice to see you.

### 互動式重新命名檔案

sample:

```shell
# this is a shell sample
echo "keyin a filename"
read name
mv $1 $name
echo $name"
```

>$ sh rename.sh file1
>keyin a filename
>newfile
>newfile

### 其它

1. 額外的 vim 問題排解 `E348: No string under cursor` 表示未輸入 i 進入 `Insert` mode
   - ESC + : , 輸入 w filename (以filename保存)
   - ESC + : , 輸入 wq (存儲並離開vim)
   - ESC + : , 輸入 q! (不存儲並離開vim)

2. 「**`**」 Backquote 或 backticks

## 參考

1. [Unix Terminal Online](https://www.tutorialspoint.com/unix_terminal_online.php)
2. [離開Vim的~~100種~~方法](https://itsfoss.com/how-to-exit-vim/)
3. [鳥哥的 Linux 私房菜---認識與學習BASH](http://linux.vbird.org/linux_basic/0320bash.php)

(fin)
