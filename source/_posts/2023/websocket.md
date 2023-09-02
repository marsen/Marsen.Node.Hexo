---
title: " [實作筆記] Websocket 初體驗"
date: 2023/08/07 15:33:16
tags:
  - 實作筆記
---

## 前情提要

tl;dr, websocket 至少是一個 10 年以上的技術了，但是筆者一直沒有機會實作它。  
大部份的時候是應用場景不符合，或是簡單 Long Polling 已經足夠，甚至就算用 Polling 這個方法也無法把 Server 打掛。  
另一個情況是團隊已經很成熟，有專門負責的部門在統一處理這塊邏輯，  
而通常這塊邏輯會與主要的核心功能作切分，所以我也沒有機會接觸到。  
這次難得有個小型的專案，有機會實作，故稍作記錄一下。

## 需求

使用者會停留在某些頁面等待系統的資料狀態更新，大約每 10 秒要作一次 Polling,  
而平均資料狀態更新需要 3~10 分鐘不等。在這個情況下要改用 websocket,  
（註：我不認為這是一個很好的應用場景，但是牽扯更多未揭露的調整故不展開討論,  
EX:Heartbeat、Keep-Alive 等等…機制也未討論到）

## WebSocket Server 實作

使用的技術：Typescript + Express + ServerSocket

```typescript
// import library
import express, { Request, Response } from "express";
import { Server as ServerSocket } from "ws"; // 引用 Server

if (process.env.NODE_ENV !== "production") {
  require("dotenv").config();
}
// form env 指定一個 port
const PORT: number = Number(process.env.WS_PORT);
// 設定斷開時間
const CT: number = 1000 * Number(process.env.CONNECTION_TIMEOUT);
const app = express();
const server = app.listen(PORT, () =>
  console.log(
    `[Server] Listening on http://localhost:${PORT} , timeout is ${CT} ms`
  )
);
const wsServer = new ServerSocket({ server });

// Connection opened
wsServer.on("connection", (ws: WebSocket, req: Request) => {
  // Connection 建立時發生的邏輯

  // Listen for messages from client
  ws.on("message", (data: string) => {
    try {
      // 發送 Message 時發生的邏輯
    } catch (error) {
      console.error("Invalid JSON format: ", data);
    }
    // Get clients who has connected
    const clients = wsServer.clients;
    // Use loop for sending messages to each client
    clients.forEach((client) => {
      client.send(JSON.stringify(docStatus));
    });
  });
  // 設定 ping 時間間隔，用來讓連線太久的 Client 斷開
  const interval = setInterval(() => {
    if (ws.alive) {
      ws.terminate();
      clearInterval(interval);
      return;
    }

    ws.alive = false;
    ws.ping("", false, () => {
      console.log("[Timeout]", ws.key);
    });
  }, CT); // 30 秒

  // Connection closed
  ws.on("close", () => {
    // Connection 關閉時發生的邏輯
    console.log("[Close connected]", ws.key);
  });
});

// 新增一個路由處理器
app.get("/", (req: Request, res: Response) => {
  // health and version check
  res.send(`WebSocket Running Version ${process.env.WS_VERSION}`);
});
```

## 前端實作，收送雙向

使用的技術：JavaScript  
以下內容大多參考於[神 Q 超人](https://medium.com/enjoy-life-enjoy-coding/javascript-websocket-%E8%AE%93%E5%89%8D%E5%BE%8C%E7%AB%AF%E6%B2%92%E6%9C%89%E8%B7%9D%E9%9B%A2-34536c333e1b)的文章，  
作為參考用，實務上會搭配使用的前端框架作修改。

```javascript
var ws;

// 監聽 click 事件
document.querySelector("#connect")?.addEventListener("click", (e) => {
  console.log("[click connect]");
  connect();
});

document.querySelector("#disconnect")?.addEventListener("click", (e) => {
  console.log("[click disconnect]");
  disconnect();
});

document.querySelector("#sendBtn")?.addEventListener("click", (e) => {
  const msg = document.querySelector("#sendMsg");
  sendMessage(msg?.value);
});

function connect() {
  // Create WebSocket connection
  ws = new WebSocket("wss://localhost:8088");
  // 在開啟連線時執行
  ws.onopen = () => {
    // Listen for messages from Server
    ws.onmessage = (event) => {
      // 收到訊息的 Logic
      console.log(`[Message from server]:\n %c${event.data}`, "color: yellow");
    };
  };
}

function disconnect() {
  ws.close();
  // 在關閉連線時執行
  ws.onclose = () => console.log("[close connection]");
}

// 監聽 click 事件
document.querySelector("#sendBtn")?.addEventListener("click", (e) => {
  const msg = document.querySelector("#sendMsg");
  sendMessage(msg?.value);
});

// Listen for messages from Server
function sendMessage(msg) {
  // Send messages to Server
  ws.send(msg);
  console.log("[send message]", msg);
}
```

## 後端實作，只送不收

使用技術: php

```php
<?php
include "./vendor/autoload.php";

$client = new WebSocket\Client("wss://localhost:8088");

$message = json_encode(array(
    "your_property" => "Your Data",
    "complex_property" => array(
        "no" => 1,
        "status" => "Hello word"
    )
));

$client->text($message);
$client->close();
?>
```

## 後記

實務上在股票看盤即時更新、多人聊天室、多人網頁遊戲上或許十分有用，  
但實際上在這此的案例上就有些大材小用了，先當作學習了。  
在查找資料的過程有[一段話](https://www.quora.com/What-is-the-best-language-to-program-a-websocket-server-in)很受用,  
我稍作總結如下：
當討論到 WebSocket 時，不應用 Http 的標準去審視它，  
更應該關注這些 Connection 會持續連接多久? Connection 之間交互的行為是什麼?　　
是運算密集的行為還是讀寫密集的行為？…等等，才是你決策的關鍵。

## 參考

- [WebSocket 与 TCP/IP](https://zhuanlan.zhihu.com/p/27021102)
- [JavaScript | WebSocket 讓前後端沒有距離](https://medium.com/enjoy-life-enjoy-coding/javascript-websocket-%E8%AE%93%E5%89%8D%E5%BE%8C%E7%AB%AF%E6%B2%92%E6%9C%89%E8%B7%9D%E9%9B%A2-34536c333e1b)
- [关于 websocket 到底要不要做心跳](https://zhuanlan.zhihu.com/p/411440557)
- [How To Connect To WebSockets With PHP](https://www.piesocket.com/blog/php-websocket)
- [A JSON event-based convention for WebSockets](https://thoughtbot.com/blog/json-event-based-convention-websockets)
- [Socket.io](https://socket.io/)
- [Pusher.com](https://pusher.com/)

(fin)
