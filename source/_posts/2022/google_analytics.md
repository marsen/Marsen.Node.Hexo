---
title: "[實作筆記] Hexo 更新 Google Analytics 4"
date: 2022/04/26 20:12:53
tag:
  - 實作筆記
---

## 前情提要

幾天前收到來自 Google Analytics 的這樣一封信

```text
Google Analytics (分析) 4 即將取代通用 Analytics (分析)

我們即將以新一代的成效評估解決方案 Google Analytics (分析) 4，
取代通用 Analytics (分析)。從 2023 年 7 月 1 日起，通用 Analytics (分析) 資源將停止處理新的命中資料；
如果通用 Analytics (分析) 仍是您常用的工具，建議您採行相應措施，準備好改用 Google Analytics (分析) 4
```

## 實作步驟

參考以前我寫的這篇--[如何讓 Google Analytics 追踪你的 Hexo Blog](https://blog.marsen.me/2016/08/25/2016/add_google_analytics_to_hexo_blog_1/)，
我們知道在 hexo 的佈景主題 `landscape` 有提供舊版 GA 的設定，
我們只需要修改 `theme/layout/_partial/google-analytics.ejs` 即可
程式參考如下

```javascript
<% if (theme.google_analytics){ %>
<!-- Google Analytics -->
<!-- Global site tag (gtag.js) - Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=<%= theme.google_analytics %>"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', '<%= theme.google_analytics %>');
</script>
<!-- End Google Analytics -->
<% } %>
```

由於通用 GA 在 2023 年後就不再支援，所以我就不特地處理相容，直接切換到 GA4

## 參考

- [Hexo Theme Landscape](https://github.com/hexojs/hexo-theme-landscape)
- [如何讓 Google Analytics 追踪你的 Hexo Blog](https://blog.marsen.me/2016/08/25/2016/add_google_analytics_to_hexo_blog_1/)

(fin)
