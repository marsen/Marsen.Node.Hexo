---
title: "[工具筆記] Terminal 工具怎麼選？Ghostty、iTerm2 與其他選手比較"
date: 2026/03/11 06:05:19
tags:
  - 工具筆記
---

## 前情提要

最近有人問我 Ghostty 跟 iTerm2 哪個比較好，
順便聊到還有哪些選手值得比較，就整理一下。

## 選手介紹

### iTerm2

macOS 老牌 terminal，功能多到有點過頭。

split pane、tmux 整合、Triggers、Shell Integration、Python scripting API 全都有，
幾乎是 macOS 工程師的預設選項，做了很多年。

缺點是比較重，啟動慢一點，只能用在 macOS。

### Ghostty

2024 底爆紅的新秀，現在已經是很多人的主力工具。

Rust 的精神，GPU 加速渲染，開箱即用，預設樣式就乾淨好看。
支援 Kitty keyboard protocol，Nerd Fonts 圖示正常顯示，光標移動順到不行。

設定是純文字檔，簡單幾行就搞定：

```
font-family = "JetBrainsMono Nerd Font"
font-size = 14
theme = "GruvboxDark"
```

macOS + Linux 都支援。

### Warp

主打 AI 整合，內建 AI 指令輔助，Block-based UI，輸出可以區塊操作。

Rust 寫成，效能不錯。
最大的爭議是需要帳號登入，有隱私疑慮，這點讓不少人卻步。

### Alacritty

效能派代表，極簡哲學。

刻意拿掉 Tab、split pane、滑鼠增強，理由是「這些交給 tmux 做就好」。
純文字設定（TOML），跨平台（macOS / Linux / Windows）。

缺點是預設偏素，要自己調設定，不適合「裝好就好看好用」的需求。

### Kitty

GPU 加速，原生支援 split pane 和 tab。
Kitty keyboard protocol 就是它發明的，Ghostty 後來也採用。
macOS + Linux，設定有點多，學習曲線稍高。

### WezTerm

GPU 加速，Lua 腳本設定，功能豐富。
內建 multiplexer，split、tab、session 管理全包，不需要 tmux。
跨平台（macOS / Linux / Windows）。

---

## 快速比較

| | iTerm2 | Ghostty | Warp | Alacritty | Kitty | WezTerm |
|---|---|---|---|---|---|---|
| 效能 | 普通 | 快 | 快 | 極快 | 快 | 快 |
| 跨平台 | macOS only | mac+Linux | mac+Linux | 全平台 | mac+Linux | 全平台 |
| 設定方式 | GUI | 文字 | GUI | TOML | conf | Lua |
| AI 功能 | 無 | 無 | 有 | 無 | 無 | 無 |
| 內建 multiplexer | 無 | 無 | 無 | 無 | 有 | 有 |
| Nerd Fonts 支援 | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| 成熟度 | 高 | 新 | 中 | 高 | 高 | 中 |

---

## 補充：tmux 是什麼

很多文章提到要配合 tmux 使用，稍微說明一下。

tmux 是 **terminal multiplexer**，在一個 terminal 視窗裡模擬多視窗和分割畫面。
最大的賣點是 **session 保持**，關掉視窗程式不會死，SSH 到遠端時特別實用。

```
┌─────────────────────────────┐
│  tmux session               │
│  ┌──────────┬──────────┐   │
│  │  vim     │  shell   │   │
│  │          │          │   │
│  ├──────────┴──────────┤   │
│  │  npm run dev        │   │
│  └─────────────────────┘   │
└─────────────────────────────┘
```

現代 terminal（Ghostty、WezTerm、iTerm2）內建 split 和 Tab，
一般使用不需要特別學 tmux，除非有 SSH session 保持的需求。

---

## 我的建議

需求很簡單，**好看 + 圖示顯示 + 光標流暢**，直接選 **Ghostty**。

裝好之後指定一個 Nerd Font，10 分鐘搞定：

```bash
brew install --cask ghostty
brew install --cask font-jetbrains-mono-nerd-font
```

Nerd Fonts 常用選擇：

| 字型 | 特色 |
|------|------|
| JetBrains Mono NF | 易讀、開發者最愛 |
| FiraCode NF | ligature 好看 |
| Hack NF | 傳統乾淨 |

其他場景的建議：

- 需要 AI 輔助 → Warp
- 要配合 tmux、極致效能 → Alacritty
- 要 Lua 腳本、功能完整 → WezTerm
- 已經習慣、不想換 → iTerm2 繼續用也沒問題

## 小結

業界風向已經轉向 Ghostty，不是因為它功能最多，
而是它把**該有的都做好了**，多餘的不硬塞。

開箱好看、設定簡單、效能夠快，這樣就夠了。

(fin)
