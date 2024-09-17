---
title: " [學習筆記] Linux 語法學習筆記 一"
date: 2018/03/26 00:23:01
tags:
  - Unix
  - Linux
  - Shell
---

## 參考

- [Unix Terminal Online](https://www.tutorialspoint.com/unix_terminal_online.php)
- [Learn Shell Scripting all Levels](https://www.udemy.com/learn-shell-scripting-all-levels/)
- [umask 指令](http://linux.vbird.org/linux_basic/0320bash/csh/no3-8-01.html)

## 學習筆記

### `Clear`

#### 清除目前 terminal 畫面

### `Cal`

#### 產生當下的月曆

> $ cal

> March 2018  
> Su Mo Tu We Th Fr Sa

             1  2  3

4 5 6 7 8 9 10  
11 12 13 14 15 16 17  
18 19 20 21 22 23 24  
25 26 27 28 29 30 31

> $ cal 2 1985

> February 1985
> Su Mo Tu We Th Fr Sa

                1  2

3 4 5 6 7 8 9
10 11 12 13 14 15 16
17 18 19 20 21 22 23
24 25 26 27 28

### `Date`

#### 顯示日期與時間

> $ date

> Sat Mar 10 19:01:37 UTC 2018

> $ date  '+ %y-%m-%d %n %H:%M:%S:%N'

> 18-03-10
>  19:06:24:126172657

### `pwd`

#### 目前所在的檔案路徑

### `Touch`

#### 建立檔案

### `mkdir`

#### 建立資料夾

### `cat`

#### 寫檔案 `cat > filename`

> ctrl + d 可以離開編輯

#### 讀檔案 `cat < filename`

#### 合併檔案 `cat file1 file2 > merged_file`

> $ cat > file1
> this is file1
> $ cat file1
> this is file1
> $ cat > file2
> this is file2
> $ cat file2
> this is file2
> $ cat file1 file2 > merged_file
> $ cat merged_file
> this is file1
> this is file2

### `mv`

#### 重新命名檔案

> mv origin_name new_name

### `rm`

#### 刪除檔案或資料夾

> $ rm file_name

> $ rm -r folder_name/

### `rmdir`

#### 刪除資料夾

> $ rmdir folder_name/

### `cp`

#### 複製檔案

> $ cp oldfile other_folder/newfile

### `ln`

#### 聯結檔案(hard link)

> $ touch one  
> $ cat < one
> $ ln one two  
> $ ls
> one two
> $ cat > one  
> this is one  
> $ cat < one
> this is one
> $ cat < two  
> this is one

#### `ln -s` soft link

hard link 會產生實體檔案,soft link 只是指標的轉向.
如果使用 soft link,當刪除原始檔案時,link 檔案將無法開啟.

### 檔案權限概觀

#### 三種權限

- read / 讀 / 100 => 4
- write / 寫 / 010 => 2
- execute / 執行 / 001 => 1

每個權限都有一個代號,
read 表示可讀權限, 意味著可以開啟檔案與看見內容,
代號為 4,二進位表示為 100
write 表示可以複寫其內容,
代號為 2,二進位表示為 010,
execute 代表可執行,適用可執行檔或 shell script,
代號為 1,二進位表示為 001.
三種權限都有的話,權限為(111=>7)

#### 三種身份

- owner 開啟的帳號
- owner group 開啟的帳號所屬的群組
- other group 其它的群組

新增一個檔案的時候,
預設只有讀寫,沒有執行的權限 (100|010=110=>6)

> 指令 umask 的設定值以三個八進位的數字“nnn”代表。
> 第一個設定數字給使用者自己（owner user），
> 第二個則是設定給用使用者所屬的群體（group），
> 第三個給不屬於同群體的其它使用者（other）。
> 每一位數字的設定值都是三項不同權限的數值加總，
> read 權限數值為 4；write 權限數值為 2；execute 權限數值為 1。
> 結合了前三者的權限數值，單一的數字可設定的範圍是 0 ~ 7；
> 整體的可設定範圍是 000 ~ 777。
> --- 鳥哥的 Linux 私房菜

### `ls`

#### 列出資料夾中的所有檔案

#### `ls foldername`

列出指定的資料夾中所有的檔案

#### `ls -l`

列出資料夾中的所有檔案與其權限資訊

> ls 最常被使用到的功能還是那個 -l 的選項，為此，很多 distribution 在預設的情況中， 已經將 ll (L 的小寫) 設定成為 ls -l 的意思了！其實，那個功能是 [Bash shell](http://linux.vbird.org/linux_basic/0320bash.php) 的 [alias](http://linux.vbird.org/linux_basic/0320bash.php#alias) 功能呢
> --- 鳥哥的 Linux 私房菜

### `chmod`

#### 修改檔案權限

> sh-4.4$ ls -l
> total 4
> -rw-r--r-- 1 33581 33581 978 Mar 12 17:30 README.txt
> -rw-r--r-- 1 33581 33581 0 Mar 12 17:32 test
> sh-4.4$ chmod 777 test
> sh-4.4$ ls -l
> total 4
> -rw-r--r-- 1 33581 33581 978 Mar 12 17:30 README.txt
> -rwxrwxrwx 1 33581 33581 0 Mar 12 17:32 test
> sh-4.4$ chmod 444 test
> sh-4.4$ ls -l
> total 4
> -rw-r--r-- 1 33581 33581 978 Mar 12 17:30 README.txt
> -r--r--r-- 1 33581 33581 0 Mar 12 17:32 test

### `uname`

#### 顯示系統相關的資訊

> $ uname -a                                                            
> Linux e955582759de 3.10.0-514.26.2.el7.x86_64 #1 SMP Tue Jul 4 15:04:05 UTC 
> 2017 x86_64 x86_64 x86_64 GNU/Linux

> 選項與參數：
> -a ：所有系統相關的資訊，包括底下的資料都會被列出來；
> -s ：系統核心名稱
> -r ：核心的版本
> -m ：本系統的硬體名稱，例如 i686 或 x86_64 等；
> -p ：CPU 的類型，與 -m 類似，只是顯示的是 CPU 的類型！
> -i ：硬體的平台 (ix86)
> --- 鳥哥的 Linux 私房菜

### `file`

#### 查詢檔案基本資料(類型)

> file \*

> jazzy: ASCII text
> mark: empty
> marsen: directory

### `wc`

#### 顯示檔案資訊

行數 字數 字元數 檔名

> $ wc jazzy

> 3 10 39 jazzy

#### `wc -l filename`

顯示檔案行數資訊

#### `wc -w filename`

顯示檔案字數資訊

#### `wc -c filename`

顯示檔案字元數資訊

### `sort`

#### 印出排序過後的結果(遞增)

> $ sort
> owls
> pigs
> dogs
> cats

> cats
> dogs
> owls
> pigs

#### `sort filename`

印出檔案內排序過後的結果(遞增)

### `cut`

#### 切割資料

參數:
-d 分割字元
-f index (從 1 開始)

範例

> cat > filenames
> Name-Sport-Age
> Roger-Tennis-30
> Nadal-Tennis-25
> Tiger-Golf-37
> Michael-Baseball-49

> $ cut -d"-" -f 1,3 filenames
> Name-Age
> Roger-30
> Nadal-25
> Tiger-37
> Michael-49

### `dd`

#### 資料處理、拷貝、備份、轉碼;[更多](https://blog.gtwang.org/linux/dd-command-examples/)

> $ cat > infile
> this is the input file  
> $ cat infile
> this is the input file

> $ dd if=infile of=outfile conv=ucase
> 0+1 records in  
> 0+1 records out  
> 23 bytes copied, 6.6972e-05 s, 343 kB/s
> $ cat outfile
> THIS IS THE INPUT FILE

### `man`

#### 查詢其它指令用法

> $ man ls

```
LS(1)	User Commands  LS(1)
NAME	ls - list directory contents
SYNOPSIS	ls [OPTION]... [FILE]...

DESCRIPTION
       List  information  about the FILEs (the current directory by default).
	   Sort entries alphabetically if none of
       -cftuvSUX nor --sort is specified.

       Mandatory arguments to long options are mandatory for short options too.
       -a, --all
              do not ignore entries starting with .

       -A, --almost-all
              do not list implied . and ..

       --author
              with -l, print the author of each file

       -b, --escape
              print C-style escapes for nongraphic characters
 Manual page ls(1) line 1 (press h for help or q to quit)
```

按`h`看更多訊息

> $ man ls
> h

```
                   SUMMARY OF LESS COMMANDS

      Commands marked with * may be preceded by a number, N.
      Notes in parentheses indicate the behavior if N is given.

  h  H                 Display this help.
  q  :q  Q  :Q  ZZ     Exit.
 ---------------------------------------------------------------------------

                           MOVING

  e  ^E  j  ^N  CR  *  Forward  one line   (or N lines).
  y  ^Y  k  ^K  ^P  *  Backward one line   (or N lines).
  f  ^F  ^V  SPACE  *  Forward  one window (or N lines).
  b  ^B  ESC-v      *  Backward one window (or N lines).
  z                 *  Forward  one window (and set window to N).
  w                 *  Backward one window (and set window to N).
  ESC-SPACE         *  Forward  one window, but don't stop at end-of-file.
  d  ^D             *  Forward  one half-window (and set half-window to N).
  u  ^U             *  Backward one half-window (and set half-window to N).
  ESC-)  RightArrow *  Left  one half screen width (or N positions).
  ESC-(  LeftArrow  *  Right one half screen width (or N positions).
  F                    Forward forever; like "tail -f".
  r  ^R  ^L            Repaint screen.
HELP -- Press RETURN for more, or q when done
```

按`q`退出查詢畫面

### `banner`

#### 輸出用#組成的大形文字

實測未出現,上網查了一下 banner 好像有蠻多不同的類型可以安裝?

### `compress`

#### 壓縮檔案

### `zcat`

#### 讀取壓縮檔案

### `uncompress`

#### 解壓縮檔案

> compress 已經退流行了。為了支援 windows 常見的 zip，其實 Linux 也早就有 zip 指令了！ gzip 是由 [GNU 計畫](http://www.gnu.org/)所開發出來的壓縮指令，該指令已經取代了 compress 。
> --- 鳥哥的 Linux 私房菜

### 小結

以上是一些基本的 Linux Command ,  
下一篇,我們會建立.sh 檔,將 Linux Command 依照指定的順序執行  
並使用 `sh` 命令執行  
用以完成一些更進階的工作.

(more..)
