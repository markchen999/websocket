# WebSocket & Real-Time Communication — 定稿版 Presentation Plan

## Overview

- **Topic:** WebSocket & Real-Time Communication
- **Duration:** 30–45 minutes
- **Slides:** 21 張
- **Audience:** 3–5 人（1 位初學後端、1 位 PM、其他混合背景）
- **Language:** 投影片全英文（簡單單字），口頭用中文帶
- **Theme:** 靜態頁白色背景、Demo 頁深色背景
- **Diagrams:** Excalidraw 手繪風格 SVG，存在 assets/

---

## Slide-by-Slide Content（定稿）

### Slide 1 — Cover

```
WebSocket & Real-Time Communication
How the web talks in real time
[Name] / [Date]
```

---

### Slide 2 — Agenda

```
Today's Agenda

1  The Problem — HTTP & its limits
2  WebSocket — persistent two-way connections
3  Staying Connected — lifecycle, heartbeat, reconnection
4  Security & Patterns — encryption, auth, message routing
5  Real World — use cases, trade-offs, live demo
```

- 視覺：5 列全寬橫排，左側數字（accent 色）+ 右側標題（--h3-size）
- 口頭：快速帶過每一段，讓觀眾知道接下來的結構

---

### Slide 3 — How HTTP Works

```
How HTTP Works

The client sends a request, the server sends back a response.
Then the connection is done.
```

- 圖：時序圖風格，兩組 Request/Response 各配一小段短色條（代表短暫連線）
- 口頭用中文帶比喻：「就像去櫃台問問題，問完就走，下次要再排隊」
- 不要把比喻寫在 slide 上

---

### Slide 4 — What if we want real-time with HTTP?

```
What if we want real-time with HTTP?
```

- 圖：三欄時序圖（Short Polling / Long Polling / SSE），風格跟 Slide 3 一致
- Short Polling：密密麻麻的 Request → No data，最後一個 → New message!
  - 底部：Many requests, mostly wasted
- Long Polling：一條 request → ⏳ waiting → 回應
  - 底部：Fewer, but held open
- SSE：一條 connect，三條 ← Server event（單向），✗ Client can't send
  - 底部：One-way only
- 口頭：These all work, but none of them are ideal. That's why WebSocket was created.

---

### Slide 5 — Demo: 三種方法對比

**類型：** Live Demo | **時間：** ~3 min

- 三欄：Short Polling / Long Polling / SSE
- 每欄：Network 面板 + 計數器（不要聊天區）
- 上方共用 message input + Send 按鈕
- 操作：空跑 10 秒 → Short Polling 瘋狂刷 → 按 Send → 三邊收到但方式不同

---

### Slide 6 — HTTP vs WebSocket

```
HTTP vs WebSocket

WebSocket is a protocol that creates a persistent, two-way connection.
Once connected, both sides talk freely — no need to wait for a request.

HTTP: Every message needs a new request.
The connection closes after each response.

WebSocket: One handshake to connect.
The connection stays open.
```

- 圖：左右對比時序圖。左邊 HTTP（兩組短色條，反覆斷開重連），右邊 WebSocket（一條長色條，握手一次然後持續）
- 上方文字是 WebSocket 的一句話定義
- 口頭帶比喻：「前面那些方法像寫信，WebSocket 像打電話，接通就自由對話」
- 不要把比喻寫在 slide 上
- 不提 Upgrade header 和 101 Switching Protocols

---

### Slide 7 — The Cost of One HTTP Request

```
The Cost of One HTTP Request

Every HTTP request carries overhead — even when the server
has nothing new to say.

One HTTP request + response: 282 bytes total
One WebSocket message + response: 54 bytes total

When polling every 2 seconds:
- 30 requests per minute
- ~8.4 KB of overhead per minute — just to ask "anything new?"

WebSocket: 1 handshake, then only data.
```

- 視覺：全寬水平長條圖。HTTP bar 撐滿 100% 寬度，WebSocket bar 很短。面積對比。
- 數據來源：Feathers.js benchmark（daffl/websockets-vs-http）
- CSS 版本已做好，不需要 Excalidraw 圖

---

### Slide 8 — Demo: Polling vs WebSocket

**類型：** Live Demo (KEY DEMO) | **時間：** ~4 min

- 左右對比：HTTP Short Polling vs WebSocket
- 每邊：Network 面板 + 計數器（不要聊天區）
- 計數器：Requests / Wasted / Header Bytes（左）vs Frames / Overhead（右）
- 操作：空跑 10 秒 → 左邊累積一堆空 request → 按 Send → 左邊等 poll cycle，右邊即時
- 口頭：Same messages, same result, but look at the cost.

---

### Slide 9 — Connection Lifecycle

