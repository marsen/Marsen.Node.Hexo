---
title: "[實作筆記] 盤點與清理自訂網域的 Email DNS 殘留（用郵件 header 當偵探）"
date: 2026/06/04 00:11:18
tags:
  - 實作筆記
---

## 前情提要

某天整理 Cloudflare 的 DNS，看到一筆我自己都看不懂的記錄：

```text
send.marsen.me   MX   feedback-smtp.ap-northeast-1.amazonses.com
```

Amazon SES？我什麼時候設過？還在用嗎？敢不敢刪？

不確定就不要亂刪 DNS——刪錯一筆，輕則信進垃圾桶，重則收不到信。所以乾脆把整個 `marsen.me` 上的 email 服務盤點一次，搞清楚每一筆 DNS 到底在幹嘛。

結論先講：我的網域上同時掛了**四套** email 服務，其中三套在用、一套是殘留。盤點的關鍵工具是「郵件 header」——它不會騙人。

---

## 我的網域上有幾套 email 服務？

把 DNS 裡跟 email 有關的記錄抓出來，對應到服務：

| 服務 | 相關 DNS 記錄 | 功能 |
|------|--------------|------|
| Cloudflare Email Routing | `marsen.me` MX（route1-3.mx.cloudflare.net） | 收信 |
| Brevo（原 Sendinblue） | `brevo1/2._domainkey` CNAME | 發信 |
| Resend | `resend._domainkey` TXT | 發信（電商訂單通知） |
| Amazon SES | `send.marsen.me` MX + SPF | ？ |

四套服務、一堆 DKIM/SPF/MX，光看 DNS 根本分不出哪些還活著。要查證，得先搞懂一件事。

---

## 先搞懂：收信和發信，用的是不同的 DNS

這是 email DNS 最容易混淆的地方：

| 功能 | 用什麼記錄 | 作用 |
|------|-----------|------|
| **收信** | **MX** | 告訴別人「寄到 @你的網域 的信，送去哪台伺服器」 |
| **發信驗證** | **SPF + DKIM** | 證明「這封信真的是你（或你授權的服務）寄的」，防止被當垃圾或冒名 |

打個比方：

- **MX** = 你家的**收件信箱地址**，別人寄東西來才送得到。
- **SPF / DKIM** = 你寄信時的**防偽印章和授權書**，證明這封是你本人或你授權的人寄的。

所以一個純發信的服務（像 Brevo、Resend），只需要 DKIM 印章，**根本不需要 MX**。這個觀念是後面判讀的基礎。

---

## 怎麼確認哪個服務「真的在用」？

我一開始想去各家 dashboard 看用量，但 dashboard 的計量會有延遲、也可能看錯頁面。

最硬的證據是**郵件 header**——它記錄了一封信實際走過的每一站。方法很簡單：寄一封測試信給自己，打開「顯示原始郵件」看 header。

### 踩坑：第一次測錯了

我第一次直接從 Gmail 寄了封信，看 header：

```text
From: <me@gmail.com>
Message-ID: <...@mail.gmail.com>
```

寄件人是我的**個人 Gmail**，不是 `admin@marsen.me`。當然測不到 Brevo——因為 Brevo 只負責「以 `admin@marsen.me` 寄信」。我用個人信箱寄，走的是 Gmail 自己的發送。

**要測網域信箱的發信，寄信時的寄件人就得選網域地址。** 在 Gmail 撰寫新信時，From 欄位下拉選 `admin@marsen.me`（前提是你有設定 send-as）。

### 第二次：header 給了鐵證

改用 `admin@marsen.me` 寄出後，header 立刻露餡：

```text
Message-Id: <...@smtp-relay.sendinblue.com>
Received: from hb.d.sender-sib.com (...)
DKIM-Signature: ... d=marsen.me s=brevo2 ...   → PASS
Feedback-ID: ...:Sendinblue
Return-Path: <...=hb.d.sender-sib.com=bounces-...@marsen.me>
```

`sendinblue`、`sender-sib`（SendInBlue 的縮寫）、`s=brevo2` 的 DKIM 簽章——這封信百分之百走 Brevo 發出。`s=brevo2` 還正好對應到我 DNS 裡的 `brevo2._domainkey`。**Brevo 確認在用。**

### 同一封信，還證明了收信端

更妙的是，這封我寄給 `tester@marsen.me` 的信，被轉發回我 Gmail，header 把收信路徑也記下來了：

```text
Received: by cloudflare-email.net       → Cloudflare Email Routing 收到
X-Forwarded-To: me@gmail.com
X-Forwarded-For: tester@marsen.me me@gmail.com
Delivered-To: me@gmail.com
```

一封信同時驗證了兩端：**Brevo 發信** → **Cloudflare Email Routing 收信並轉發** → 進 Gmail。

---

## 那「收信」到底是怎麼運作的？

順著 header 就看懂了，我的收信是 **Cloudflare Email Routing**：

