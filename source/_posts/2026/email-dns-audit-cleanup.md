---
title: "[實作筆記] 盤點與清理自訂網域的 Email DNS 殘留（用郵件 header 當偵探）"
date: 2026/06/04 00:11:18
tags:
  - 實作筆記
---

## 前情提要

這幾天整理 Cloudflare 的 DNS，看到幾筆我沒有記憶也不理解的記錄：

```text
send.marsen.me   MX   feedback-smtp.ap-northeast-1.amazonses.com
```

Amazon SES？我什麼時候設過？還在用嗎？敢不敢刪？

不確定就不要亂刪 DNS——刪錯一筆，輕則信進垃圾桶，重則收不到信。所以乾脆把整個 `marsen.me` 上的 email 服務盤點一次，搞清楚每一筆 DNS 到底在幹嘛。

## 第一步，列出 DNS Record

把 DNS 裡跟 email 有關的記錄抓出來，對應到服務：

| 服務 | 相關 DNS 記錄 |
|　------　|--------------|
| Cloudflare Email Routing | `marsen.me` MX（route1-3.mx.cloudflare.net） |
| Brevo（原 Sendinblue） | `brevo1/2._domainkey` CNAME |
| Resend | `resend._domainkey` TXT |
| Amazon SES | `send.marsen.me` MX + SPF |

四套服務、一堆 DKIM/SPF/MX，光看 DNS 根本分不出哪些還在使用，用途為何？

## 第二步，理解 DNS Record

可以參考文章最末的參考資料，這裡簡單說明 MX 與 SPF，以後有機會再進一步說明

**MX**: 告訴別人「寄到 @你的網域 的信，送去哪台伺服器，  
  
你家的**收件信箱地址**，別人寄東西來才送得到。

**SPF / DKIM** = 你寄信時的**防偽印章和授權書**。

證明「這封信真的是你（或你授權的服務）寄的」，防止被當垃圾或冒名

所以一個純發信的服務（像 Brevo、Resend），只需要 DKIM 印章，**根本不需要 MX**。這個觀念是後面判讀的基礎。

> 補充：SPF 和 DKIM 的內容本體都是 TXT，但 DKIM 有兩種發布方式——
> 一種像 Resend 是把公鑰直接寫成 TXT 給你貼；
> 另一種像 Brevo 則是給你一筆 **CNAME**，把 `brevoX._domainkey` 委派指向它自家的 DNS，公鑰放在它那邊，方便它自己輪替金鑰。
> 所以上面表格裡 Brevo 是 CNAME、Resend 是 TXT，但做的是同一件事。

## 寄收信測試

我們可以查看**郵件 header**——它記錄了一封信實際走過的每一站。

方法很簡單：寄一封測試信給自己，打開「顯示原始郵件」看 header。

我用 `admin@marsen.me` 寄一封信給自已

```text
Message-Id: <...@smtp-relay.sendinblue.com>
Received: from hb.d.sender-sib.com (...)
DKIM-Signature: ... d=marsen.me s=brevo2 ...   → PASS
Feedback-ID: ...:Sendinblue
Return-Path: <...=hb.d.sender-sib.com=bounces-...@marsen.me>
```

`sendinblue`、`sender-sib`（SendInBlue 的縮寫）、`s=brevo2` 的 DKIM 簽章——這封信百分之百走 Brevo 發出。

`s=brevo2` 還正好對應到我 DNS 裡的 `brevo2._domainkey`。**Brevo 確認在用。**

### 同一封信，還證明了收信端

更妙的是，這封我寄給 `tester@marsen.me` 的信，被轉發回我 Gmail，header 把收信路徑也記下來了：

```text
Received: by cloudflare-email.net       → Cloudflare Email Routing 收到
X-Forwarded-To: me@gmail.com
X-Forwarded-For: tester@marsen.me me@gmail.com
Delivered-To: me@gmail.com
```

一封信同時驗證了兩端：**Brevo 發信** → **Cloudflare Email Routing 收信並轉發** → 進 Gmail。

## 那「收信」到底是怎麼運作的？

順著 header 就看懂了，我的收信是 **Cloudflare Email Routing**：

```text
別人寄到 xxx@marsen.me
  → marsen.me 的 MX 指向 Cloudflare（route1-3.mx.cloudflare.net）
  → Cloudflare Email Routing 收到
  → 依轉發規則（Routes）轉寄
  → 我的真實 Gmail
```

