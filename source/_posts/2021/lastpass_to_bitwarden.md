---
title: "[生活筆記] 拋棄 LastPass 擁抱 BitWarden"
date: 2021/09/14 09:15:44
---

## 前情提要

2021 年 3 月, 看到了一則新聞, 指出 LastPass 將改變它們的經營策略,
免費版本將只支援一種裝置, 行動裝置或電腦。
產品收費是天經地義, 但消費者找尋更高性價比的產品也是。  
讓我想離開 LastPass 的主因在於, 突襲式的宣布讓我幾乎沒有反應時間,  
免費版的限制太過苛刻, 一樣在限制裝置的功能上,  
EverNote 這款筆記軟體也是逐步降低免費版本可跨平台的數量。
但是 LastPass 更狠, 瞬間降到僅限一種, 這基本上是在玩文字遊戲,  
因為大多數的人應該都同時具備手機與個人電腦, 一般上族班只能被迫選擇電腦,  
或是付費了事。付費當然不是問題, 但宣告期極短且粗爆的方式讓人覺得被當韭菜割。

當我稍微作了一下功課, 發現這類的服務已經多如繁星, 甚至內建在作業系統或是瀏覽器之中。
再進一步比較功能, 與自已所需要的功能(密碼保管、密碼產生器、跨裝置etc…)
有更多免費的軟體已經能滿足我的需求，比如說: BitWarden

## 比較心得

BitWarden 的好處，

1. 免費帳戶即可跨裝置存取密碼  
2. 可以匯入 LastPass 匯出的密碼
3. 有密碼產生器
4. 有瀏覽器外掛，支援自動填入
5. 開源方案

最後一點我是覺得最吸引我的地方，這表示如果你有一定的程式語言能力，
你將可以自行搭建一個密碼管理的伺服器，這點甚至對企業也是有相當的吸引力，
而 BitWarden 使用的語言為 TypeScript、C# 與 Python。

在轉換上是非常容易的，可以在一小時內完成這項[作業](https://free.com.tw/migrate-passwords-from-lastpass-to-bitwarden/)，  
實務操作上也沒有遇到太大的困難，除了原本 LastPass 右鍵產生密碼的方式，在 BitWarden 上面沒有外，
其它功能或是外觀設計上，我都覺得 BitWarden 的表現更加優秀，
甚至有點相見恨晚的感覺。

至於沒有的功能，不是什麼大事，而且開源專案誰都可替他加上這個功能。
但對我來說，我已經忘了 LastPass 有什麼優點，
轉移到了 BitWarden 6 個月，從未再次開啟，於是今天正式將帳號刪除了。

## 參考

- [從 LastPass 匯出密碼、移轉到 BitWarden 密碼管理工具教學](https://free.com.tw/migrate-passwords-from-lastpass-to-bitwarden/)
- [密碼管理器搬家教學 - LastPass to BitWarden](https://infosecdecompress.com/posts/patches_password_manager_transfer_tutorial)
- [LastPass 被發現用了 7 個第三方追蹤程式](https://www.ithome.com.tw/news/142967)

(fin)
