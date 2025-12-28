---
title: " [實作筆記] 我的 Vim 設定"
date: 2025/05/24 13:06:05
tags:
  - 實作筆記
---

我的 Vim 快捷鍵設定總表，**IdeaVim 與 JetBrain**：

---

| 快捷鍵         | 模式 | 功能說明                                 | IdeaVim 設定寫法                           |
|----------------|------|------------------------------------------|--------------------------------------------|
| zcc            | n    | 清除所有自訂映射                         | `nmap zcc :mapclear!<CR>`                  |
| zso            | n    | 重新載入主 vimrc                         | `nmap zso :source ~/_ideavimrc<CR>`        |
| zf             | n/i  | 跳到定義（Goto Declaration）             | `nmap zf :action GotoDeclaration<CR>`<br>`imap zf <Esc>:action GotoDeclaration<CR>` |
| zrc, zo        | n/i  | 格式化程式碼（Reformat Code）            | `nmap zrc :action ReformatCode<CR>`<br>`imap zrc <Esc>:action ReformatCode<CR>`<br>`nmap zo :action ReformatCode<CR>` |
| zk             | n/i  | 返回（Back）                             | `nmap zk :action Back<CR>`<br>`imap zk <Esc>:action Back<CR>` |
| zj             | n/i  | AceAction（自訂動作）                    | `nmap zj :action AceAction<CR>`<br>`imap zj <Esc>:action AceAction<CR>` |
| ztt            | n/i  | 複製區塊並格式化（自訂巨集）             | `nmap ztt y?[F<CR>/{<CR>%o<Esc>p:action ReformatCode<CR>`<br>`imap ztt <Esc>y?[F<CR>/{<CR>%o<Esc>p:action ReformatCode<CR>` |
| zn             | n/i  | 跳到上一個錯誤（Goto Previous Error）    | `nmap zn :action GotoPreviousError<CR>`<br>`imap zn <Esc>:action GotoPreviousError<CR>` |
| zrr            | n/i  | 重新命名（Rename Element）               | `nmap zrr :action RenameElement<CR>`<br>`imap zrr <Esc>:action RenameElement<CR>` |
| zra            | n/i  | 執行所有單元測試                         | `nmap zra :action RiderUnitTestRunSolutionAction<CR>`<br>`imap zra <Esc>:action RiderUnitTestRunSolutionAction<CR>` |
| zrm            | n/v  | 萃取方法（Extract Method）               | `nmap zrm :action ExtractMethod<CR>`<br>`vmap zrm :action ExtractMethod<CR>` |
| zri            | n    | Inline（內聯變數/方法）                  | `map zri :action Inline<CR><Esc>`           |
| zrp            | n    | Introduce Parameter（引入參數）          | `map zrp :action IntroduceParameter<CR>`    |
| zrv            | n    | Introduce Variable（引入變數）           | `map zrv :action IntroduceVariable<CR>`     |
| zrf            | n    | Introduce Field（引入欄位）              | `map zrf :action IntroduceField<CR>`        |
| ,m             | n    | 顯示檔案結構                             | `nmap ,m :action FileStructurePopup<CR>`    |
| zrt            | n    | 最近檔案                                 | `nmap zrt :action RecentFiles<CR>`          |
| zgc            | n    | 開啟 Commit 工具視窗                     | `nmap zgc :action ActivateCommitToolWindow<CR>` |
| zx             | n    | 快速重構選單                             | `nmap zx :action Refactorings.QuickListPopupAction <CR>` |
| zae            | n/i  | 插入 Assert.Equal(,);                    | `nmap zae aAssert.Equal(,);<Esc>T(i`<br>`imap zae Assert.Equal(,);<Esc>T(i` |
| zrs            | n    | 新增 class 並格式化（自訂巨集）          | `nmap zrs "add? class<CR>2w"bywjopublic <Esc>"bpa()<CR>{<Esc>"apo}<Esc>:action ReformatCode<CR>` |
| z;             | n/i  | 行尾加分號                               | `nmap z; $a;<Esc>`<br>`imap z; <Esc>$a;`    |
| ,p, ,P         | n    | 黏貼暫存區 0 的內容                      | `nmap ,p "0p`<br>`nmap ,P "0P`              |
| z,p, z,P       | i    | 插入暫存區 0 的內容                      | `imap z,p <Esc>"0pa`<br>`imap z,P <Esc>"0Pa`|
| z,             | n    | 選取括號內內容（vi)）                    | `:nmap z, vi)`                              |
| z.             | n    | 選取大括號內內容（vi}）                  | `:nmap z. vi}`                              |
| <Esc>          | v    | 連續多次離開選取模式                     | `:vmap <Esc> <Esc><Esc><Esc>`               |
| jj             | i    | 退出插入模式                             | `:imap jj <Esc>`                            |
| <BS>           | n    | 在普通模式下插入 Backspace               | `:nmap <BS> a<BS>`                          |
| zh             | n/i  | 跳到行首                                 | `:nmap zh ^`<br>`:imap zh <Esc>^i`          |
| zl             | n/i  | 跳到行尾                                 | `:nmap zl $`<br>`:imap zl <End>`            |
| hc             | n    | Ctrl+C                                   | `:nmap hc ^C`                               |
| zb             | n/v  | 單字取代                                 | `:nmap zb bcw`<br>`:vmap zb <Esc>bcw`       |
| zd             | i    | 刪除當前行                               | `:imap zd <Esc>dd`                          |
| j, k           | n    | 軟換行移動                               | `nmap j gj`<br>`nmap k gk`                  |
| qq             | n    | 強制關閉檔案                             | `nmap qq ZQ`                                |
| zq             | n    | 儲存並關閉檔案                           | `nmap zq :wq<CR>`                           |
| <C-x>          | i    | 刪除當前行                               | `:imap <C-x> <Esc>dd`                       |
| <C-a>          | i    | 全選                                     | `:imap <C-a> <Esc>ma<CR>ggVG`               |
| ,i, ,a         | n    | 替換字元                                 | `:map ,i <Esc>r`<br>`:map ,a <Esc>r`        |

