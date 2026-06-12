---
title: "[實作筆記] n8n AI 週報踩坑記：一條接錯的線，與找免費生圖來源的旅程"
date: 2026/06/05 12:02:20
tags:
  - 實作筆記
---

## 前情提要

我有個 n8n workflow，每週自動抓 AI 新聞 → Gemini 生週報 → 發一篇 IG，順便生一張配圖、也產一篇 Blog 週報開 PR。

跑了一陣子，兩個毛病：

1. **每次發兩篇 IG**，一篇正常、一篇是空的。
2. **配圖每週都是動物**，跟當週主題八竿子打不著。

本來以為是兩個獨立問題，結果查下去——**根因是同一條接錯的線**。這篇記錄整個排查過程，還有後面為了「免費生圖」繞的一大圈。

---

## 流程長怎樣

出問題時的原始架構：

```text
Schedule → RSS×4 → Filter → Edit Fields → Aggregate
  ├─ 文字生成(Gemini)        → IG 準備發文 → IG 發佈
  ├─ 生成圖片 Prompt(Gemini)  → IG 準備發文   ↑（兩條線都接這）
  └─ Blog 生成(Gemini)       → GitHub 建 branch → commit → 開 PR
```

配圖用 `image.pollinations.ai` 生成，圖片 URL 直接餵給 IG。

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

## 問題二：圖片每週都動物

### 根因（其實還是那條線）

Pollinations 要的是「視覺場景描述」（英文、具體畫面），餵新聞摘要長文它抓不到重點 → 退化成安全牌，而動物是文字生圖最常見的 fallback。

但更前面的根因是：workflow 裡**其實已經有**一個「生成圖片 Prompt」的 Gemini 節點，prompt 也寫得不錯（挑最有畫面感的一則、用英文描述場景）。問題是它**接錯位置**——從 `Aggregate` 進來，而 Aggregate 輸出的是 `{ data: [...] }`，**沒有 `.text`**。節點模板寫 `{{ $json.text }}` → 拿到 undefined → 等於餵空內容 → 圖當然爛。

所以**同一個接線錯誤**：既讓 `生成圖片 Prompt` 拿不到週報內容，又跟 `文字生成` 併線造成重複發文。

### 修法

問題一的串接修法，順便就解了問題二——`生成圖片 Prompt` 接在 `文字生成` 之後，`$json.text` 真的拿得到週報全文了。再把它的 prompt 強化（固定畫風關鍵字 `digital illustration, vibrant colors, no text, no logo`、排除人名品牌、加 SFW 防護）。

> 教訓：兩個看起來無關的 bug，根因可能是同一個。先畫連線圖再動手。


## 插曲一：Blog 分支同天重跑撞名（422）

修完去重跑驗證，Blog 那條噴 `422 Reference already exists`。

根因：branch 名固定用 `weekly/{{yyyyMMdd}}`，同一天手動重跑就撞到自己上次建的 branch。正常每週排程日期不同不會撞，但測試時會。

修法：

- branch 名改成 `weekly/{{yyyyMMdd-HHmmss}}`，同天重跑不撞
- 後面 commit / 開 PR 的節點，branch 改成**讀「建 branch」實際回傳的 ref**（`.ref.replace('refs/heads/','')`），保證三個節點用同一個名字，不會各自用 `$now` 重算而漂移


## 插曲二：4 個 RSS 來源讓整條鏈跑多次

上線後跑了一次排程，GitHub 吐出 PR #65 和 PR #66——同一週跑出兩個週報 PR，IG 也連帶發了兩篇。

根因跟問題一其實是同一個模式：4 個 RSS 來源（Theverge、OpenAI、Google AI、Hackernews）各自直接連到 Filter 的 input 0，n8n 把每條來源視為獨立執行分支，Filter → Edit Fields → Aggregate 以下的整條鏈跑了 4 次，GitHub Create PR 就出現了 4 次（有的因 branch 撞名被 422 擋掉，但 IG 發文的那幾次成功了）。

修法：在 4 個 RSS Read 之後加一個 **Merge RSS 節點（append mode）**，把 4 條分支匯聚成一條再進 Filter，確保下游只跑一次。

```text
RSS×4 → Merge RSS（append）→ Filter → Edit Fields → Aggregate → ...
```

Merge 節點已加入，等下次排程（每週三）實際跑一次驗證。

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

