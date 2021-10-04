---
title: "[實作筆記] CPAU 加密仿登入"
date: 2019/08/20 10:46:10
---

## 前情提要

登入 SQL Server 需要受到權限管控，  
而在 Windows 中需要使用特定用戶身份登入時，  
我們常常會選擇 runas 命令。  

## 原始作法

大概類似如下:

```bash
runas.exe /netnoly /user:MyCompany\Marsen "C:\Program Files (x86)\Microsoft SQL Server\140\Tools\Binn\ManagementStudio\Ssms.exe"
```

我會寫成一個 ps (PowerShell) File 來執行這件事。
但是會有一些問題。

## 問題

1. 每次登入都輸入密碼好麻煩
2. 密碼很長又複雜，Terminal 又看不出輸入是否正確，常常會打錯
3. 密碼如果寫在 PowerShell 中會有風險

我希望點兩下就能以正確的身份開啟 SSMS 而且密碼不會被明碼存儲

## CPAU

1. 請先下載[CPAU](https://www.joeware.net/freetools/tools/cpau/)\
   - 我採用綠色安裝，如果有必要可以設為環境變數

2. 加密,以下的語法會產生以 `MyCompany\Marsen` 的身份登入執行 SSMS 的批次檔 `run.bat`

    ```bash
      CPAU -u MyCompany\marsen -p *************** -ex "C:\Program Files (x86)\Microsoft SQL Server\140\Tools\Binn\ManagementStudio\Ssms.exe" -enc -file D:\run.bat
    ```

3. 解密

    ```bash
    CPAU -file D:\run.bat -dec
    ```

最後只要將步驟 4 的語法存成一個 bat 檔，就可以隨時點兩下執行了。

## 免責聲明

雖然是加密過的檔案，不過如果被人知道你是使用 CAPU 加密，攻擊者取得檔案後仍然可以透過 CAPU 解密登入。
所以使用上一定要小心檔案不要外流，主要的優點仍然是節省打密碼的時間與打錯的風險。

## 參考

- [Runas](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771525(v%3Dws.11))
- [CPAU](https://www.joeware.net/freetools/tools/cpau/)
- [Rusas的替代方案CPAU](https://blog.xuite.net/billchu1109/wretch/142970080-Rusas%E7%9A%84%E6%9B%BF%E4%BB%A3%E6%96%B9%E6%A1%88CPAU)

(fin)
