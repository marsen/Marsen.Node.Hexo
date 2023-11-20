---
title: " [草稿] Hexo 7 升級的錯誤處理03"
date: 2023/11/20 01:31:32
---

## 速記

Hexo 升級 7 版後的問題

### hexo action 壞掉

錯誤訊息

```text
node:internal/modules/cjs/loader:1051
  throw err;
  ^

Error: Cannot find module 'hexo-util/lib/spawn'
Require stack:
```

### 地端執行 hexo d 時 Permission Denied

解決方式，直接用將　repository 改 ssh 為 https，不深究原因，  
畢竟有 CI/CD 後換了幾次電腦，都不在地端部署，所以沒有發現問題。
而地端權限改走 https 很久，原因是授權相對簡單

```yaml
deploy: 
  type: git
  repository: https://github.com/marsen/marsen.github.io.git
  branche: master
```

#### 後續

因為 https 的設定方式會提需要供出帳密，所以改回 git@github.com:marsen/marsen.github.io.git 的設定。
但是 hexo d 就會噴錯

```text
位於分支 master
沒有要提交的檔案，工作區為乾淨狀態
git@github.com: Permission denied (publickey).
致命錯誤: 無法讀取遠端版本庫。

請確認您有正確的存取權限並且版本庫存在。
FATAL Something's wrong. Maybe you can find the solution here: https://hexo.io/docs/troubleshooting.html
Error: Spawn failed
    at ChildProcess.<anonymous> (/Users/mark.lin/Repo/Marsen/Marsen.Node.Hexo/node_modules/hexo-deployer-git/node_modules/hexo-util/lib/spawn.js:51:21)
    at ChildProcess.emit (node:events:515:28)
    at ChildProcess._handle.onexit (node:internal/child_process:294:12)
```
處理方式
GitHub Copilot: 你可以透過在終端機中輸入 `ls ~/.ssh` 來檢查 `.ssh` 目錄下是否有 `config` 檔案。如果有 `config` 檔案，那麼你的 SSH 連線設定應該已經存在。如果沒有，你可能需要創建一個新的 `config` 檔案並設定你的 SSH 連線。

你可以使用以下的命令來創建和編輯 `config` 檔案：

```bash
touch ~/.ssh/config
nano ~/.ssh/config
```

在 `config` 檔案中，你可以添加以下的內容來設定你的 SSH 連線：

```text
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa
```

這裡的 `IdentityFile` 應該指向你的私鑰檔案。如果你的私鑰檔案不是 `~/.ssh/id_rsa`，請將這裡的路徑改為你的私鑰檔案的路徑。

完成以上的設定後，你應該就可以正常地使用 `git@github.com:marsen/marsen.github.io.git` 來部署你的 Hexo 網站了。

### youtube tag 無法處理

錯誤訊息

```error
unknown block tag: youtube {% youtube XXXX %}
```

原因 hexo 7.0 的破壞式更新中包含了 youtube tag 的移除  
(此外還有gist、jsfiddle、vimeo)
<https://hexo.io/news/2023/03/18/hexo-7-0-0-rc1-released/index.html>

修正方式
引入安裝 <https://github.com/hexojs/hexo-tag-embed> package

### Hexo 部署錯誤快取

升版後，地端、CI 都有異常，一時找不到錯誤，原來 hexo 會有留有快取，需要清除。
完整部署 SOP 如下

1. sudo rm -rf .deploy_git
2. hexo clean
3. hexo g
4. hexo d

## 參考

<https://rs11.xyz/articles/2.html>
<https://hexo.io/docs/tag-plugins.html#google_vignette>
<https://github.com/hexojs/hexo-tag-embed>

(待續)
