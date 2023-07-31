---
title: " [踩雷筆記] Gitlab CI 執行 Command 結果與本地環境不一致(草稿)"
date: 2023/07/31 17:16:56
tags:
  - 踩雷筆記
---

## 前情提要

在建立 CI 的一個目標是讓繁鎖的工作流程自動化，  
而自動化的前題是標準化，而自動化的未來是可以規模化。
每一次的自動化，就像是在為工作流程添加加速器

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

原來所謂的 shell，有分為 Interactive 與 Non-Interactive 兩種，
這兩種的取得的基本設定(Path)，不會一樣，導致即使用了相同的帳號登入了相同的機器，  
能夠運行的指令可能還是不一樣。

## 解法

```shell
- ssh -i ~/.ssh/id_rsa gitlab-runner@$VM 'export NVM_DIR="$HOME/.nvm" && [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" &&
      pm2 restart app.js
```

這裡其實有多種解法，而我選擇了重新 export NVM_DIR 這個作法。

另一種作法會多上一層 shell 我覺得較不易理解

```shell
- ssh -i ~/.ssh/id_rsa gitlab-runner@$VM 'bash -i -l -c "pm2 restart app.js"'
```

## 參考

- <https://www.gnu.org/software/bash/manual/html_node/What-is-an-Interactive-Shell_003f.html>
