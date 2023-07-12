---
title: " [實作筆記] 錯誤處理 dotnet core @ GKE logger "
date: 2022/06/09 16:37:53
tags:
  - 實作筆記
---

## 問題

為了追蹤 Dotnet Core 專案上的 log ，
參考[這篇](https://cloud.google.com/dotnet/docs/reference/Google.Cloud.Diagnostics.AspNetCore3/latest)文章進行了設定，
沒想到在 GKE 上啟動服務時，會產生錯誤，導致服務啟動異常

```text
system.io.ioexception:
error loading native library "/app/runtimes/linux-x64/native/libgrpc_csharp_ext.x64.so".
error loading shared library ld-linux-x86-64.so.2:
no such file or directory (needed by /app/runtimes/linux-x64/native/libgrpc_csharp_ext.x64.so)
```

## 原因

看起來是專案啟動之時，會需要底層的 `libgrpc_csharp_ext.x64.so` 檔案。
[tl;dr](https://github.com/grpc/grpc/issues/21446#issuecomment-1067926001)
簡而言之，我使用的 docker image `mcr.microsoft.com/dotnet/aspnet:6.0-alpine`，
裡面沒有包 Grpc.Core ，但是 Grpc.Core 壞壞要被 grpc-dotnet 汰換([tl;dr](https://grpc.io/blog/grpc-csharp-future/?ref=https://githubhelp.com))
所以要嘛你自已在 alpine image 自已裝上 `libgrpc_csharp_ext.x64.so`，
或是使用一個不公開的 image [版本](https://github.com/grpc/grpc/issues/21446#issuecomment-998966196)，
ex:`6.0.1-cbl-mariner1.0-distroless-amd64`
這些不公開的版本可以在[這裡查到喔](https://mcr.microsoft.com/v2/dotnet/aspnet/tags/list)

## 參考

- <https://github.com/grpc/grpc/issues/21446>

(fin)
