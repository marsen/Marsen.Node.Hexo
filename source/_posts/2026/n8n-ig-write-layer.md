---
title: "[實作筆記] 個人自動化平台(五) n8n 實作：寫層，Instagram API 取得 Token 全記錄"
date: 2026/04/23 02:14:41
tags:
  - 實作筆記
---

## 前情提要

取層（RSS 抓資料）和讀層（Gemini 整理週報）都跑通了，接下來要實作寫層：把 AI 整理好的內容自動發布到社群媒體。

這篇記錄從「選哪個平台」到「拿到 Instagram Long-lived Token」的完整過程，包含踩到的坑和解法。

---

## 平台選擇

想把 AI 週報自動發布到社群媒體，先評估三個選項：

| 平台 | 純文字 | API | 帳號門檻 | 難度 |
|------|--------|-----|---------|------|
| Instagram | 不行（必須圖片） | 有 | Business 或 Creator | 高 |
| YouTube 影片 | 不行（必須影片） | 有 | 普通 Google 帳號 | 最高 |
| YouTube Community | 可以 | **無** | 500+ 訂閱 | — |
| LinkedIn | 可以 | 有（需審查） | 需要 Page | 中 |
| Telegram 公開頻道 | 可以 | 最簡單 | 無 | 低 |

最終決定：**Instagram**（目標平台），但 IG 必須有圖片，寫層要加圖片生成步驟。

---

## IG 帳號準備

Instagram API 發文需要 **Business 或 Creator 帳號**。個人帳號的 Basic Display API 已於 2024 年 12 月關閉。

切換 Creator 帳號（免費，不需要 Facebook Page）：

```text
IG App → 設定和隱私 → 帳號類型和工具 → 切換為專業帳號 → 創作者
```

切換後保留所有粉絲和貼文，只是解鎖 API 權限。

---

## Facebook Developer App 設定

### App 類型要選對

建 App 時類型有坑：

- **Consumer**：給舊版 Basic Display API 用，已關閉，選了找不到 Instagram Graph API
- **Business**：正確選項，支援 Instagram Graph API content publishing