---

**說明：**  

- n：普通模式（normal mode）  
- i：插入模式（insert mode）  
- v：視覺模式（visual mode）  
- `<CR>` 代表 Enter 鍵，`<Esc>` 代表 Esc 鍵

---

以下是我將 JetBrains（IdeaVim）Vim 快捷鍵移植到 VSCode（VSCodeVim）的對應表，  
並說明哪些已成功移植、哪些無法移植及原因：

---

| JetBrains 快捷鍵 | VSCode 設定 (before) | VSCode 指令 (commands)                | 功能說明                   | 可否移植 | 備註/原因                                       |
|------------------|----------------------|----------------------------------------|----------------------------|----------|--------------------------------------------------|
| zrr              | ["z","r","r"]        | editor.action.rename                   | 重新命名                   | ✅       | 完全支援                                         |
| zf               | ["z","f"]            | editor.action.revealDefinition         | 跳到定義                   | ✅       | 完全支援                                         |
| zrc/zo           | ["z","r","c"]/["z","o"] | editor.action.formatDocument         | 格式化程式碼               | ✅       | 完全支援（你已設 zrc，zo 也可加）                |
| zn               | ["z","n"]            | editor.action.marker.prev              | 跳到上一個錯誤             | ✅       | 完全支援                                         |
| zrt              | ["z","r","t"]        | workbench.action.openRecent            | 最近檔案                   | ✅       | 完全支援                                         |
| ,m               | [",","m"]            | workbench.action.gotoSymbol            | 檔案結構/大綱              | ✅       | 完全支援                                         |
| zx               | ["z","x"]            | editor.action.refactor                 | 顯示重構選單               | ✅       | 完全支援                                         |
| zgc              | ["z","g","c"]        | workbench.view.scm                     | 開啟 Source Control 面板   | ✅       | 完全支援                                         |
| zh               | ["z","h"]            | cursorHome                             | 跳到行首                   | ✅       | 完全支援                                         |
| zl               | ["z","l"]            | cursorEnd                              | 跳到行尾                   | ✅       | 完全支援                                         |
| zrv              | ["z","r","v"]        | editor.action.codeAction.extract.variable | 萃取變數                | ⚠️       | VSCodeVim 有選取限制，部分情境下無法觸發         |
| zrm              | ["z","r","m"]        | editor.action.codeAction.extract.method   | 萃取方法                | ⚠️       | VSCodeVim 有選取限制，部分情境下無法觸發         |
| zri              | -                    | -                                      | Inline（內聯變數/方法）    | ❌       | VSCode 沒有公開指令 ID，無法 keybinding          |
| zrp              | -                    | -                                      | Introduce Parameter        | ❌       | VSCode 沒有公開指令 ID，無法 keybinding          |
| zrf              | -                    | -                                      | Introduce Field            | ❌       | VSCode 沒有公開指令 ID，無法 keybinding          |
| ztt              | -                    | -                                      | 巨集/多步驟自動化          | ❌       | VSCode 不支援 Vim 巨集或多步驟自動化             |
| zso              | -                    | -                                      | 重新載入 vimrc             | ❌       | VSCodeVim 不支援 :source 或重新載入 vimrc        |
| zra              | ["z","r","a"]        | testing.runAll                          | 執行所有單元測試           | ✅       | 需安裝 VSCode 官方 Testing 功能，已可 keybinding |

---

### 補充說明

- ✅：完全支援，已可用於 VSCodeVim。
- ⚠️：VSCodeVim 有技術限制（如需選取內容，觸發不一定成功）。
- ❌：VSCode/VSCodeVim 無對應指令或功能，無法移植。
- 你可以持續用 VSCode 內建的 `Cmd+.` 或右鍵選單補足無法 keybinding 的重構功能。
- **zra**：VSCode 1.59 以上內建 Testing 功能，指令為 `testing.runAll`，可直接 keybinding，無需額外外掛。

---

**總結：**  
我已成功將大部分常用 JetBrains Vim 快捷鍵移植到 VSCode，  
僅少數 JetBrains 專屬重構、巨集、vimrc 相關功能因 VSCode 限制無法移植。  

(fin)