```
Connection Lifecycle

A WebSocket connection goes through four stages:

Opening → Open → Closing → Reconnecting
```

- 圖：橫向流程圖，Closing 分岔成 Normal close → Done 和 Unexpected close → Reconnecting → Opening
- 四階段各一句說明：
  - Opening — HTTP upgrade handshake
  - Open — Both sides send freely
  - Closing — One side says "I'm done"
  - Reconnecting — Client retries after drop

---

### Slide 10 — Heartbeat Ping/Pong

```
Keeping the Connection Alive

The server sends a Ping, the client replies with a Pong.
If no Pong comes back, the server knows the client is gone.

Without this, the connection might be silently dropped
by a proxy or firewall — and neither side would know.
```

- 圖：時序圖風格，2-3 組 Ping → Pong 來回，最後一組 Ping 沒回 → ✗ Connection lost
- 口頭帶比喻：「像通話中突然安靜，你會說 Hello? 還在嗎？」
- 不要把比喻寫在 slide 上

---

### Slide 11 — Disconnection & Reconnection

```
When the Connection Drops

Normal close — Both sides agree to end the connection.
Nothing to worry about.

Unexpected close — The network fails, the server crashes,
or a timeout happens. The client needs to reconnect.

Reconnection strategy: Don't retry immediately.
Wait 1 second, then 2, then 4, then 8...
This prevents all clients from hitting the server at the same time.
```

- 圖：兩塊對比。Unexpected close 那邊有 1s → 2s → 4s → 8s 時間軸
- 不提 close code 編號、不提 exponential backoff 術語

---

### Slide 12 — Demo: Lifecycle 視覺化

**類型：** Live Demo | **時間：** ~3 min

- 真實 Socket.IO 連線（不是模擬）
- 上方：狀態燈（綠 Connected / 黃 Reconnecting / 紅 Disconnected）
- 中間：事件 log
- 下方：Kill Connection 按鈕
- 操作：展示 ping/pong log → 按 Kill → 燈變紅 → 重連倒數 → 自動恢復

---

### Slide 13 — Security

```
Security

ws:// is unencrypted. wss:// is encrypted.
Same idea as http:// vs https://.

Authentication happens during the first HTTP handshake —
the client sends a token in the headers.
After the upgrade, the token is not sent again.

If the token expires while the connection is open,
the server must actively close the connection.
```

- 圖：左 ws:// 明文 JSON vs 右 wss:// hex 亂碼
- 口頭對 PM 說：「產品設計要考慮——使用者 session 過期但 WebSocket 還連著怎麼辦？」

---

### Slide 14 — Message Patterns

```
Message Patterns

Broadcast — Send to everyone connected.
Room — Send to a specific group.
One-to-One — Send to one specific person.
```

- 圖：6 個人物圖示 × 3 組。Broadcast 全亮、Room 3 亮 3 暗、One-to-One 1 亮 5 暗
- 口頭對 PM：「設計產品時第一個問題是：誰該收到這則訊息？」

---

### Slide 15 — Demo: 訊息模式互動

**類型：** Live Demo | **時間：** ~3 min

- 上方：模式下拉選單 + message input + Send
- 下方：6 張 user card（Room A × 3、Room B × 3）
- 操作：Broadcast 全亮 → Room A 只亮三個 → One-to-One 只亮一個

---

### Slide 16 — Real World

```
WebSocket in the Real World

Chat apps — Slack, Discord, LINE
Collaborative editing — Google Docs, Figma
Live data — Stock tickers, sports scores
Gaming — Multiplayer online games
Notifications — Real-time push alerts
```

- 口頭舉例：「Google Docs 裡看到別人的游標在動，就是 WebSocket 在推位置資料」

---

### Slide 17 — The Cost of WebSocket

```
The Cost of WebSocket

Server resources
Every connected user holds an open connection.
10,000 users = 10,000 persistent connections on your server.
HTTP serves a request and moves on — WebSocket doesn't.

Scaling is harder
HTTP is stateless — any server can handle any request.
WebSocket is stateful — a client is tied to one server.
Scaling across multiple servers needs extra work.

More things can go wrong
Network drops, proxy timeouts, mobile sleep mode...
You need heartbeat, reconnection logic, and error handling.

Infrastructure cost
Load balancers, firewalls, and proxies must be configured
to support long-lived connections. Not all do by default.
```

---

### Slide 18 — When to Use / Not Use

```
When to Use WebSocket

✓ The server needs to push data to the client without being asked
✓ Both sides need to send messages frequently
✓ Low latency matters

When NOT to Use WebSocket

✗ Simple form submissions
✗ Loading a page or fetching data once
✗ The data doesn't change often

If a normal HTTP request can do the job, use HTTP.
WebSocket adds complexity — only use it when you need it.
```