### 最終解：Cloudflare Workers AI + GitHub 托管 + 一圖共用

先講結論：用 **Cloudflare Workers AI 的 `flux-1-schnell`** 生圖（免費、不綁卡），把圖 **commit 進 Hexo repo**，**文章嵌入它、IG 也用同一張的 raw URL**。

**為什麼是 Cloudflare Workers AI**

- 免費層每天 10,000 neurons，**不用綁信用卡**，只要免費 Cloudflare 帳號
- 一週一張圖，額度永遠用不完 → 實質 $0
- 實測 OK：回合法 1024×1024 JPEG（base64）

**為什麼一定要自己托管圖（這步躲不掉）**

IG 發「照片」貼文時，是給它一個**公開 URL、它自己伺服器去抓**——不能直接上傳圖片 bytes（只有 Reels／影片能 binary 上傳）。而 flux 回的是 bytes 不是 URL。所以中間一定要有個「把 bytes 變成公開 URL」的托管處。

評估過 R2 / GCS / Imgur / 備份 repo，最後選 **Hexo 的 public repo**：

- GitHub 憑證**早就接在 n8n 裡**、repo 又是 public → 零新帳號、零成本、IG 抓 `raw.githubusercontent.com` 很穩
- 備份 repo 不能用（private + 內含明文 token，不能公開）
- 巧合：flux 回 base64、GitHub Contents API 剛好也吃 base64，中間免轉檔

**一圖共用（這次的重點設計）**

原本 Blog 與 IG 是兩條平行分支。為了「文章嵌圖 + IG 用同一張」，把它們**串成一條**：生一次圖 → commit 到該週分支 → 文章 md 嵌入 `![](/images/...)` → IG 用同一張的 raw URL。一次生圖、零重複、圖也進了 blog。

```text
Aggregate → Blog生成 → 文字生成 → 生成圖片Prompt → CF生圖(flux,base64)
→ 取main SHA → 建分支(weekly/<日期-時間>) → 上傳圖到GitHub
→ Blog建檔(嵌圖) → Blog Post → 開PR → IG準備發文(同一張raw URL) → Wait → IG發佈
```

代價：兩條從「獨立」變「耦合」（中間任一步失敗，後面不跑）。換來一圖共用。

### 接的時候又踩的坑

- **n8n Header Auth 憑證填錯欄位 → 401**：`Name` 要填 `Authorization`（不是憑證用途名），`Value` 要 `Bearer <token>`（不是只貼 token）。
- **CLI import vs UI 編輯互相蓋**：用 CLI 匯入改的是 DB，但若瀏覽器還開著舊畫布，一存就把匯入蓋掉。改之前先關掉所有編輯分頁、改完再 refresh。
- **Gemini 503 過載**：`gemini-flash-latest` 遇大尖峰整個 workflow 掛。加 `retryOnFail`（4 次／5 秒）擋小尖峰；大範圍過載只能換模型（換成 `gemini-flash-lite-latest`）。已記成 backlog，想做「多模型自動 fallback」。
- **Blog CI 競態**：repo 的 `ci.yml` 在 `pull_request` 也跑，會在 PR 的 detached HEAD 上把 `ncu -u` 結果推回 main → 必然 `! [rejected] (fetch first)`，每開一個週報 PR 就失敗一次。修法：拿掉 `pull_request` 觸發 + 加 `concurrency`。

## 小結

- 兩個看似獨立的 bug（發兩篇、圖都動物），根因是**同一條接錯的線**。排查 n8n 先畫連線圖。
- n8n「同一輸入埠接多條線會重複觸發」，要合併分支用 Merge、別直接併線。
- 跨節點共用的值（branch 名）**算一次、其他節點引用**，別各自 `$now` 重算。
- 免費服務會退化：Pollinations 從「免簽免 key」變 402 + 加密付款；Gemini 圖片免費層 limit=0。架構別把命脈押在單一免費服務上。
- IG 發照片**一定要公開圖床**（它自己抓 URL），逼你先想清楚托管在哪。
- **動手前先驗證**（列得出模型 ≠ 生得出圖、free tier 可能 limit=0），而且**會花錢／動到別人系統的事先問過再做**。

（Merge RSS 節點已加，等下次排程實際跑過驗證。之後補上實際成品圖。）

(fin)
