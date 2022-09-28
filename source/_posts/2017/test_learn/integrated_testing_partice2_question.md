---
title: " [實作筆記] 單元測試與重構記錄(二) 發問篇"
date: 2017/12/08 19:04:42
tag:
  - Unit Testing
  - Testing
---

## Q1 Controller 要測試嗎？

### Logics in controller

```csharp
[Route("Member/Get/{Id}")]
public JsonResult GetMemberList(long Id, string cc = "f")
{
    var cleanCache = false;
    //// logics here
    if (this.IsFromCompany() && cc == "t")
    {
        //// do something ...
    }

    try
    {
        var memberList = this.memberService.GetMemberList(Id, cleanCache);
        //// logics here
        if (memberList.Any())
        {
            //// do something ...
        }
        else
        {
            //// do something ...
        }

        return this.Json(result, JsonRequestBehavior.AllowGet);
    }
    catch (Exception ex)
    {
        //// logics here
        //// do something ...
    }
}
```

#### 自問自答

我認為要，
但是對於 WebAPI 回傳的`JsonResult`或是`ActionResult`  
需要轉形才能作驗証  
可以考慮整合測試勝於單元測試,
Controller 的通常是面對 Client Side 的呼叫.

## Q2 當 Controller 只有取資料的邏輯

### No Logics in Controller

```csharp
public ActionResult Index()
{
    return this.Service.GetIndex();
}
```

## Q3 當 Service 只有取資料的邏輯

### No Logics in Service

```csharp
public Member Get(long id)
{
    return this.DataAccessor.GetMember(id);
}
```

#### Q3.自問自答

我認為不要,
要測試商業邏輯,不要在意覆蓋率

## Q4. 當 Service 只有取 Catch 資料的邏輯

### No Logics in Service , just call another service

```csharp
public Member Get(long id)
{
    var enableCache = true;
    var result = this.CacheService.GetCacheData(
        cacheKey,
        () => {
            return this.DataAccessor.GetMember(id);
        },
        enableCache
    );
}
```

#### Q4.自問自答

同上,仍然不需要,
要測試商業邏輯,不要在意覆蓋率,  
要注意的或許是`CacheService.GetCacheData`是不是有包測試 ?
一般來說,Cache 的功能很泛用,測試的報酬率很高

## Q5. 承上,當邏輯存在 Func 參數之中？

### Logics in Func

```csharp
public Member Get(long id)
{
    var enableCache = true;
    var result = this.CacheService.GetCacheData(
        cacheKey,
        () => {
            //// logics here
            if(id > 9487)
            {
                return this.MemberAccessor.GetMember(id);
            }else
            {
                return this.MemberV2Accessor.GetMember(id);
            }

        },
        enableCache
    );
}
```

#### Q5.自問自答

暫時無解,
或許是這樣 Pattern 不適合測試,需要調整架構嗎？
為了測試多包成一個公開方法,反而失去匿名函數的彈性優點,
不在匿名函數內寫邏輯更不合理,待求解答

## Q6.當邏輯在 DA 層或 ORM 的 Query 中要如何測試？

### Logics in ORM

```csharp
上略...
using (var transactionScope = new TransactionScope(TransactionScopeOption.Required, transactionOptions))
{
    using (Entities context = Entities.CreateNew(isReadOnly: true))
    {
        //// logics here
        var query = from a in context.Activies.Valids()
                    where a.Activies_StartDateTime <= startTime &&
                    a.Activies_EndDateTime >= now &&
                    a.Activies_ShopId == shopId &&
                    a.ActiviesCondition.Any(i => i.Activies_ValidFlag
                    && TypeList.Contains(i.Activies_TypeDef))
                    select a;
    }
}
```

#### Q6.自問自答

不適用單元測試,應該整合測試作包覆

## Q7. 當邏輯在 MappingProfile 該如何測試?

### Logics in MappingProfile

```csharp
protected override void Configure()
{
Mapper.CreateMap<PageEntity, UserPageEntity>()
  .ForMember(i => i.Id, s => s.MapFrom(i => i.User_Id))
  .ForMember(i => i.Title, s => s.MapFrom(i => i.User_Name))
  .ForMember(i => i.PageName, s => s.MapFrom(i => i.User_Name + i.User_LastName))
  .ForMember(i => i.LightBox, s => s.MapFrom(i => i.User_Sex == "male" ? true : false));
}
```

#### Q7. 自問自答

要作測試,檢查欄位 Mapping 是否正確,
但實務上若重用性不高,寫 MappingProfile 不如直接在代碼內轉換.
可以少寫 MappingProfile 的測試.

待解答...

(fin)
