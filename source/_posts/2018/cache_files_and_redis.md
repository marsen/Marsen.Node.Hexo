---
title: "[內部分享]Cache 使用 Files 與 Redis"
date: 2018/09/03 15:45:42
tag:
  - Cache
  - Redis
---
## Why Cache ?
1. 保護稀缺資源(database/disk io/server)
2. 提高反應速度,減少延遲(Latency)

## Cache Point
4. 資料即時性/如何除錯/如何更新
5. Hit Rate
6. High Availability(HA)
7. 單位時間承載量

## Our Cache
1. CDN(Cloudfront)
2. OutputCache
3. Memory Cache
4. Redis Cache
5. File Cache

## More Cache not in above
1. Browser Cache
2. Database Execution Plan Cache (執行計劃快取)
3. Elastic Search


### File Cache
- pro
    - 簡單易用
- con
    - 咬檔
    - 不會自動過期
    - 自已要處理lock機制

### Redis Cache
- pro
    - 會自動過期
    - 有提供Queue的功能
    - 高速(in memory)
- con
    - 單緖(Single Thread)
    - HA的機制費用高
    - Redis 承載量受限於記憶體大小
    - 存儲的東西過大,會導致存取速度下降
    
### 實踐問題:在大流量的站台，Cache 過期時如何避免大量 Request 同時更新同一份 Cache ?

Ans: 使用 Lock ， 
不要使用.NET的`lock` ;.NET Lock 的機制問題在於，Lock 會導致其它 Thread 排隊  
當系統處於高流量的情境底下，會發生間歇性Latency提高，  
而且難以追查問題，也難以重現(一般的壓測無法達到如此高量)。

使用共用的 Lock Key 作判斷標準:
1. Lock Key 存在,回傳 Cache Data
2. Lock Key 不存在,建立 Lock Key(包含 Server Id)
3. 重新取得 Lock Key 如果 
    - Server Id 與 Request 相同，更新 Cache Data 並回傳
    - Server Id 與 Request 不相同，直接回傳 Cache Data

#### [Cache Double check Lock Patter(https://docs.microsoft.com/en-us/previous-versions/office/developer/sharepoint-2010/ee558270(v%3Doffice.14)

```csharp=
private static object _lock =  new object();

public void CacheData()
{
   SPListItemCollection oListItems;
       oListItems = (SPListItemCollection)Cache["ListItemCacheName"];
      if(oListItems == null)
      {
         lock (_lock) 
         {
              // Ensure that the data was not loaded by a concurrent thread 
              // while waiting for lock.
              oListItems = (SPListItemCollection)Cache["ListItemCacheName"];
              if (oListItems == null)
              {
                   oListItems = DoQueryToReturnItems();
                   Cache.Add("ListItemCacheName", oListItems, ..);
              }
         }
     }
}
```



    
## Other
1. 使用多執行緖時要小心處理Exception,不然會導致主程序被中斷  
(File Cache 寫入真實案例,多緖的錯誤不佳會使得IIS無法處理錯誤,進而中斷程序(w3wp.exe))
2. AWS S3 是走 Https Protocal
3. Redis Delete/Backup 會 Lock Thread
4. Redis Command Timeout 極短，所以要注意單一　Cache　資料量大小 (建議資料要小於1k?)

## 參考
- [Redis](https://redis.io/)
- [memcached](https://memcached.org/)
- [Caching Data And Objects](https://docs.microsoft.com/en-us/previous-versions/office/developer/sharepoint-2010/ee558270(v%3Doffice.14)#caching-data-and-objects)

(fin)


