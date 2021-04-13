---
title: "[實作筆記] 簡單工廠與工廠方法"
date: 2021/04/13 16:55:49 
tag:
  - 實作筆記
---

## 前情提要

最近被要求介紹一下 Factory Method 這個 Design Pattern，
以前看大話設計模式的時候，這個 Pattern 總會跟 Simple Factory 一起講. 
有了一些工作經驗後，現在回頭來重新看這兩個模式.

## 問題

我們為什麼需這個模式 ? 我們面臨什麼樣子的問題 ?
參考以下程式:

```csharp=
public class Email:INotification
{
    void Send(string message)
    {
        ////Send Mail
    }
}
```

先說明一下這個程式，
在整個系統中依據不同情況中，會發送不同的通知，
比如說:電子郵件(Email)、語音電話(Voice Call)或通訊軟體(Slack)等…

所以我們實際使用的場景可能如下:

```csharp=
public class ProcessNotifyCustomerService()
{
    var notify = new Email();
    notify.from = "system@mail.com";
    notify.to = "customerService@mail.com";

    notify.Send(msg);
    //// ....
}

public class ProcessNotifyDevOps()
{
    var notify = new Email();
    notify.from = "system@mail.com";
    notify.to = "devOps@mail.com";

    notify.Send(msg);
    //// ....
}

public class ProcessNotifyManager()
{
    var notify = new VoiceCall();
    notify.server = "voiceCall.server";
    notify.port = "9527";

    notify.Send(msg);
    //// ....
}

//// more ...

```

以上面的例子來說明一下簡單工廠想解決的問題:  
當我們在整個系統中使用不同的 Notification 時,  
我們建立整個物件的細節都在 Client 端之中,  
這是非常擾人的, 特別當你要建立個複雜的物件,  
你肯定不會希望每次都要重來一遍.
第一個想法就是把細節封裝起來,如下:

```csharp=
public class EmailFactory()
{
    public static Email Create(){
        Email notify = new Email();
        notify.from = "system@mail.com";
        notify.to = "devOps@mail.com";
        return Email;
    }
}
```

再進一步, 當我們有相同的行為時也可以封裝到簡單工廠之中,
在我們的例子中, 我們可以把 Email 與 VoiceCall 放在同一個工廠裡面
在這裡我們命名為 `NotifyFactory`

```csharp=
public class NotifyFactory()
{
    public static INotification Create(string type){
    
        INotification notify;
        switch (type) {
           case "DevOpsEmail":
              notify = new Email();
              notify.from = "system@mail.com";
              notify.to = "devOps@mail.com";
              return notify;
    
           case "CustomerServiceEmail":
              notify = new Email();
              notify.from = "system@mail.com";
              notify.to = "customerService@mail.com";
              return notify;
    
           case "VoiceCall":
              notify = new VoiceCall();
              notify.server = "voiceCall.server";
              notify.port = "9527";
              return notify;
  
           default:
              throw new UnsupportedOperationException("不支援該操作");
       }
    }
}
```

簡單工廠的好處在於，
當你想改變某一個功能時你只需要修改一個點，
而且你在改動的過程入無需再次涉入建構的細節

比如說:發信通知改為播打語音電話

![只需要修改Client，無需處理建構細節](https://i.imgur.com/dRcWMPq.jpg)

而另一個好處是，如果在建立物件的細節有所調整的話，
可以只要在一處就完成所有修正.

比如說: Email Notify 更換為客服的信箱

![只需要改一個地方，就完成所有的修正](https://i.imgur.com/LQIRpPQ.jpg)

## 工廠方法

當我們需要加入(擴展)新的商業邏輯，會修改到不止一個地方  
舉例來說:
我要加入一種新的通知叫飛鴿傳書(PigeonNotify)好了，
除了修改 Client 端使用工廠建立新的 notify 外，
也要在簡單工廠裡面修改。

```csharp=
public class EmailFactory()
{
    public static Email Create(){
        Email notify = new Email();
        notify.from = "system@mail.com";
        notify.to = "devOps@mail.com";
        return Email;
    }
}
```

再進一步, 當我們有相同的行為時也可以封裝到簡單工廠之中,
在我們的例子中, 我們可以把 Email 與 VoiceCall 放在同一個工廠裡面
在這裡我們命名為 `NotifyFactory`

```csharp=
public class NotifyFactory()
{
    public static INotification Create(string type){
    
        INotification notify;
        switch (type) {
           //// 中間省略
           case "Pigeon":
                return new PigeonNotify();
           default:
              throw new UnsupportedOperationException("不支援該操作");
       }
    }
}
```

![面對功能的擴展，也需要修改工廠的邏輯](https://i.imgur.com/uIAAcNL.jpg)

簡單工廠中的 Switch 會與參數耦合，  
每次加入一個新的 Notify 都會異動，  
這違反**開放封閉原則**.  
改用工廠方法, 我們只需要新增一個新的工廠方法,  
並在 Client 呼叫使用工廠, 不再傳遞任何參數.  

![Client 相依於工廠介面上，需要呼叫指定工廠取得物件](https://i.imgur.com/FoMuHwG.jpg)

## TDD to Simple Factory

目的，透過 TDD 建立出不同的通知(Notification)類別，
再透過 TDD 趨動簡單工廠，再透過重構改變為工廠方法。

1. 利用測試作為 Client 寫出測試案例
2. 測試案例先簡單使用 `new` 通過測試
    - 因為是概念性的測試，所以會缺乏實作細節，實務上可能會不只有 `new`  

    到這一步只是一般建立物件，  
    下一步開始是趨動成為簡單工廠，  
    但實際上你是可以跳過簡單工廠，直接 TDD 出工廠方法的  

3. 再寫一個測試案例，來製造(Notify)功能的重複(Email、SNS)
4. 功能重複讓我們可以抽出介面
5. 建立簡單工廠使用邏輯分支回傳不同的(Notify)功能實作
6. 在簡單工廠的邏輯分支使用不同工廠方法實作
   - 因為封裝了實作細節，方法簽章應該不需要任何參數，回傳值應該為 `void`

    下一步開始趨動成為工廠方法

7. 讓 Client 端直接呼叫不同的工廠
   - 簡單工廠類別就會變成多餘無用的類別 
8. 因為有相同的方法簽章，所以可以抽出工廠介面
9. 讓 Client 端相依工廠介面

可以參考以下[我的 commit](https://github.com/marsen/Marsen.NetCore.Dojo/pull/33/commits) 順序:

## 參考

- [一篇搞定工廠模式【簡單工廠、工廠方法模式、抽象工廠模式】](https://www.it145.com/9/55827.html)
- [Factory Method](https://refactoring.guru/design-patterns/factory-method)

(fin)
