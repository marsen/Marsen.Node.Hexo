---
title: " [實作筆記] Linux NO Password 執行指令"
date: 2025/05/09 16:43:47
tags:
  - 實作筆記
---


## 前情提要

最近在部署某個 webapi server，  
這個 webapi 會用一個特別的帳號去執行，但是不會給他設定密碼，  
我需要使用 pm2 startup 設定開機自動啟動 webapi 服務。  

## 問題：執行指令會要求輸入密碼

但是我沒有也不打算提供密碼給個 webapi 服務帳號

```bash
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u marsen --hp /home/marsen
[sudo] password for marsen:
```

## 嘗試解法：用 visudo 寫 NOPASSWD

為了讓 `aiplux` 帳號執行特定指令時不需要輸入密碼，可以透過 `visudo` 編輯 `sudoers` 檔案。以下是具體步驟：

1. 開啟 `sudoers` 編輯器：

    ```bash
    sudo visudo
    ```

2. 在檔案中新增以下內容：

  ```bash
  aiplux ALL=(ALL) NOPASSWD: /usr/lib/node_modules/pm2/bin/pm2
  ```

這樣的設定可以確保 `aiplux` 帳號在執行 `/usr/lib/node_modules/pm2/bin/pm2` 時不需要輸入密碼，同時避免開放過多權限。

## 小心資安風險

上面的作法是改自另一位同事的作法，

```bash
echo "marsen ALL=(ALL) NOPASSWD: /usr/bin/apt, /usr/bin/apt-get, /usr/bin/env" | sudo tee /etc/sudoers.d/marsen >/dev/null
chmod 440 /etc/sudoers.d/marsen
```

這段設定的目的是：

1. 允許 `marsen` 自由使用 `apt` 和 `apt-get` 安裝套件。
2. 使用 `env` 包裝 PATH 或其他環境變數來執行 `pm2`。
3. 避免密碼卡住，讓 CI/CD 流程順利執行。

雖然給足夠的權限，可以無密碼執行，但是也帶來很多潛在風險。

資安問題：/usr/bin/env 是個危險洞口

看起來只是想加環境變數用 `env`，但其實這一條非常危險。為什麼？來看看 `env` 的本質：

```bash
sudo /usr/bin/env bash
sudo /usr/bin/env node
sudo /usr/bin/env python
```

這些指令會用 root 權限執行對應的 shell 或程式，等於你給了 `marsen` 完整 root 執行任意程式的能力，這十分危險。

再看 `/usr/bin/apt` 和 `apt-get`，同樣是高權限指令。如果沒有限制，也可以被濫用來刪除套件或改動系統。

## 小結

**審慎選擇開放的指令**
在自動化流程中，為了避免 sudo 密碼擋路，設定 `NOPASSWD` 是常見的解法。  
但在設定時，務必審慎選擇開放的指令範圍，避免一時圖方便，打開整台主機的後門。  
建議只開放必要的指令，並明確指定路徑，確保安全性。  

(fin)
