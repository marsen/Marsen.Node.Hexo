---
title: " [踩雷筆記] Gitlab CI/CD 與 GCP - 靜態網站部署整合 404 與 `/`"
date: 2024/05/14 11:42:53
tags:
  - 實作筆記
---

## 前情提要

最近將公司的所有靜態網站轉換到 GCS 上了，遇到一些情境，特此記錄  
這篇會以 Vue 與 Nuxt 的角度出發。
在前後端分離的情況下，SEO 變成一個問題，SSR 可以解決這個問題，  
我們可以使用 Nuxt 建立我們的專案，並透過 `generate` 生成靜態檔案。

再進一步來說，我們可以整合 [GCS 進行靜態網站的部署](https://cloud.google.com/storage/docs/hosting-static-website)

### 題外話

Nuxt 的幾種建置方式

1. `npx nuxt build`
  The build command creates a .output directory with all your application, server and dependencies ready for production.  
  這個命令會建立一個 .output 目錄，裡面包含了你的應用程式、伺服器和所有必要的依賴，準備好用於生產環境。  
  預設的行為，可以實作 SSR(Server Side Render)，適用有 SEO 需求，且變化快速的網站，Ex: 電商產品頁

2. `npx nuxt build --prerender`
  --prerender	false	Pre-render every route of your application. (note: This is an experimental flag. The behavior might be changed.) .  
  --prerender false 會對應用程式的每個路由進行預渲染。（注意：這是一個實驗性的標誌。行為可能會更改。）；  
  它會先產生好 HTML，適合 SSG 網站(內容不常改動，但有 SEO 需求）。Ex: BLOG

3. `npx nuxt generate `

  The generate command pre-renders every route of your application and stores the result in plain HTML files that you can deploy on any static hosting services. The command triggers the nuxi build command with the prerender argument set to true  
  這個命令會對你的應用程式的每個路由進行預渲染，並將結果存儲在普通的 HTML 文件中，你可以部署到任何靜態托管服務上。　　
  該命令會觸發 nuxi build 命令，並將 prerender 參數設置為 true。  
  它適合純靜態的網頁(SPA)。Ex: Landing Page.　與前者最大的差異是，前者會建置成不同的 HTML，

## 問題

當我們在建置好靜態網站並部署到 GCS 上時，我遇到了一個異常的問題，  
當我複製貼上網址時會發生 404 Not Found，主因是當我的網址結尾不是`/`時，  
瀏覽器會轉導到 `/index.html`。
舉例說明:

`https://marsen.me/sample` 複製貼上，會轉導到 `https://marsen.me/sample/index.html`
而這頁不存在，導致產生 404 的錯誤

## 解法

採取升級 Nuxt 到 [v3.8.0](https://github.com/nuxt/nuxt/releases/tag/v3.8.0) 以上，  
這是實驗性的功能，雖然目前有效，仍需觀注未來更新的狀況。  
若不打算升級，另外有用 `middleware` 處理重定向，  
可參考此 [issue](https://github.com/nuxt/nuxt/issues/15462#issuecomment-1407374859) 的討論串，  
要寫比較多，而且留言者在部署到 vercel 有遇到其他問題。故採升級的方式解決此題。

## 參考

- [靜態網站部署整合 GCP - Load Balancing & Cloud Storage](https://blog.marsen.me/2022/03/09/2022/gcp_static_site_with_cloud_storage_and_loading_balancing/)
- [實作筆記] Gitlab CI/CD 與 GCP 相關文章
  - [架構全貌](https://blog.marsen.me/2023/04/13/2023/gitlab_ci_and_gcp_vm/)
  - [建立 Web Server VM](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_create_server/)
  - [建立 Gitlab Runner VM](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_cretae_runner/)
  - [防火牆設定](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_firewall/)
  - [Linux User 與資料夾權限](https://blog.marsen.me/2023/04/24/2023/gitlab_ci_and_gcp_vm_account/)
  - [機敏資料的處理](https://blog.marsen.me/2023/05/29/2023/gitlab_ci_and_gcp_vm_secret_config/)
  - [錯誤處理](https://blog.marsen.me/2023/11/16/2023/gitlab_ci_error_handle/)
  - [Workload Identity Federation](https://blog.marsen.me/2024/03/13/2024/gitlab_ci_and_gcp_workload_federation/)
  - [Cloud Run](https://blog.marsen.me/2024/04/17/2024/gitlab_ci_and_gcp_cloud_run/)

(fin)
