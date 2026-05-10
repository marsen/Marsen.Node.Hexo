---
title: "[實作筆記] 用 GitHub Projects + Skills 管理個人 Backlog：踩過的授權坑"
date: 2026/05/11 06:52:14
tags:
  - 實作筆記
---

## 前情提要

想把個人 idea 清單整合進 GitHub Projects，
同時設定 GitHub Actions 讓新 Issue 自動加進 Project 看板。
過程踩了幾個授權的坑，記錄一下。

## 授權流程

用 `gh` CLI 操作 GitHub Projects 和 Actions 需要分開授權，
預設的 token 不夠用。

### Step 1：Project 讀寫權限

要新增 Issue 到 Project、建立 Project item，需要 `project` 和 `read:project` scope：

```bash
gh auth refresh -s project,read:project
```

執行後會跳出 device flow：

```text
! First copy your one-time code: 70C4-9535
Press Enter to open https://github.com/login/device in your browser...
✓ Authentication complete.
```

### Step 2：Workflow 修改權限

Push `.github/workflows/` 目錄下的檔案需要額外的 `workflow` scope，
即使已有 `repo` 完整寫入權限也不夠：

```bash
gh auth refresh -s workflow
```

一樣走 device flow，完成後才能 push workflow 檔案。

## 為什麼要分開？

GitHub 的設計：CI/CD workflow 修改權限獨立出來，
防止 OAuth App 未經授權竄改 Actions。
錯誤訊息長這樣：

```text
refusing to allow an OAuth App to create or update workflow
```

看到這個就知道要補 `workflow` scope。

## GitHub Actions 自動加 Issue 進 Project

workflow 長這樣：

```yaml
name: Add Issue to Project

on:
  issues:
    types: [opened]

jobs:
  add-to-project:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/add-to-project@v1.0.2
        with:
          project-url: https://github.com/users/marsen/projects/2
          github-token: ${{ secrets.PROJECT_TOKEN }}
```

`PROJECT_TOKEN` 需要在 repo settings 裡設定 secret，
用一個有 `project` scope 的 PAT。

## 小結

- `gh project` 操作要先 `gh auth refresh -s project,read:project`
- Push `.github/workflows/` 要先 `gh auth refresh -s workflow`
- 兩個都是 device flow，不能在非互動環境下跑（例如 Claude Code 的 `!` 指令）

(fin)
