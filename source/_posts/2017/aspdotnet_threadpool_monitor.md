---
title: ASP.NET Thread Pool 監控
date: 2017/01/28 20:46:49
tag:
  - .Net Framework
  - Thread
---

## 前情提要

[上一次](/2016/11/21/aspdotnet_threadpool_and_redis/)說明了 .NET Thread Pool 的機制如何影響 Redis,  
.NET Thread Pool 的機制會貫穿任何與它互動的系統或服務,  
所以這篇會簡單描述如何對 .NET Thread Pool [建立監控](https://msdn.microsoft.com/zh-tw/library/ff650682.aspx)。

## 步驟

1. 建立監控記數器
2. 在系統寫入監控數值
3. 開啟效能計數器

## 建立監控記數器

|記數器|說明|
|---|---|
|Available Worker Threads|目前在 thread-pool 可以使用的 worker threads |
|Available IO Threads|目前在 thread-pool 可以使用的 I/O threads |
|Max Worker Threads| 最大可以建立的 worker threads 數量 |
|Max IO Threads| 最大可以建立的 I/O threads 數量 |

### 建立一個 Console 應用程式

``` csharp
using System;
using System.Diagnostics;

class MyAspNetThreadCounters
{
  [STAThread]
  static void Main(string[] args)
  {
    CreateCounters();
    Console.WriteLine("MyAspNetThreadCounters performance counter category " +
                      "is created. [Press Enter]");
    Console.ReadLine();
  }

  public static void CreateCounters()
  {
    CounterCreationDataCollection col =  
      new CounterCreationDataCollection();

    // Create custom counter objects
    CounterCreationData counter1 = new CounterCreationData();
    counter1.CounterName = "Available Worker Threads";
    counter1.CounterHelp = "The difference between the maximum number " +
                           "of thread pool worker threads and the " +
                           "number currently active.";
    counter1.CounterType = PerformanceCounterType.NumberOfItems32;

    CounterCreationData counter2 = new CounterCreationData();
    counter2.CounterName = "Available IO Threads";
    counter2.CounterHelp = "The difference between the maximum number of " +
                           "thread pool IO threads and the number "+
                           "currently active.";
    counter2.CounterType = PerformanceCounterType.NumberOfItems32;

    CounterCreationData counter3 = new CounterCreationData();
    counter3.CounterName = "Max Worker Threads";
    counter3.CounterHelp = "The number of requests to the thread pool "+
                           "that can be active concurrently. All "+  
                           "requests above that number remain queued until " +
                           "thread pool worker threads become available.";
    counter3.CounterType = PerformanceCounterType.NumberOfItems32;

    CounterCreationData counter4 = new CounterCreationData();
    counter4.CounterName = "Max IO Threads";
    counter4.CounterHelp = "The number of requests to the thread pool " +
                           "that can be active concurrently. All "+  
                           "requests above that number remain queued until " +
                           "thread pool IO threads become available.";
    counter4.CounterType = PerformanceCounterType.NumberOfItems32;

    // Add custom counter objects to CounterCreationDataCollection.
    col.Add(counter1);
    col.Add(counter2);
    col.Add(counter3);
    col.Add(counter4);
    // delete the category if it already exists
    if(PerformanceCounterCategory.Exists("MyAspNetThreadCounters"))
    {
      PerformanceCounterCategory.Delete("MyAspNetThreadCounters");
    }
    // bind the counters to the PerformanceCounterCategory
    PerformanceCounterCategory category =
            PerformanceCounterCategory.Create("MyAspNetThreadCounters",
                                              "", col);
  }
}
```

編譯後並執行即可,  
執行 `regedit` 開啟登錄編輯程式,  
輸入機碼 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services`  
將可以找到剛剛建立的 `MyAspNetThreadCounters` 記數器

## 在系統寫入監控數值

在站台的 `Global.asax` 加入以下的程式碼

```csharp
public bool MonitorThreadPoolEnabled = true;

protected void Application_Start(Object sender, EventArgs e)
{
  Thread t = new Thread(new ThreadStart(RefreshCounters));
  t.Start();
}

public void RefreshCounters()
{
  while (MonitorThreadPoolEnabled)
  {
    ASPNETThreadInfo t = GetThreadInfo();
    ShowPerfCounters(t);
    System.Threading.Thread.Sleep(500);
  }
}

protected void Application_End(Object sender, EventArgs e)
{
  MonitorThreadPoolEnabled = false;
}

public struct ASPNETThreadInfo
{
  public int MaxWorkerThreads;
  public int MaxIOThreads;
  public int MinFreeThreads;
  public int MinLocalRequestFreeThreads;
  public int AvailableWorkerThreads;
  public int AvailableIOThreads;

  public bool Equals(ASPNETThreadInfo other)
  {
    return (
      MaxWorkerThreads == other.MaxWorkerThreads &&
      MaxIOThreads == other.MaxIOThreads &&
      MinFreeThreads == other.MinFreeThreads &&
      MinLocalRequestFreeThreads == other.MinLocalRequestFreeThreads &&
      AvailableWorkerThreads == other.AvailableWorkerThreads &&
      AvailableIOThreads == other.AvailableIOThreads
    );
  }
}

public ASPNETThreadInfo GetThreadInfo()
{
  // use ThreadPool to get the current status
  int availableWorker, availableIO;
  int maxWorker, maxIO;

  ThreadPool.GetAvailableThreads( out availableWorker, out availableIO);
  ThreadPool.GetMaxThreads(out maxWorker, out maxIO);

  ASPNETThreadInfo threadInfo;
  threadInfo.AvailableWorkerThreads = (Int16)availableWorker;
  threadInfo.AvailableIOThreads = (Int16)availableIO;
  threadInfo.MaxWorkerThreads = (Int16)maxWorker;
  threadInfo.MaxIOThreads = (Int16)maxIO;
 // hard code for now; could get this from  machine.config
  threadInfo.MinFreeThreads = 8;
  threadInfo.MinLocalRequestFreeThreads = 4;
  return threadInfo;
}

public void ShowPerfCounters(ASPNETThreadInfo t)
{

  // get an instance of our Available Worker Threads counter
  PerformanceCounter counter1 = new PerformanceCounter();
  counter1.CategoryName = "MyAspNetThreadCounters";
  counter1.CounterName = "Available Worker Threads";
  counter1.ReadOnly = false;

  // set the value of the counter
  counter1.RawValue = t.AvailableWorkerThreads;
  counter1.Close();

  // repeat for other counters

  PerformanceCounter counter2 = new PerformanceCounter();
  counter2.CategoryName = "MyAspNetThreadCounters";
  counter2.CounterName = "Available IO Threads";
  counter2.ReadOnly = false;
  counter2.RawValue = t.AvailableIOThreads;
  counter2.Close();

  PerformanceCounter counter3 = new PerformanceCounter();
  counter3.CategoryName = "MyAspNetThreadCounters";
  counter3.CounterName = "Max Worker Threads";
  counter3.ReadOnly = false;
  counter3.RawValue = t.MaxWorkerThreads;
  counter3.Close();

  PerformanceCounter counter4 = new PerformanceCounter();
  counter4.CategoryName = "MyAspNetThreadCounters";
  counter4.CounterName = "Max IO Threads";
  counter4.ReadOnly = false;
  counter4.RawValue = t.MaxIOThreads;
  counter4.Close();
}
```

當你的站台重新啟動後,  
就會定期將 Worker Thread 與  I/O Threads 的監控數值傳遞給系統。  

### 開啟效能計數器

1. 執行 `perfmon.exe` 開啟效能計數器
2. 新增效能計數器(點選綠色加符號)
3. 選取前面所建立監控記數器就能看到當前 ThreadPool 的使用狀況。
![開啟效能計數器](https://i.imgur.com/HhlbNH2.jpg)

(fin)
