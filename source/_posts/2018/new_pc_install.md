---
title: "[實作筆記]重灌開發環境"
date: 2018/04/17 11:34:51
tag:
    - 實作筆記
---

## 1.Typescript 踩雷

### 問題

visual studio 預設會安裝 typescript 2.6
專案使用 typescript 2.3 , 因為暫時無法升級到2.6以上的版本
會導致專案無法編譯成功

### 解決步驟

1. 在專案目錄執行 `npm i` 重新安裝相關module
2. complie 後發現 `node_modules/@types` 中有檔案無法成功編譯
3. 移除 `node_modules/@types` 整個資料夾
4. 重新 complie 後仍會無法成功
5. 移除 `C:\Program Files (x86)\Microsoft SDKs\TypeScript\2.6` (非必要,好像要看vs預設載入的版本為何?)

## 2.多語系dll衝突

1. 清空`bin`資料夾
2. 清空 `c:\Windows\Microsoft.NET\Framework64\v4.0.30319\Temporary ASP.NET Files\`資料夾
3. 重建前台專案

## 3.Chocolatey

```shell=
 choco install googlechrome -y
 choco install dropbox -y
 choco install evernote -y

 choco install git -y
 choco install nodejs -y
 choco install putty -y
 choco install visualstudiocode -y

 choco install winmerge -y
 choco install slack -y
 choco install linqpad -y
 choco install 7zip -y
  
 choco install gitkraken -y
 #choco install sourcetree -y
```

(fin)
