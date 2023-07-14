---
title: "[實作筆記] GitHub Actions：停用 save-state 和 set-output 命令的措施"
date: 2023/07/14 18:12:02
tags:
  - 實作筆記
---

## 前情提要

GitHub Actions 是一個強大的自動化工作流程工具，可讓您在 GitHub 存儲庫中執行各種自動化任務。　　
它允許您根據事件觸發工作流程，例如提交代碼或創建拉取請求。　　
您可以使用預設的操作或自定義操作來建立工作流程，並將其用於自動化測試、部署、持續集成等開發流程。　　
GitHub Actions 提供了一個靈活、可擴展和可自訂的方式來增強您的開發工作流程。

而我實務上的情境是用來寫 Blog，並進行自動部署，而我注意到了一個警告訊息

```shell
Warning: The `set-output` command is deprecated and will be disabled soon. Please upgrade to using Environment Files. For more information see: https://github.blog/changelog/2022-10-11-github-actions-deprecating-save-state-and-set-output-commands/
```

## 問題

在 Github 的官方 Blog 指出，自 CI Runner 版本 2.298.2 起，  
如果用戶在使用 save-state 或 set-output 將開始發出警告。  
並計劃在 2023 年 5 月 31 日完全禁用它們。  
從 2023 年 6 月 1 日開始，使用 `save-state` 或 `set-output` 命令的工作流程將因錯誤而失敗。

### 補充

實際上根據 Github 觀測數據顯示，這些命令的使用率相當高。考慮到受影響的客戶數量，而推遲了移除的時間。

## 我的情況

為什麼我會用到這值呢？看看我的 CD 部署檔

```yaml
## Skip ...
step:
  # Deploy hexo blog website.
  - name: Deploy
    id: deploy
    uses: marsen/hexo-action@v1.0.8
    with:
      deploy_key: ${{ secrets.DEPLOY_KEY }}
  - name: Get the output
    run: |
      echo "${{ steps.deploy.outputs.notify }}"

## Skip ...
```

在我這裡有兩步驟，Deploy 　與 Get the output
Deploy 直接用了我的另一專案進行部署，內部的工作簡單的說明如下：

1. 用 Dockerfile 建立執行環境
2. 拉取 Hexo Blog
3. 執行相關指令 ex: `hexo g`
4. 部署完成印出成功訊息

問題就出在這個第四步，在 Github Runner 啟動的 image 執行過程的輸出，　　
並不會顯示在 Action Steps 的 Output 資料之中。
所以在這個專案多作一步 `Get the output` 來印出這個訊息。

```yaml
- name: Get the output
  run: |
    echo "${{ steps.deploy.outputs.notify }}"
```

可以知道我們的訊息是放在 `steps.deploy.outputs.notify` 之中，
而原始接出資料的方式如下，這個作法即將被逃汰了(為了避免無意間的資訊外洩)

```shell
echo ::set-output name=notify::"Deploy complate."
```

## 解決方式

如果官方 Blog 所說，你只要將 key-value 用以下的形式設定到環境變數 `$GITHUB_OUTPUT` 即可，

```shell
echo "{key}={value}" >> $GITHUB_OUTPUT
```

舉例來說:

```shell
result="Deploy complate."
echo "notify=$result" >> $GITHUB_OUTPUT
```

## 參考

- [GitHub Actions: Deprecating save-state and set-output commands](https://github.blog/changelog/2022-10-11-github-actions-deprecating-save-state-and-set-output-commands/)
- [[實作筆記] Hexo CI/CD 設置](https://blog.marsen.me/2022/09/26/2022/Hexo_CICD/)
- [[實作筆記] Hexo CI 自動執行 ncu -u 更新相依套件](https://blog.marsen.me/2022/09/26/2022/Hexo_CICD/)

(fin)
