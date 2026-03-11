---
title: "[實作筆記] Ghostty 打字出現 [O[27u 亂碼的原因與修正"
date: 2026/03/11 14:08:43
tags:
  - 實作筆記
---

## 前情提要

剛裝好 Ghostty，打字有時正常，有時畫面會出現像這樣的亂碼：

```text
[O[27u
```

查了一下，原因跟 Ghostty 預設啟用的 **Kitty Keyboard Protocol** 有關。

## 原因

Ghostty 預設使用 `TERM=xterm-ghostty`，同時啟用 **Kitty Keyboard Protocol**（又叫 CSI u encoding）。

這個協定會把按鍵轉成擴展的 escape sequence，格式長這樣：

```text
ESC [ <code> u
```

例如 Escape 鍵 → `ESC[27u`，搭上其他序列就變成你看到的 `[O[27u`。

這本來是為了解決傳統終端機鍵碼不夠精確的問題，現代工具（Neovim、WezTerm、Kitty）都支援。

但如果你的 shell 或其他程式**不支援**這個協定，就不會解析這些 escape sequence，直接把它們當一般文字印出來，就是你看到的亂碼。

## 修正方法

改掉 `TERM` 設定，換成廣泛相容的 `xterm-256color`，讓 Ghostty 不送出這些擴展序列。

建立或編輯 `~/.config/ghostty/config`，加入：

```ini
term = xterm-256color
```

完整設定檔範例：

```ini
# ~/.config/ghostty/config

# 使用廣泛相容的 TERM，避免 Kitty keyboard protocol 造成亂碼
term = xterm-256color
```

存檔後**完全關閉 Ghostty 再重開**（Cmd+Q，不只是關視窗），讓設定生效。

## 如果你有用 tmux

tmux 本身也可能會攔截或破壞這些 escape sequence，在 `~/.tmux.conf` 加入：

```tmux
set -g extended-keys on
set -as terminal-features 'xterm-ghostty:extkeys'
```

或直接關掉：

```tmux
set -g extended-keys off
```

最簡單的做法還是先改 `term = xterm-256color`，大多數情況直接解決。

## 小結

Ghostty 預設啟用 Kitty Keyboard Protocol，對不支援的程式來說會直接吐出 `[27u` 這類亂碼。
改一行設定 `term = xterm-256color` 就搞定，完全關掉重開生效。

(fin)
