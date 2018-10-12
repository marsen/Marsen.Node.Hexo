---
title: "[踩雷筆記]Docker 起動 MongoDB Server Error"
date: 2018/10/12 11:33:45
tag:
  - Mongodb
  - Docker
---

## 錯誤訊息

```
> docker-compose up db
Starting marsen-mongodb ... error
ERROR: for marsen-mongodb  Cannot start service db: driver failed programming external connectivity on endpoint marsen-mongodb (6f2be666a700187b7f30884478d7231cc210182e2650602e513c6627d0bc3e5a): Error starting userland proxy: mkdir /port/tcp:0.0.0.0:27017:tcp:172.18.0.2:27017: input/output error
```

## 除錯

**重啟 docker**

(fin)