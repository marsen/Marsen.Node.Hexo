---
title: " [實作筆記]使用 Windows PowerShell 批次上傳 AWS S3"
date: 2019/02/20 19:09:18
tag:
  - PowerShell
  - AWS
  - 實作筆記
---

## 目標

相關服務要雲端化，要將站台、資料等…遷移至 AWS，  
這個案例需要將大量在 File Server 上的檔案(報表/單據/報告書等…)上傳至 S3
因為檔案數量相當的大，所以開發一個簡單的指令來執行。

### Step 1 下載並安裝適用於 Windows PowerShell 的 AWS 工具

- [下載適用於 Windows PowerShell 的 AWS 工具](https://aws.amazon.com/tw/powershell/)

![只需要安裝必要的程式](/images/2019/2/awstools.jpg)

### 設定 aws config

打開 terminal 執行以下語法

```sh
> aws configure
```

依指示設定 `Access key ID` 與 `Secret access key`，  
這個資料需要具備一定的權限才能取得，如果權限不足請向你的 AWS 服務管理員申請。

### 撰寫 PowerShell 與執行

```powershell
$bucketName = "********************-your_bucket_name"
$path = "Your\s3\path\"

Get-ChildItem -Filter *.pdf |
ForEach-Object -Begin{$i=0} {
  $i++;
  $key = $path+$_ ;
  ## 進度顯示
  Write-Host $key "($i/1000)"  -ForegroundColor Green ;
  ## 上傳 S3
  Write-S3Object -BucketName $bucketName -File $_.FullName -Key $key ;
}
```

(fin)
