---
title: " [踩雷筆記] CORS 錯誤的元兇竟然是瀏覽器外掛？"
date: 2025/05/08 14:31:47
tags:
  - 踩雷筆記
---

## 前情提要

最近在本機開發一支簡單的上傳 API，使用 Express.js 搭配 TypeScript。  
預期只是在 local 上做個整合測試，前端跑在 `localhost:5173`，後端是 `localhost:3000`，照理來說只要 CORS 設好就萬事 OK。

結果——炸了。

## 問題現象

瀏覽器跳出熟悉的錯誤訊息：

```bash
Access to fetch at 'http://localhost:3000/upload' from origin 'http://localhost:5173' has been blocked by CORS policy...
```

一看就是 CORS 錯誤，老問題了，馬上開始排查：

- 確認 `cors()` middleware 有設好
- 檢查 headers 有無正確設定
- 懷疑是 OPTIONS preflight 沒處理好，也做了補強

怎麼改都一樣。

接著我試了無痕模式，欸？沒事了。  
懷疑是瀏覽器快取作怪，清除後還是一樣。  
換用 Edge（沒裝任何外掛）測試，正常。

## 真相大白

這時我就卡關了，因為一切設定看起來都沒問題。  

隨口問了新人同事，提到有可能是瀏覽器外掛的問題，我再測試一下  

確定是我在 Brave 安裝的外掛「Page Assist - A Web UI for Local AI Models」在搞鬼！  

這個外掛可能會偷偷注入 JS 或改寫 request header，導致 CORS 預檢請求出現異常，讓瀏覽器誤以為是 server 設定問題。  

我一停用外掛，再次測試，一切正常。  

## 心得與提醒

這次的經驗讓我再次體會：

> 最難 debug 的錯誤，往往來自你「以為不會出錯」的地方。

給未來的自己幾個建議：

- **開無痕模式測試**：能快速排除快取與外掛的干擾  
- **換乾淨的瀏覽器**：有時候 Edge 或 Safari 能救你一命  
- **早點求救**：卡住就問，別一個人浪費太多時間在錯的地方打轉，別人可以給你不同觀點

這類奇怪的問題其實很常見，但也正因為難以複製與預期，才更值得記錄下來。

下次遇到奇怪的 CORS 錯誤時，可以考慮是**裝的外掛在搞事。**

(fin)
