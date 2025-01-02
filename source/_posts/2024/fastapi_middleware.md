---
title: " [實作筆記] FastAPI 的　middleware"
date: 2025/01/03 01:21:26
tags:
  - 實作筆記
---
## 前請提要

最近團隊的小朋友作的系統發生了錯誤。

在追查的過程之中，  
小朋友說了一個驚人的**事實**，  
~~Python 的 FastAPI Web 框架，發生 422 錯誤時（輸入的參數類別錯誤），無法追踪 Request$$~~  
當然這不是事實，那麼 FastAPI 應該怎麼處理呢？

## 本文

讓我們先簡單實作一個 Fast API

```python
from fastapi import FastAPI
from pydantic import BaseModel

# 初始化 FastAPI 應用
app = FastAPI()

# 定義請求資料模型
class Item(BaseModel):
    name: str
    price: float
    description: str = None  # 可選的欄位

# 定義 API 路徑
@app.post("/items/")
async def create_item(item: Item):
    # 假設邏輯檢查
    if item.price <= 0:
        return {"error": "Price must be greater than 0"}
    return {"message": "Item created successfully!", "item": item}
```

接下來，讓我試著用不正確的方式呼叫 API，並且得到一個 422 的錯誤

```bash
curl 
  -X POST "http://127.0.0.1:8000/items/" 
  -H "Content-Type: application/json" 
  -d '{"item": "should be number", "price": 10.5}'
```

FastAPI 會提供 Client 端非常友善的資訊

```json
{
  "detail":[
    {
      "type":"missing",
      "loc":["body","name"],
      "msg":"Field required",
      "input":{
        "item":"should be number",
        "price":10.5
        }
    }]
}
```

但在 Server 端只會有類似以下的資訊，這以後端的角度要排查錯誤會非常不便

```bash
INFO:     127.0.0.1:57617 - "POST /items/ HTTP/1.1" 422 Unprocessable Entity
```

Server Side 有沒有什麼好方法可以協助我們 Debug 呢？

### 解決方法：Middleware

這其實是 web 開發者的小常識才對，
實作如下，

```python
from fastapi import FastAPI,Request
from pydantic import BaseModel
import logging
from datetime import datetime

# 初始化 FastAPI 應用
app = FastAPI()

# 初始化日誌
logging.basicConfig(level=logging.INFO, format="%(asctime)s - %(message)s")

# 定義請求資料模型
class Item(BaseModel):
    name: str
    price: float
    description: str = None  # 可選的欄位

# 中間件，用於記錄請求
@app.middleware("http")
async def log_requests(request: Request, call_next):
    # 記錄請求進來的時間與基本資訊
    request_time = datetime.utcnow()
    logging.info(f"Incoming request at {request_time}")
    logging.info(f"Method: {request.method} | Path: {request.url.path}")
    
    # 嘗試讀取請求 Body
    try:
        body = await request.json()
        logging.info(f"Request Body: {body}")
    except Exception:
        logging.info("No JSON body or failed to parse.")

    # 執行後續處理並取得回應
    response = await call_next(request)

    # 記錄回應狀態碼
    logging.info(f"Response status: {response.status_code}")
    return response

# 定義 API 路徑
@app.post("/items/")
async def create_item(item: Item):
    # 假設邏輯檢查
    if item.price <= 0:
        return {"error": "Price must be greater than 0"}
    return {"message": "Item created successfully!", "item": item}
```

Request 送進來時 Server Side 就會如下記錄

```bash
2025-01-03 03:37:37,294 - Incoming request at 2025-01-02 19:37:37.294509
2025-01-03 03:37:37,294 - Method: POST | Path: /items/
2025-01-03 03:37:37,294 - Request Body: {'item': 'should be number', 'price': 10.5}
2025-01-03 03:37:37,295 - Response status: 422
INFO:     127.0.0.1:59545 - "POST /items/ HTTP/1.1" 422 Unprocessable Entity
```

## 心得

回到我們原本的情境，我們真正要解決的問題是，

｜發生異常時，Server Side 要可以追蹤錯誤

因為是 **FastAPI 所以沒有辦法追蹤錯誤，是非常錯誤判斷**

在 Google 與 AI 工具這麼方便的時代，應該要能正確的提出問題。

ex: FastAPI 可以記錄什麼時候 request 送進來，並帶了什麼資訊嗎？

(fin)