```text
別人寄到 xxx@marsen.me
  → marsen.me 的 MX 指向 Cloudflare（route1-3.mx.cloudflare.net）
  → Cloudflare Email Routing 收到
  → 依轉發規則（Routes）轉寄
  → 我的真實 Gmail
```

重點：Cloudflare Email Routing 是**轉發**，不是信箱。它不存信，收到就立刻轉寄到你真正的 Gmail。所以 `@marsen.me` 沒有獨立的信箱空間，所有信最終都進同一個 Gmail。

整套自訂網域信箱長這樣：

```text
            ┌─ 收信 → Cloudflare Email Routing（MX）→ 轉發 ┐
@marsen.me ─┤                                              ├→ Gmail
            └─ 寄信 ← Brevo SMTP relay ← Gmail「以 admin@ 寄」┘
```

用免費服務，把 Gmail 變成 `@marsen.me` 的網域信箱。

---

## 盤點結果

三重交叉查證（程式碼 grep、n8n 備份、郵件 header）之後：

| 服務 | 狀態 | 證據 |
|------|------|------|
| Cloudflare Email Routing | ✅ 在用 | header 轉發路徑 |
| Brevo | ✅ 在用 | header 的 sendinblue / brevo2 簽章 |
| Resend | ✅ 在用 | 電商專案發訂單信，程式碼有引用 |
| **Amazon SES** | ❌ **殘留** | 任何地方都找不到使用，連 SES 驗證記錄都沒有 |

SES 是當初設定網域信箱時評估過的備選方案（按量計費、便宜但設定複雜），最後選了 Brevo，但 `send.marsen.me` 的 MAIL FROM 記錄留下來沒清。

---

## 為什麼在用的 Brevo 沒有 MX，沒用的 SES 反而有？

這題剛好回扣前面「收信才用 MX」的觀念，但有個例外：

- **Brevo**：純發信，靠 DKIM + SPF 驗證授權，**不需要 MX**。所以它在用，卻沒有 MX。
- **SES**：那個 `send.marsen.me` 的 MX **不是收一般信**，而是 SES 的「custom MAIL FROM」機制要求的，用來接收**退信/投訴回報**（bounce/complaint）。

所以很諷刺：沒在用的 SES 因為特殊機制留了 MX，在用的 Brevo 因為純發信反而沒有 MX。光看「有沒有 MX」完全判斷不出哪個在用。

---

## 安全清理

確認 SES 是殘留後，要刪的只有兩筆（name 都是 `send.marsen.me`）。

**刪之前先把完整內容記下來**——理論上它一定沒用了，但留個底，萬一哪天要恢復可以照著重建：

```ini
# ── 被刪除的 SES 殘留記錄（刪除日：2026-06-04，留供日後恢復）──

# 第一筆：SES custom MAIL FROM 的 bounce 接收
Type     = MX
Name     = send.marsen.me
Content  = feedback-smtp.ap-northeast-1.amazonses.com
Priority = 10
Proxy    = DNS only
TTL      = 1 hour

# 第二筆：SES 的 SPF
Type     = TXT
Name     = send.marsen.me
Content  = "v=spf1 include:amazonses.com ~all"
Proxy    = DNS only
TTL      = 1 hour
```

> 註：若真要復原 Amazon SES 寄信，光補這兩筆 DNS 不夠，還得回 AWS Console（`ap-northeast-1` 東京）重新驗證 identity 與 custom MAIL FROM。這兩筆只是當時設定留下的 DNS 半成品。

最容易刪錯的地方——**這筆千萬別動**：

```text
marsen.me   MX   route1/2/3.mx.cloudflare.net   ← 這是收信用的！
```

`send.marsen.me` 的 MX（要刪）和 `marsen.me` 的 MX（要留）長很像，但一個是 SES 殘留、一個是你的收信命脈。**認準 name 欄位有沒有 `send.` 前綴。**

其他一律別碰：各家 `_domainkey`（DKIM 印章）、`_dmarc`、主網域的 MX/SPF，都還在用。

如果想清得更徹底，可以登入 AWS Console（SES 在 `ap-northeast-1` 東京）刪掉 identity 設定，順便看一眼 Billing 有沒有忘記關的資源在計費——閒置的雲端帳號最容易偷偷扣錢。

---

## 小結

盤點 email DNS 殘留，兩個心法：

1. **分清收信和發信**——MX 是收信，SPF/DKIM 是發信驗證。看到一筆記錄先問它管的是收還是發。
2. **用郵件 header 當偵探**——dashboard 會騙人，header 不會。寄封測試信，原始 header 裡寫得清清楚楚這封走了哪些服務。記得用**網域地址**寄，別用個人信箱，不然測了個寂寞。

殘留通常來自「評估過但沒採用的方案」——設定設了一半，淘汰後忘了清。盤點一次，把死記錄清掉，DNS 才乾淨。

## 參考

- [怎麼建立一個網站？(四) - 自訂網域 EMail 收寄信（使用 Cloudflare 與 Brevo）](/2025/08/27/2025/brevo_smtp/)

(fin)
