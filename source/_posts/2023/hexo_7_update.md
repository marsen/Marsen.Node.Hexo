---
title: " [草稿] Hexo 7 升級的錯誤處理"
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

(待續)