前往 [developers.facebook.com/apps/create](https://developers.facebook.com/apps/create)，選 **Business**。

### Instagram API 流程選擇

進 App 後找 Instagram 產品，有兩種 API 流程：

| | Instagram Login | Facebook Login |
|--|--|--|
| 支援 Creator 帳號 | 是 | 需要連 FB Page |
| 需要 Facebook Page | 不需要 | 需要 |

選 **API setup with Instagram business login**（Instagram Login 流程）。

### 加 Tester 帳號

Step 1 要先在 Roles 頁籤加 Instagram Tester：

1. 左側選單 → App Roles → Instagram Testers
2. 輸入 IG username 送出邀請
3. 在畫面下方找到連結，點開到 `https://www.instagram.com/accounts/manage_access/` 接受邀請

---

## OAuth 黑/白畫面 bug

點 **Generate token** 後，Instagram 授權頁面出現黑畫面或白畫面，token 無法顯示。

這是 Instagram Developer Console 已知的 rendering bug。OAuth 授權其實已完成，authorization code 在 redirect URL 裡，只是 UI 沒渲染出來。

### 解法：webhook.site 當臨時 Redirect URI

**步驟一**：前往 [webhook.site](https://webhook.site)，取得唯一 URL，例如：

```text
https://webhook.site/98ec2192-c62e-459e-848a-334b370887f0
```

**步驟二**：加到 Instagram App → Step 3 → OAuth Redirect URLs。

**步驟三**：手動打開授權 URL（替換 `YOUR_WEBHOOK_URL`）：

```text
https://www.instagram.com/oauth/authorize?client_id=APP_ID&redirect_uri=YOUR_WEBHOOK_URL&response_type=code&scope=instagram_business_basic,instagram_business_content_publish
```

**步驟四**：Instagram 授權後 redirect 到 webhook.site，從 URL 取得 `code=` 後面的值。

**步驟五**：馬上換 short-lived token（code 幾分鐘過期）：

```bash
curl -X POST https://api.instagram.com/oauth/access_token \
  -F client_id=APP_ID \
  -F client_secret=APP_SECRET \
  -F grant_type=authorization_code \
  -F redirect_uri=YOUR_WEBHOOK_URL \
  -F "code=YOUR_CODE"
```

**步驟六**：換 long-lived token（有效 60 天）：

```bash
curl -X GET "https://graph.instagram.com/access_token?grant_type=ig_exchange_token&client_secret=APP_SECRET&access_token=SHORT_LIVED_TOKEN"
```

回傳範例：

```json
{
  "access_token": "IGAAYLng3...",
  "token_type": "bearer",
  "expires_in": 5165491
}
```

Token 更新（60 天到期前跑一次即可）：

```bash
curl -X GET "https://graph.instagram.com/refresh_access_token?grant_type=ig_refresh_token&access_token=LONG_LIVED_TOKEN"
```

---

## IG Token Generator 工具

OAuth 流程太繁瑣，自製工具放在 `demo.marsen.me/admin/tools/ig-token`，下次取 token 不需要手動操作。

**技術棧**：Next.js 16 App Router（Marsen.AI.Did 專案）

**流程設計**：

```text
使用者輸入 App ID + App Secret
  → POST /api/instagram/start
    → server 存 session cookie（5 分鐘 TTL）
    → 回傳 Instagram OAuth URL
  → 瀏覽器跳轉到 Instagram 授權
  → Instagram redirect 到 /api/instagram/callback
    → server 讀 cookie 取得 App Secret
    → 換 short-lived token → 換 long-lived token
    → redirect 到工具頁顯示結果
```

App Secret 全程只在 server-side 流通，不暴露給客戶端。

---

## Vercel 部署的 prebuilt 限制

這個專案用 GitHub Actions 做 CI/CD，採 `vercel build --prebuilt` 方案：

```yaml
# GitHub Actions
- run: vercel build --prod    # Actions 裡 build
- run: vercel deploy --prebuilt --prod  # 上傳 build 產出物
```

更新 Vercel 環境變數後，**不能直接 Redeploy**，因為 Vercel 沒有原始碼可重新 build。必須 push 新 commit 觸發 GitHub Actions 重新 build 才能套用新設定：

```bash
git commit --allow-empty -m "chore: trigger redeploy"
git push origin main
```

prebuilt 的好處是 CI 與 deploy 分離，確保 lint/type-check/test 通過才部署；代價是 env var 更新要重新 build。

---

## 參考

- [個人自動化平台(一) n8n & GCP VM](/2026/n8n-1-in-gcp-free-tier-vm/)
- [個人自動化平台(二) Cloudflare Access & Cloudflare Tunnel](/2026/n8n-2-cloudflared-and-tunnel/)
- [個人自動化平台(三) 收集、處理、輸出：三層可插拔管道設計](/2026/n8n-3-pipeline-overview/)
- [個人自動化平台(番外) 拆掉重建 GCP VM & Cloudflared Tunnel & Cloudflare Access](/2026/n8n-4-rebuild-infromation/)
- [個人自動化平台(四) n8n 實作：收集層，RSS 資料來源](/2026/n8n-5-pipeline-fetch/)
- [個人自動化平台(五) n8n 實作：處理層，Aggregate + Gemini 週報整理](/2026/n8n-6-pipeline-process/)

## 小結

- IG API 只支援 Business 或 Creator 帳號，個人帳號的 Basic Display API 已關閉
- Facebook Developer App 類型要選 Business，流程選 Instagram Login（不需要 FB Page）
- Developer Console 的 token generator 有 bug，用 webhook.site 當臨時 redirect URI 繞過
- Long-lived token 有效 60 天，到期前用 refresh API 更新即可
- 自製工具把 OAuth 流程自動化，之後取 token 一鍵搞定

（待補：圖片生成 + 實際 IG 發文流程）

---

## IG Token Generator 待改進

- **範例資料不用真實值**：截圖或 UI 展示時，input 欄位的 placeholder 不應出現真實 App ID，用假資料代替
- **Input 欄位短暫快取**：App ID 和 App Secret 用 `sessionStorage` 快取 15～30 分鐘，避免每次授權都要重填（注意 App Secret 快取有資安風險，要考慮清楚）
- **Token 長期快取 90 天**：取得 long-lived token 後存到 `localStorage`，90 天內重開頁面自動顯示，不需重新授權

(fin)
