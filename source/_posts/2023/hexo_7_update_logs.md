---
title: " [實作筆記] Hexo 7 升級的錯誤處理心得"
date: 2023/11/20 01:31:32
---

## 前情提要

最近在對Hexo進行升級至7.0版本的過程中，遇到了一些錯誤與問題。  
這篇部落格將記錄下這次升級的過程中所遇到的問題以及解決方法，希望對有需要的讀者有所幫助。

## 錯誤記錄

### Hexo Action 錯誤

升級後，遇到了Hexo Action報錯的問題，錯誤訊息如下：

```text
node:internal/modules/cjs/loader:1051
  throw err;
  ^

Error: Cannot find module 'hexo-util/lib/spawn'
Require stack:
```

解決方式是查找相應的模組，更新配置文件，在本案中，hexo-util/lib/spawn 路徑不存在，應該改為 hexo-util/dist/spawn。

### 地端執行 Hexo d 時 Permission Denied

在地端執行Hexo d時，遇到了權限問題，無法提交到GitHub的問題。  
暫時解決方式是將repository的協議由ssh改為https，暫時繞過了問題。

```yaml
deploy:
  type: git
  repository: https://github.com/marsen/marsen.github.io.git
  branch: master
```

後來因 https 需要提供帳號密碼，而這些資訊不適合簽入版本控制，故改回了 ssh，  
為了解決相應的 SSH 連線問題，需配置 SSH 連線 .ssh/config  
參考：

```text
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa
```

### YouTube Tag 無法處理

升級至Hexo 7.0後，發現內建的YouTube標籤無法使用，原因是Hexo 7.0中刪除了這些內建的標籤。  
解決方式是引入安裝 hexo-tag-embed package。  

### Hexo 部署錯誤快取

在部署過程中，遇到了Hexo的快取問題，導致地端和CI都出現異常。  
解決方式是清除 Hexo 的快取 `hexo clean`，完整 SOP 如下：

```terminal
sudo rm -rf .deploy_git
hexo clean 
hexo g
hexo d
```

### 使用 Docker 偵錯

[Hexo Action](https://github.com/marsen/hexo-action)本身是以容器去運行的，可以在本地端執行測試，不需推版 
使用 Docker 進行 Hexo Action 時，進行偵錯的方法：

建置與執行

```terminal
docker build -t hexo-action .
docker run -d hexo-action
```

觀察 logs 與 files 去偵錯，Docker Desktop 是很好的工具。

## 小結

這次Hexo 7.0的升級過程中，遇到了不少問題，但通過查找相應的解決方案，一一解決了這些問題。  
在升級過程中，需要留意Hexo的官方文檔以及相關插件的更新，以確保能夠順利完成升級。

## 參考

- <https://rs11.xyz/articles/2.html>
- <https://hexo.io/docs/tag-plugins.html#google_vignette>
- <https://github.com/hexojs/hexo-tag-embed>

(fin)