重點：Cloudflare Email Routing 是**轉發**，不是信箱。

它不存信，收到就立刻轉寄到你真正的 Gmail。所以 `@marsen.me` 沒有獨立的信箱空間，所有信最終都進同一個 Gmail。

整套自訂網域信箱長這樣：

```text
            ┌─ 收信 → Cloudflare Email Routing（MX）→ 轉發 ┐
@marsen.me ─┤                                              ├→ Gmail
            └─ 寄信 ← Brevo SMTP relay ← Gmail「以 admin@ 寄」┘
```

用免費服務，把 Gmail 變成 `@marsen.me` 的網域信箱。

這就是我免費達成私有 domain 信件寄收的作法，但是要注意 Brevo 有每日 300 封信的使用上限

## 盤點結果

三重交叉查證（程式碼 grep、n8n 備份、郵件 header）之後：

| 服務 | 狀態 | 證據 |
|　------|------|------|
| Cloudflare Email Routing | ✅ 在用 | header 轉發路徑 |
| Brevo | ✅ 在用 | header 的 sendinblue / brevo2 簽章 |
| Resend | ✅ 在用 | 電商專案發訂單信，程式碼有引用 |
| **Amazon SES** | ❌ **殘留** | 任何地方都找不到使用，連 SES 驗證記錄都沒有 |

SES 是當初設定網域信箱時評估過的備選方案（按量計費、便宜但設定複雜），最後選了 Brevo，

但 AWS 用的　`send.marsen.me` 的 MAIL FROM 記錄留下來沒清。

## 為什麼在用的 Brevo 沒有 MX，沒用的 SES 反而有？

這題剛好回扣前面「收信才用 MX」的觀念，但有個例外：

- **Brevo**：純發信，靠 DKIM + SPF 驗證授權，**不需要 MX**。所以它在用，卻沒有 MX。
- **SES**：那個 `send.marsen.me` 的 MX **不是收一般信**，而是 SES 的「custom MAIL FROM」機制要求的，用來接收**退信/投訴回報**（bounce/complaint）。

諷刺的是：沒在用的 SES 因為特殊機制留了 MX，在用的 Brevo 因為純發信反而沒有 MX。

光看「有沒有 MX」完全判斷不出哪個在用。

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

## 小結

盤點 email DNS 殘留，兩個心法：

1. **分清收信和發信**——MX 是收信，SPF/DKIM 是發信驗證。看到一筆記錄先問它管的是收還是發。
2. **用郵件 header 當偵探**——dashboard 會騙人，header 不會。寄封測試信，原始 header 裡寫得清清楚楚這封走了哪些服務。記得用**網域地址**寄，別用個人信箱，不然測了個寂寞。

殘留通常來自「評估過但沒採用的方案」——設定設了一半，淘汰後忘了清。盤點一次，把死記錄清掉，DNS 才乾淨。

## 補充：DNS 記錄類型速查

| 記錄 | 說明 |
| ------ | ------ |
| **MX** | 收信路由。告訴外部郵件伺服器「寄到這個網域的信，應該送到哪台 SMTP 伺服器」，數字優先度越小越優先，可設多筆做備援 |
| **TXT** | 純文字記錄，DNS 裡的萬用槽。SPF、DKIM 公鑰、網域所有權驗證都塞這裡 |
| **CNAME** | 別名記錄。把一個 subdomain 指向另一個 hostname，由對方那端負責最終解析。根網域（`marsen.me` 本身）不能用 |
| **SPF** | 發信授權白名單，格式寫在 TXT 裡（`v=spf1 ...`）。聲明哪些 IP 或服務有資格以這個網域寄信，收件方用來驗發件 IP |
| **DKIM** | 郵件防偽簽章。發信服務用私鑰簽章，公鑰掛在 DNS（TXT 或 CNAME 委派），收件方取公鑰驗簽，確認內容未被竄改 |
| **A** | 最基本的記錄。把網域直接對應到一個 IPv4 位址 |
| **AAAA** | 同 A，對應的是 IPv6 位址 |
| **NS** | 管轄聲明。指定這個網域的 DNS 問題，由哪幾台 Name Server 負責回答 |

## 參考

- [怎麼建立一個網站？(四) - 自訂網域 EMail 收寄信（使用 Cloudflare 與 Brevo）](/2025/08/27/2025/brevo_smtp/)

(fin)
