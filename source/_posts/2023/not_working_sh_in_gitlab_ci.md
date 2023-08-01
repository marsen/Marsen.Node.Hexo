---
title: " [踩雷筆記] Gitlab CI 執行 Command 結果與本地環境不一致"
date: 2023/07/31 17:16:56
tags:
  - 踩雷筆記
---

## 前情提要

我們建立 CI 的一個目標是讓繁鎖的工作流程自動化，  
而自動化的前題是標準化，而自動化的未來是可以規模化。  
每一次的自動化，就像是在為工作流程添加柴火，讓未來的的每一步可以走的更穩更快。

## 問題

一直以來我認為只要在本機環境上可以執行的指令(command),就一定可以在 CI 中執行，
沒想到這次不一樣。

下面這個語法是我 CI 流程的一部份，主要的目的是要重啟 `pm2` 的服務

```shell
pm2 restart app.js
```

這段在 Gitlab-runner CI 的寫法如下，主要的目的就在 Gitlab-runner 中將指令送到指定的機器`$VM`上

```shell
- ssh -i ~/.ssh/id_rsa gitlab-runner@$VM 'pm2 restart app.j'
```

不過我會收到錯誤訊息

```shell
bash: pm2: command not found
```

當我直接在`$VM`執行時，確又不會有錯誤

## 根本原因

原來在 Unix-like 系統中，shell 分為 Interactive Shell（互動式）和 Non-Interactive Shell（非互動式）。  
兩者環境變數和 PATH 不同，導致在 CI 或遠程機器上執行命令可能與本地不一致

查詢了一下兩者的 PATH 如下

Interactive Shell

> /home/gitlab-runner/.nvm/versions/node/v20.4.0/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin

Non-Interactive Shell

> /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin

明顯可見的差異在於`NVM`的路徑設定,這個 PATH 是在安裝 `NVM` 時被加上去的

## 解法

我們可以選擇重新 export NVM_DIR 這個作法

```shell
- ssh -i ~/.ssh/id_rsa gitlab-runner@$VM 'export NVM_DIR="$HOME/.nvm" && [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" &&
      pm2 restart app.js
```

另一種作法會多上一層 shell，並再次執行互動式的 bash 我覺得較不易理解  
不好維護，故不採用。

```shell
- ssh -i ~/.ssh/id_rsa gitlab-runner@$VM 'bash -i -l -c "pm2 restart app.js"'
```

## 參考

- [What is an Interactive Shell?](https://www.gnu.org/software/bash/manual/html_node/What-is-an-Interactive-Shell_003f.html)

(fin)
