---
title: " [記錄]串接 GOOGLE API 取得資料(二)"
date: 2017/07/20 01:04:14
tags:
  - Google
  - API
  - OAuth
---

## 前情提要

在[前篇](https://blog.marsen.me/2017/07/14/google_api_auth_1/)說明為什麼我要作「精神力評鑑」

原因是為了記錄自已的精力,能更有效率使用時間,進而提昇個人的生產力.

## 應用

有了數據後, 就要看怎麼運用.  
Google 的表單,可以自動產生回應結果試算表,  
持之以恒的每天記錄,很快就有上百千筆的資料.

我打算取得這些資料後,繪制成分佈圖
這樣就可以知道,我的黃金時間是在每一天的什麼時段.

### 如何透過 [GOOGLE Sheets API](https://developers.google.com/sheets/api/reference/rest/) 可以取得資料.

在[Google Cloud Platform](https://cloud.google.com/?hl=zh-tw),建立起 Google API 的服務。

1. 前往[Google API Wizard](https://console.developers.google.com/start/api?id=sheets.googleapis.com),建立或選擇專案。
2. 建立憑証,選擇 OAuth Client ID
   - 在這裡我會一次性的建立起所有環境(開發、測試、正式)的憑証 。
3. 下載 JSON 放置專案的指定位置.
4. 如何取得授權與取得資料，請參考[QuickStart](https://developers.google.com/sheets/api/quickstart/nodejs)
   - 安全考量,我不會將 client_secret.json 與取回的 token 加入版本控制
   - 以 Google 試算表為例，如何取得[spreadsheetId](https://developers.google.com/sheets/api/guides/concepts)? 很簡單，網址上就可以取得。
   ```
   ex:
   https://docs.google.com/spreadsheets/d/1qpyC0XzvTcKT6EISywvqESX3A0MwQoFDE8p-Bll4hps/edit#gid=0
   的spreadsheetId就是1qpyC0XzvTcKT6EISywvqESX3A0MwQoFDE8p-Bll4hps
   ```

### 說明

以 QuickStart 的程式為例 ,  
下載回來的檔案 `client_secret.json`  
可以提供 `clientSecret`、`clientId` 與授權後轉導的 url ,  
當程式執行時, 便會透 `googleAuth` 去取得授權 ,  
過程之中會需要使用者作驗証, 驗証完成即取得授權 ,
授權有一定的效期, 故一段時間之後需要重新取得授權

### 其它

1. `client_secret.json` 是機敏資料, 不可以放入版本控制, 需要特殊的流程步驟上傳到你的 Web Server 的位置 - Openshift 可以透過 SSH 或是 SFTP 登入來上傳`client_secret.json` - CI 以 JENKINS 為例 ; 可以使用 Publish over SSH 上傳檔案
   ![](https://i.imgur.com/Fo4Ml5M.jpg) - 需要注意 CI Server 要有 Web Server 的 SSH Key - LINUX 複製資料夾語法 `cp -rf src/folder/. target/folder`

2. 在正式公開的環境上可能會發生`Error: invalid_scope`的錯誤 ，可以[參考](https://support.google.com/code/contact/oauth_app_verification?id=705847791246&client=705847791246-l20jeqj1ncv2vffaki70ing4c8cda2r1.apps.googleusercontent.com&query=https://www.googleapis.com/auth/spreadsheets.readonly)。

   - 填寫表單作驗証
   - 個人測試可以加入[Risky Access Permissions By Unreviewed Apps](https://groups.google.com/forum/#!forum/risky-access-by-unreviewed-apps) 論壇

3. 我最後是使用[d3.js](https://d3js.org/)作為繪圖的 library , 因為網路上的資源相當的多 , 而且這次的著墨並不多 , 所以不在這裡介紹.

## Demo

![](https://i.imgur.com/8FYzhdg.gif)

## 參考

1. [Openshift SFPT](https://blog.openshift.com/using-filezilla-and-sftp-on-windows-with-openshift/)
2. [Linux CP](https://www.phpini.com/linux/cp-force-copy)

(fin)
