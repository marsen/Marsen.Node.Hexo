---
title: "[實作筆記] Gitlab CI/CD 與 CGP - 建立 Web Server VM"
date: 2023/04/14 10:41:50
---

## 前言

請參考[前篇](https://blog.marsen.me/2023/04/13/2023/gitlab_ci_and_gcp_vm/)今天將來建立兩台 VM，  
一台作為 CI/CD 用的 Gitlab Runner，另一台作為 Web Server，  
架構如下
![GCP 與 Gitlab](/images/2023/gitlab-gcp.jpg)

## 建立 VM

建立　GCP VM 步驟如下：

1. 登入 GCP 控制台並選擇適合的專案。
2. 在主選單中選擇「Compute Engine」，進入虛擬機器的管理頁面。
3. 點選「建立」(Create) 按鈕，開始建立新的虛擬機器。
4. 選擇適合的地區 (Region) 與區域 (Zone)，這會決定虛擬機器的物理位置與可用性。
5. 選擇適合的機器類型 (Machine type) 與規模 (Instance size)，這會決定虛擬機器的運算能力與資源配置。
6. 選擇適合的作業系統映像 (Operating System Image)，這會作為虛擬機器的基本作業系統。基本上我都選 Ubantu
7. 選擇適合的開機磁碟 (Boot disk) 配置與容量，這會作為虛擬機器的主要儲存空間。
8. 選擇適合的防火牆設定 (Firewall) 與網路 (Network) 設定，這會決定虛擬機器的網路連線與安全性設定。
9. 完成其他選項設定，例如使用者資料 (User data)、預設密鑰 (SSH Keys)、啟用擴展 (Enable extensibility) 等。
10. 檢閱與確認虛擬機器的設定，並按下「建立」(Create) 按鈕，開始建立虛擬機器。
11. 等待 GCP 完成虛擬機器的建立與啟動。
12. 成功建立虛擬機器後，可以透過 SSH 連線或其他遠端連線方式進入虛擬機器，並進行相關的設定與應用程式部署。

## Web Server

Web Server 我們建置得很簡單，
使用 Vue 建立的靜態網站，並且由 Nginx 來 host 它

- 安裝 Nginx

```terminal
sudo apt update
sudo apt install nginx
```

設定 Nginx：預設的 Nginx 設定檔通常位於 /etc/nginx/nginx.conf.。  
可以編輯此設定檔，設定 Nginx 如何處理 Vue 的靜態檔案。  
以下是一個範例設定檔：

```conf
server {
    listen 80;
    server_name example.com;  # 替換為你的網域名稱

    root /www/example_output;  # Vue 建置的靜態檔在 .output/public 之中

    location / {
        try_files $uri $uri/ /index.html;  # 使用 Vue 的路由模式，將所有請求都導向 index.html
    }
}

server {
    listen 443 ssl;
    server_name example.com;
    root /www/example_output;
    index index.html;

    ssl_certificate /etc/letsencrypt/live/example.com/full_chain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/private_key.pem;

  location /www/example_output {
    index index.html;
  }
}
```

在建立測試環境的 Web Server 時，需設定 public IP 以對外提供服務，  
但同時需要建立防火牆以保障安全性。其他如 DNS、憑證、防火牆等設定，之後再補充。  

## 建立 Vue 專案

將 Vue 應用程式編譯成靜態檔案：在 Vue 應用程式的專案目錄中，可以使用以下命令編譯 Vue 應用程式成靜態檔案：

```terminal
npm run generate
```

編譯完成後，靜態檔案會生成在 ./output 目錄下。

將編譯完成的靜態檔案放置到 Nginx 的目錄,  
重新啟動 Nginx 即完成部署，如果有設定 Hot Reload 則不需要重新啟動  
現在可以透過瀏覽器來測試 Vue 的靜態網站是否正常運作。

### Tips

重啟 nginx

```terminal
systemctl restart nginx
```

Nginx hot reload

```terminal
nginx -s start
```

## 參考

- [GCP-Compute Engine](https://cloud.google.com/compute)
- [Certbot-get your site on Lock https://](https://certbot.eff.org/)
- [實作筆記] Gitlab CI/CD 與 CGP相關文章
  - [架構全貌](https://blog.marsen.me/2023/04/13/2023/gitlab_ci_and_gcp_vm/)
  - [建立 Web Server VM](https://blog.marsen.me/2023/04/14/2023/gitlab_ci_and_gcp_vm_create_vm/)
- [Vue.js](https://vuejs.org/)

(fin)