---

### Slide 19 — Live Demo: 聊天室

**類型：** Live Demo | **時間：** ~5 min

- QR code 連到 192.168.x.x:3000/chat
- 觀眾掃碼進入聊天室
- 如果時間夠，現場分 Room
- 事件名稱：`chat message`（跟 server.js 一致）

---

### Slide 20 — Summary

```
What We Learned Today

HTTP is request-response. It wasn't designed for real-time.

WebSocket gives us a persistent, two-way connection
with very low overhead.

A connection has a lifecycle — open it, keep it alive,
handle disconnections gracefully.

Choose the right message pattern for your use case
— broadcast, room, or one-to-one.
```

---

### Slide 21 — Q&A

```
Questions?
```

---

## Time Budget

| Slide | Content                              | Type      | Time        |
| ----- | ------------------------------------ | --------- | ----------- |
| 1     | Cover                                | Static    | —           |
| 2     | Agenda                               | Static    | 1 min       |
| 3     | HTTP model                           | Diagram   | 3 min       |
| 4     | Polling / Long Polling / SSE         | Diagram   | 3 min       |
| 5     | Demo: three methods compared         | Demo      | 3 min       |
| 6     | HTTP vs WebSocket                    | Diagram   | 3 min       |
| 7     | HTTP request cost breakdown          | Diagram   | 2 min       |
| 8     | Demo: Polling vs WebSocket + counter | Demo      | 4 min       |
| 9     | Connection Lifecycle overview        | Diagram   | 3 min       |
| 10    | Heartbeat Ping/Pong                  | Diagram   | 2 min       |
| 11    | Disconnection & Reconnection         | Diagram   | 2 min       |
| 12    | Demo: Lifecycle visualization        | Demo      | 3 min       |
| 13    | Security: wss:// + auth              | Diagram   | 2 min       |
| 14    | Message patterns                     | Diagram   | 3 min       |
| 15    | Demo: Message patterns interactive   | Demo      | 3 min       |
| 16    | Real-world examples                  | Static    | 2 min       |
| 17    | Cost of WebSocket                    | Static    | 2 min       |
| 18    | When to use / not use                | Static    | 2 min       |
| 19    | Live demo: chat room                 | Demo      | 5 min       |
| 20    | Summary                              | Static    | 1 min       |
| 21    | Q&A                                  | Static    | Remaining   |
|       |                                      | **Total** | **~39 min** |

---

## Excalidraw Diagrams Needed

| Slide | 圖片檔名                       | 描述                                                | 狀態      |
| ----- | ------------------------------ | --------------------------------------------------- | --------- |
| 3     | http-request-response.svg      | 時序圖：兩組 Request/Response + 短色條              | ✅ 已完成 |
| 4     | polling-comparison.svg         | 三欄時序圖：Short Polling / Long Polling / SSE      | ✅ 已完成 |
| 6     | http-vs-websocket.svg          | 左右對比：HTTP 短色條 vs WebSocket 長色條           | ⬜ 待畫   |
| 9     | connection-lifecycle.svg       | 橫向流程圖：Opening → Open → Closing → Reconnecting | ⬜ 待畫   |
| 10    | ping-pong.svg                  | 時序圖：Ping/Pong 來回 + 最後失敗                   | ⬜ 待畫   |
| 11    | disconnection-reconnection.svg | 兩塊對比 + backoff 時間軸                           | ⬜ 待畫   |
| 13    | ws-vs-wss.svg                  | 左右對比：明文 vs 加密                              | ⬜ 待畫   |
| 14    | message-patterns.svg           | 6 人圖示 × 3 組：Broadcast / Room / One-to-One      | ⬜ 待畫   |

**不需要畫圖的 slide：** 7（CSS bar chart）、所有 Demo 頁、所有純文字靜態頁

---

## Q&A 準備（不放進 slide，口頭回答用）

1. **WebSocket 和 HTTP/2 Server Push 的差別？**
   HTTP/2 Server Push 是推「資源」不是推「資料」，而且 Chrome 已移除支援

2. **Socket.IO 和原生 WebSocket 的差異？**
   Socket.IO 加了自動重連、room、namespace、fallback 到 long polling

3. **SSE 什麼時候比 WebSocket 更適合？**
   只需要 server → client 單向推送時，SSE 更簡單且穿透防火牆更容易

4. **水平擴展的挑戰？**
   需要 Sticky Session 或 Redis Pub/Sub 在 server 之間同步

5. **TCP 三次握手具體是什麼？**
   SYN → SYN-ACK → ACK，WebSocket 只需要一次 TCP 握手 + 一次 HTTP Upgrade
