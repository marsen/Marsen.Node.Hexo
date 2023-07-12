---
title: "[異常記錄] 多語系架構上線"
date: 2018/08/28 14:27:21
tags:
  - AWS
---

## 前情提要

很久很久很久以前，
【跨國語系基礎架構】上線了.  
又過了很久很久 **我們評估這多語系 API 需要上 CDN**  
![我們評估這多語系 API 需要上 CDN](https://i.imgur.com/0g5HL2z.jpg)

### Release 項目與工作內容

- 8/16 要 release 什麼?  
   我們要將多語系開始開發的項目一次回台灣正式環境。
- 我們作了什麼
  - sp23/sp24/sp25/sp26/sp27
  - 前端多語系底層架構的調整(含 CDN)
  - 國別設定
  - 還有一大堆 Merge 解衝突的項目...
- 為什麼要等這麼久才上線 ?  
   風險考量，我們先上了基礎架構，  
   在 G3 環境上觀察了一段時間確定沒有問題才上線。
- 講到風險，我們有多少保險 ?
  - 只上基礎架構 (讓異動最小)
  - 只上 G3 環境 (讓 impact 最小)
  - 藍綠部署 (標配，讓壞掉的時間儘可能縮短)
  - Merge Before Build (讓原始碼不受影響，rollback 時不會拔掉其它團隊的功能)

**講那麼多，還不是壞掉了。**

## 錯誤原因

### 錯誤畫面

![錯誤畫面](https://i.imgur.com/Etwq8nY.jpg)

### 錯誤原因

![錯誤原因](https://i.imgur.com/Ujy0Z4v.jpg)

多語系 CDN 的 Response Header Access-Control-Allow-Origin Protocol 與主頁面不符;

### 流程面

| 測試環境     | 測試結果      |
| ------------ | ------------- |
| QA8          | OK            |
| QA           | OK            |
| PP           | OK            |
| 鎖 Host 測試 | OK            |
| 半數機器上線 | OK            |
| 全數機器上線 | OK，好 (壞了) |

為什麼全數機器上線才發生異常。

- QA8 沒有 CDN
- 鎖 Host/半數機器上線未開 CDN
- QA / PP CND 設定與正式環境不一致
  - RD 並未 Check 設定的一致性
  - RD 沒有權限

### 技術細節

1. 使用 Partial View 與 OutputCache
2. 小心 Browser Cache
3. 站台上會有跨 Domain 的存取行為
   - Service to Official
   - Service to CDN
   - Official to Service
4. CDN & Access-Control-Allow-Origin
5. 稍微提一下 CORSModule

### 錯誤分析

![錯誤分析](https://i.imgur.com/GmBULAQ.jpg)

    TranslationCDN : 用來防止瞬間量對 Server 的 Impact
    FingerprintTag : 一份Cache的時戳， 用以取得新版多語系 Json 檔
    site : BrowserCache 遇到跨 Domain 存取時， 會發生 CORS Error

1. Partial View 會有 OutputCache
   → 保得了一時，保不了一世
2. 為了 Browser Cache ，F2E 在 QueryString 作了加工。
3. 網站跨 Domain 的存取行為，歷史共業。
4. CDN & Access-Control-Allow-Origin
5. CorsModule，有必要區分 protocol 嗎 ?

![架構圖](https://i.imgur.com/Dw2yIvG.jpg)

    ```csharp=93
    if (httpContext.Request.Url.Scheme == Uri.UriSchemeHttps && (sourceUri == null || sourceUri.Scheme == Uri.UriSchemeHttps))
    {
        allowOrigin = allowOrigin.Replace("http:", "https:");
    }
    ```

### 問題處理經過

1. 第一時間 QA 有看到異常， 但是在 PO 的環境無法重現， RD 初步判斷為快取的問題， 並嘗試重現錯誤 。
2. SRG 收到通報有**客戶反應**顯示異常，立即啟動藍綠部署，**耗時 93 分鐘**。
3. 縮小範圍至 CDN 與 Access-Control-Allow-Origin 異常，但是仍然**無法完整重現錯誤**(時好時壞)
4. **申請 CDN 權限**,確認各環境的設定值。
5. **申請 QAn 測試用 CDN**,在 QA8 環境反覆測試，  
   發現**頁面 OutputCache 會影響測試**
   關閉 QA8 OutputCache
   測試步驟如下:

   - Case1. http 強制更新後，連線 https

     - <http://8.qa.demo/index?r=t>
       - ts = 2387
       - ao = <http://8.qa.demo/index>
     - <http://8.qa.demo/index>
       - ts = 2387
       - ao = <http://8.qa.demo/index>
     - <https://8.qa.demo/index> => **Error**
       - ts = 2387
       - ao = <http://8.qa.demo/index>

   - Case2. https 強制更新後，連線 http
     - <https://8.qa.demo/index?r=t>
       - ts = 8458
       - ao = <https://8.qa.demo/index>
     - <http://8.qa.demo/index>
       - ts = 8458
       - ao = <https://8.qa.demo/index>
     - <https://8.qa.demo/index> => **Error**
       - ts = 8458
       - ao = <https://8.qa.demo/index>

6. 確認 Prod CDN 的 WhiteList Header 的設定有問題，
   那為什麼 PP/G3 不會有問題呢 ?
   => **在 PP/G3 設定為 Referrer**
7. 測試過程發現 x-cache Header 常常
   `Miss from cloudfront`,  
   表示資料並非來自 CDN .
   實際測試 Origin 比 Referrer 更適合

### 回饋與後續

    1. 團隊評估欠缺數據 →
    2. 客戶反應異常。
    3. 藍綠部署異常回覆速度，耗時 93 分鐘 → `已有RD著手改善`
    4. 無法完整重現錯誤
    5. RD 沒有 CDN 權限 → `已申請`
    6. QAn 沒有設定 CDN → `沒有CDN的Issue沒必要特別設定(因為需要對外)`
    7. 頁面 OutputCache 會影響測試。
    8. PP / G3 與 Prod  CDN設定不一致 → `webapi/translate/* 已調整一致`
    9. Release 三步驟 ， 最後一步才會開啟 CDN

Q. 下次我們怎麼作 ?

### 補充資料

- 什麼是 CDN ?
  ![什麼是 CDN](https://raw.githubusercontent.com/hungys/azure-blog/master/media/14-using-azure-cdn/cdn-concept.png)

      Server 跟 CDN 就像工廠與便利店的關係， 你要買東西不用跑到工廠
      只要在離自已最近的便利店買就好了， 過期的商品(資料)會被丟掉
      重新跟工廠進貨(拉新資料)。

- 什麼是 [CORS](https://developer.mozilla.org/zh-TW/docs/Web/HTTP/CORS) ?
- 什麼是 [Origin](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Origin)
- 什麼是 [Referrer](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer)
- [Slide](https://hackmd.io/p/SyFiZ2wIX#/)

(fin)
