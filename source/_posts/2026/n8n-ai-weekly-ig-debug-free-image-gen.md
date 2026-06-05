---
title: "[實作筆記] n8n AI 週報踩坑記：一條接錯的線，與找免費生圖來源的旅程"
date: 2026/06/05 12:02:20
tags:
  - 實作筆記
---

> ⚠️ 這篇是開發記錄檔（草稿），隨進度補充，最後再整理成正式文章。圖片來源那段還沒完工。

## 前情提要

我有個 n8n workflow，每週自動抓 AI 新聞 → Gemini 生週報 → 發一篇 IG，順便生一張配圖、也產一篇 Blog 週報開 PR。

跑了一陣子，兩個毛病：

1. **每次發兩篇 IG**，一篇正常、一篇是空的。
2. **配圖每週都是動物**，跟當週主題八竿子打不著。

本來以為是兩個獨立問題，結果查下去——**根因是同一條接錯的線**。這篇記錄整個排查過程，還有後面為了「免費生圖」繞的一大圈。

---

## 流程長怎樣

```text
Schedule → RSS×4 → Filter → Edit Fields → Aggregate
  ├─ 文字生成(Gemini)      → IG 準備發文 → Wait → IG 發佈
  ├─ 生成圖片 Prompt(Gemini) → IG 準備發文
  └─ Blog 生成(Gemini)     → GitHub 建 branch → commit → 開 PR
```

配圖是把文字餵給 `image.pollinations.ai/prompt/{{...}}` 生的。

---

## 問題一：一次發兩篇 IG

### 根因

讀 workflow JSON 畫連線，發現 `IG 準備發文` 這個節點的**同一個輸入埠（input 0）被兩條線餵**：`文字生成` 和 `生成圖片 Prompt` 都接進去。

n8n（executionOrder v1）在「同一輸入埠收到多條來源連線」時，**每條來源各觸發一次**該節點 → `IG 準備發文` 跑兩次 → 發兩篇。這是 n8n 經典陷阱：該用 Merge 合併分支的地方，直接把兩條線併到同一個 input。

順便還抓到一個 bug：IG 那個 httpRequest 的 body 參數名是 `" image_url"`，**開頭多一個空格**，IG 收到的 key 變成 ` image_url` 被忽略 → 圖片沒帶進去 → 空貼文。

### 修法

把「並聯」改成「串聯」：

```text
Aggregate → 文字生成 → 生成圖片 Prompt → IG 準備發文
```

- `IG 準備發文` 只剩一條 input → 跑一次 → 發一篇
- 順手把 `" image_url"` 的空格拿掉

---

## 問題二：圖片每週都動物

### 根因（其實還是那條線）

Pollinations 要的是「視覺場景描述」（英文、具體畫面），餵新聞摘要長文它抓不到重點 → 退化成安全牌，而動物是文字生圖最常見的 fallback。

但更前面的根因是：workflow 裡**其實已經有**一個「生成圖片 Prompt」的 Gemini 節點，prompt 也寫得不錯（挑最有畫面感的一則、用英文描述場景）。問題是它**接錯位置**——從 `Aggregate` 進來，而 Aggregate 輸出的是 `{ data: [...] }`，**沒有 `.text`**。節點模板寫 `{{ $json.text }}` → 拿到 undefined → 等於餵空內容 → 圖當然爛。

所以**同一個接線錯誤**：既讓 `生成圖片 Prompt` 拿不到週報內容，又跟 `文字生成` 併線造成重複發文。

### 修法

問題一的串接修法，順便就解了問題二——`生成圖片 Prompt` 接在 `文字生成` 之後，`$json.text` 真的拿得到週報全文了。再把它的 prompt 強化（固定畫風關鍵字 `digital illustration, vibrant colors, no text, no logo`、排除人名品牌、加 SFW 防護）。

> 教訓：兩個看起來無關的 bug，根因可能是同一個。先畫連線圖再動手。

---

## 插曲：Blog 分支同天重跑撞名（422）

修完去重跑驗證，Blog 那條噴 `422 Reference already exists`。

根因：branch 名固定用 `weekly/{{yyyyMMdd}}`，同一天手動重跑就撞到自己上次建的 branch。正常每週排程日期不同不會撞，但測試時會。

修法：

- branch 名改成 `weekly/{{yyyyMMdd-HHmmss}}`，同天重跑不撞
- 後面 commit / 開 PR 的節點，branch 改成**讀「建 branch」實際回傳的 ref**（`.ref.replace('refs/heads/','')`），保證三個節點用同一個名字，不會各自用 `$now` 重算而漂移

---

## 大冒險：到底哪裡能免費生圖？

IG 兩個原始 bug 修好上線了，但配圖來源出事——這段繞最久。

關鍵限制：**IG Graph API 是用「它自己的伺服器」去抓 `image_url`**。所以不管我這邊怎麼生圖，只要那個 URL 被擋，IG 就抓不到。

| 來源 | 結果 |
|---|---|
| Pollinations 匿名 | **402** Payment Required。它改政策了：匿名並發限 1，超過就回 JSON 錯誤（還夾帶 x402 加密貨幣付款，要付 USDC 繞過）|
| Pollinations 註冊 token | 舊的 `auth.pollinations.ai` 網域**已下線**（DNS 查無），現行入口是付費 Console |
| Gemini 圖片（我現有的 key）| **429，free tier limit: 0**。文字免費、圖片要開帳單付費層 |
| Replicate flux | 沒有正式免費額度，試用後**要綁信用卡**（約 $0.003/張）|

繞一圈結論：**「真正免費、不綁卡、又能讓第三方抓圖」的組合幾乎絕跡。**

### 目前的解（待接）

**Cloudflare Workers AI 的 `flux-1-schnell`**：

- 免費層每天 10,000 neurons，**不用綁信用卡**，只要免費 Cloudflare 帳號
- 一週才生一張圖，額度永遠用不完 → 實質 $0
- 回的是圖片 bytes（base64）→ 不是公開 URL

因為回 bytes，接法是：

```text
生成圖片 Prompt(英文) → Cloudflare flux-1-schnell(base64)
  → n8n 解碼 → commit 圖檔到 Hexo repo
  → 用 raw.githubusercontent URL 餵 IG（IG 抓 GitHub 很穩）
```

> TODO：拿到 Cloudflare Account ID + Workers AI token → curl 驗證生圖 → 接 workflow → 驗證 IG 真的發出帶圖的一篇。補圖、補結果。

---

## 暫定小結

- 兩個看似獨立的 bug（發兩篇、圖都動物），根因是**同一條接錯的線**。排查 n8n 先畫連線圖。
- n8n「同一輸入埠接多條線會重複觸發」，要合併分支用 Merge，別直接併線。
- 跨節點要共用的值（branch 名），**算一次、其他節點引用**，別每個節點各自用 `$now` 重算。
- 免費服務會退化：Pollinations 從「免簽免 key」變成 402 + 加密付款。架構別把命脈壓在單一免費服務上。
- 找來源這段也學到：**動手前先驗證**（列模型 ≠ 生得出來，free tier 可能 limit=0），而且**會花錢的事先問過再做**。

待 Cloudflare 接完補完整版。

(fin)
