---
title: 如何讓 Google Analytics 追踪你的 Hexo Blog
date: 2016/08/25 14:12:07
tag:
  - Hexo
  - Google
---

## 前置作業

1. 你要有 google 帳號，並申請好你的 google_analytics ID
2. 這個記錄僅針對 Hexo 預設的 Theme 有作用，未來有改 Theme 的話，可能會需要手動加入，再寫文章補上

## 開啟\_config.yml

1. 確定一下你是使用預設的 theme `landscape` ## [Themes](https://hexo.io/themes/)
   theme: landscape
2. 開啟 `root/themes/landscape/_config.yml`
3. 找到以下的設定區段 # Miscellaneous
   google_analytics:
4. 填入步驟 1. 中所取得 google_analytics ID
5. 部署網站，完成!

(fin)
