---
title: " [實作筆記] 設定 Vercel 的 DNS，並稍微記錄一下 heroku 停止免費服務後的可能方案"
date: 2022/12/18 20:42:20
---

## 前情提要

最近很紅的聊天 AI `ChatGPT` 開始出現各種應用了。  
這基本上應驗了我 2010 年說的 「10 年內，軟體工程師不會被取代」的說法  
雖然是否真的會取代工程師，我會另文寫寫我現在的看法，不過也算是敲響了一記警鐘。  
各式各樣的應用也隨之而來。

## Line Bot 與 chatGPT

最常見的應用就是 Line Bot 與 OpenAI GPT-3 API 進行串接。  
來達到一個賴機器人能夠與你有一定程度的智能對話。  
(不過實測看來 GPT-3 API 無法與 ChatGPT 的提供智能相比)

這類的應該短短幾周就出現了各式各樣的版本，  
語言有 nodejs、python、C#

主要的步驟大概如下，

1. 主程式 push 到 Github 上(大部份都很佛，可以直接 Clone)
2. 到 OPEN AI 取得 API KEY
3. 到 Line Developer 建立一個 Message API
4. 取得 Line Channel Access Token 、Channel Secret
5. 調整 Line 相關設定 ex: 停用自動回復
6. 到 Vercel 申請免費伺服器並連結 GitHub 帳號
7. Vercel 進行伺服器設定 webhook

即可完成

Vercel 這個服務也可以自定 domain，  
當然買了 marsen.me 這個 domain 的我自然不會放棄~~裝專業~~自定義 domain 的機會  
在 Vercal → 登入 → 找到你的專案 → Settings → Domains  
輸入你的網域 `subdomain.yourdoma.in`

接下來去你的 DNS 服務商的設定表(我是用 cloudflare)加上一個 CNAME 的記錄，
| Type | Name | Content |
| ---- | ---- | ---- |
| CNAME | gptbot | cname.vercel-dns.com|

## 後記

這是我第一次使用 Vercel，沒什麼門檻，簡單好上手。
不過看到各方大神使用了各種免費服務，
想說稍微記錄一下這些服務，以因應 heroku 停止免費服務，  
或許未來也會有有機會用到

### [uptimerobot](https://uptimerobot.com/)

免費服務通常一段時間沒有用就會睡著，這是個自動喚醒它們的服務

### 免費部署 service 的方案

- heroku(已停止免費方案)
- Colab + Flask(Google 提供給 python 的雲端 solution)
- [fly.io](https://fly.io/)
- [vercel](https://vercel.com/)
- [render](https://render.com/)

我還沒有用過這些服務，所以先作記錄，想要比較的話，可以使用 [slant](https://www.slant.co/)

## 參考

- [【Side Project】(全圖文教學) 用 Python flask 實作類似 ChatGPT 的 Linebot，並部屬至 Vercel 上 (updated: 2022/12/15)](https://www.wongwonggoods.com/portfolio/personal_project/gpt-linebot-python-flask-for-vercel/#comment-330)
- [OPEN AI x ChatGPT x LineBot 串接完整全紀錄](https://vocus.cc/article/639da520fd89780001e965d7) python 不採用，教學隨附影片的聲音好吵
- [ChatGPT LINE Bot](https://github.com/isdaviddong/chatGPTLineBot)　 C# 不採用，對機敏資料的處理方式我不喜歡
- [用 Node.js 建立你的第一個 LINE Bot 聊天機器人以 OpenAI GPT-3 為例](https://israynotarray.com/nodejs/20221210/1224824056/) Nodejs 未採用，就找到適合的專案了

(fin)
