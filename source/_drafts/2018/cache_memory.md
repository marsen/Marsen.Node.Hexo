---
title: "[內部分享]Cache 使用 Memory 與 Redis"
date: 2018/09/26 13:43:45
tag:
  - Cache
  - .Net
---

## Memory 
  - 阻檔突波
  - 記憶體碎
  - 不同機器不同步
    - 可能會導致只有某有一台有問題
  - 吃主機資源(memory)
  - Memory Leak (Cache 不要超過一小時 ?/Size要控制)
  - Hit Rate  
  - 可以透過 Memory Dump 找問題。
  - 主動更新 Memory Cache
    - .Net → Dependency SQL / File (不建議這樣作了，在大型的系統中反而會吃掉過多的主機資源)
  - 被動, 時間到就過期
  - 跟據不同商業邏輯要作更細緻的處理

## OutputCache
  - name 
  - varyByParam
  - varyByCustom
  - 前台 V1/V2 : memory
  - 前台 webapi : redis

## CDN
  - Edge Location
  - 存放靜態檔
  - 要觀注 Hit Rate
  - 怎麼算錢 ? n tb/month → Hit Rate & Requests Count

## 更新機制
    過期時 先回傳舊資料再更新