# WebSocket & Real-Time Communication — Presentation Plan

## Overview

- **Topic:** WebSocket & Real-Time Communication
- **Duration:** 30–45 minutes
- **Audience:** 3–5 people (1 junior backend dev, 1 PM, others mixed)
- **Language:** All slides in English (simple vocabulary)
- **Goal:** Everyone should walk away understanding (1) what WebSocket is and why we need it, (2) when to use it, (3) Connection Lifecycle basics
- **Format:** HTML/CSS/JS slides + live demos with real connections
- **Tech stack:** Node.js + Express + Socket.IO (server), HTML/JS (slides & demos)

---

## Architecture — Demo Infrastructure

### Server Setup

One local Node.js server running on `localhost:3000`, all demos connect to it. No internet required.

**Endpoints needed:**

| Purpose       | Type      | Endpoint                                                   |
| ------------- | --------- | ---------------------------------------------------------- |
| Short Polling | HTTP GET  | `/api/poll` — returns immediately                          |
| Long Polling  | HTTP GET  | `/api/long-poll` — holds connection until new message      |
| SSE           | HTTP GET  | `/api/sse` — returns `Content-Type: text/event-stream`     |
| WebSocket     | Socket.IO | Handles all WS demos (lifecycle, messaging patterns, chat) |

### Network Panel (Custom)

Rendered directly in the browser (no DevTools needed). Intercept `window.fetch` and `window.WebSocket` / `window.EventSource` to capture real traffic.

**Display columns (simplified):** URL / Response / Header Size

- Emphasize header size waste (core motivation from RFC 6455)
- Add cumulative counter: Total Requests + Total Header Bytes

### Live Chat (Slide 19)

- Audience connects via local Wi-Fi using device IP: `192.168.x.x:3000/chat`
- Display QR code on slide for easy access

---

## Slide-by-Slide Plan

### Slide 1 — Cover

**Type:** Static

```
Title: WebSocket & Real-Time Communication
Subtitle: How the web talks in real time
[Name] / [Date]
```

---

### Slide 2 — Opening Question

**Type:** Static | **Time:** ~2 min

```
When you send a message on LINE or Slack,
how does it reach the other person instantly?
```

- Visual: Chat bubble icons, one side "sent", one side "received", big question mark in between
- Verbal bridge (in Chinese): 「今天的分享就是要回答這個問題。」

---

### Slide 3 — HTTP Request-Response Model

**Type:** Diagram | **Time:** ~3 min

```
How HTTP Works

The client sends a request, the server sends back a response.
Then the connection is done.

Think of it like going to a help desk — you ask a question,
get an answer, and walk away. Every new question means waiting in line again.
```

- Visual: Client ←→ Server with Request arrow (→) and Response arrow (←), then ✕ showing connection closed
- Key point to emphasize verbally: HTTP was not designed for real-time

---

### Slide 4 — What if we want real-time with HTTP?

**Type:** Card diagram | **Time:** ~3 min

Three card columns using large → ← text arrows at `--body-size`. No URL details (that's Demo's job). Visual rhythm difference is the teaching tool:

**Short Polling:** 5 rows of `→ No data` + final `→ New message!` — looks "busy". Summary: "Many requests, mostly wasted"

**Long Polling:** request → ⏳ waiting → response rhythm with large gaps — looks "slow". Summary: "Fewer, but held open"

**SSE:** one `→ Connect`, three `← Server event`, then `✗ Client can't send` in red — looks "one-directional". Summary: "One-way only"

Verbal bridge: These all work, but none of them are ideal. That's why WebSocket was created.

---

### Slide 5 — Demo: Short Polling / Long Polling / SSE

**Type:** Live Demo | **Time:** ~3 min

**Layout:** Three columns side by side

- Top: Shared message input + "Send" button (triggers all three simultaneously)
- Each column: Method label + Network panel (URL / Response / Header Size) + counter bar (Reqs / Headers)

**Demo flow:**

1. Open page, let it idle for ~10 seconds
2. Short Polling column: requests accumulate rapidly, all responses empty, header bytes pile up
3. Long Polling column: one request sits in "pending" state
4. SSE column: one connection established, quiet
5. Press Send — all three receive the message, but through visibly different mechanisms

---

### Slide 6 — What is WebSocket

**Type:** Diagram | **Time:** ~2 min

```
WebSocket

A protocol that creates a persistent, two-way connection
between client and server.

Once connected, both sides can send messages to each other
at any time — no need to wait for a request.
```

- Visual: Same Client-Server format as Slide 4, but with bidirectional arrows and connection never closes
- Contrast with Slide 4's diagrams — this one is clean and simple
- Verbal analogy: The methods we just saw are like sending letters back and forth. WebSocket is like making a phone call — once the call is connected, both sides just talk freely.

---

### Slide 7 — WebSocket Handshake (HTTP Upgrade)

**Type:** Diagram | **Time:** ~3 min

```
How WebSocket Connects

It starts as a normal HTTP request.
The client says "Can we upgrade to WebSocket?" and the server says "Yes."

After that, the connection switches from HTTP to WebSocket.

After the upgrade, messages only need a very small frame header
— as little as 2 bytes.
```

- Visual: Two-step flow
  - Step 1: Client → Server HTTP Request with simplified header `Upgrade: websocket`
  - Step 2: Server → Client HTTP Response `101 Switching Protocols`
  - Arrow down → "WebSocket connection is now open"
- The "2 bytes" line creates suspense for the next Demo — audience will look for it
- Do NOT mention ws:// vs wss:// here — save for security slide

---

### Slide 8 — The Cost of One HTTP Request

**Type:** Diagram | **Time:** ~2 min

```
The Cost of One HTTP Request

Every HTTP request carries overhead — even when the server has nothing new to say.

TCP handshake:  ~1 round trip
HTTP Headers:   200-800 bytes (cookies, auth, content-type...)
Body:           often 0 bytes ("no new messages")

When polling every 2 seconds:
- 30 requests per minute
- ~12KB of headers per minute — just to ask "anything new?"

WebSocket: 1 handshake, then each message = 2-6 bytes overhead
```

- Visual: Left-right comparison — fat amber HTTP request (TCP handshake + headers + empty body) vs slim blue WebSocket frame (2-6B header + payload)
- Purpose: Build intuition before the Polling vs WebSocket demo

---

### Slide 9 — Demo: HTTP Polling vs WebSocket (with cumulative counter)

**Type:** Live Demo (KEY DEMO) | **Time:** ~4 min

**Layout:** Left-right split

- Left: **HTTP Short Polling** — Network panel + cumulative counter (Requests / Wasted / Header Bytes)
- Right: **WebSocket** — Network panel + cumulative counter (Frames / Overhead)
- "Wasted Requests" = empty poll responses (status 204), shown in red
- Top: Shared message input + Send button

**Demo flow:**

1. Open page, do nothing for ~10 seconds
2. Left side (Polling): 5-6 requests accumulated, several KB of headers, all responses empty
3. Right side (WebSocket): one line "Connection established", counters barely moved
4. Press Send — left side waits for next poll cycle (visible delay); right side appears instantly
5. Send a few more messages — let the counter gap widen
6. Verbal: "Same messages, same result, but look at the cost."

---

### Slide 10 — Connection Lifecycle Overview

**Type:** Diagram | **Time:** ~3 min

```
Connection Lifecycle

A WebSocket connection goes through four stages:

Opening → Open → Closing → Reconnecting
```

- Visual: Horizontal flowchart, four boxes connected by arrows
  - From "Closing": fork into two paths
    - "Normal close → Done"
    - "Unexpected close → Reconnecting → back to Opening"
- Each stage with one-line description:
  - **Opening** — The client and server do the HTTP upgrade handshake
  - **Open** — Both sides can send and receive messages freely
  - **Closing** — One side says "I'm done" and the connection ends gracefully
  - **Reconnecting** — If the connection drops unexpectedly, the client tries to connect again

---

### Slide 11 — Heartbeat (Ping/Pong)

**Type:** Diagram | **Time:** ~2 min

```
Keeping the Connection Alive

How do you know the other side is still there?

The server sends a Ping, the client replies with a Pong.
If no Pong comes back, the server knows the client is gone.

Without this, the connection might be silently dropped
by a proxy or firewall — and neither side would know.
```

- Visual: Server ←→ Client with 2-3 Ping → Pong pairs, then last Ping gets ✕ on Client side, labeled `Connection lost`
- Verbal analogy: It's like when you're on a phone call and it goes quiet — you say "Hello? Are you still there?" That's what Ping does.

---

### Slide 12 — Disconnection & Reconnection

**Type:** Diagram | **Time:** ~2 min

```
When the Connection Drops

There are two kinds of disconnection:

Normal close — Both sides agree to end the connection.
Nothing to worry about.

Unexpected close — The network fails, the server crashes,
or a timeout happens. The client needs to reconnect.

Reconnection strategy: Don't retry immediately.
Wait 1 second, then 2, then 4, then 8...
This prevents all clients from hitting the server at the same time.
```

- Visual: Two blocks — "Normal close" and "Unexpected close"
  - Unexpected close block: timeline showing 1s → 2s → 4s → 8s with increasing gaps
- No close code numbers, no "exponential backoff" terminology — just behavior description

---

### Slide 13 — Demo: Lifecycle Visualization

**Type:** Live Demo | **Time:** ~3 min

**Implementation:** Real Socket.IO connection to localhost:3000 (not simulated)

**Layout:**

- Top: Large connection status indicator (green "Connected" / yellow "Reconnecting" / red "Disconnected")
- Middle: Event log area (timestamped entries)
- Bottom: One button — **"Kill Connection"**

**How it works:**

- Slide entry → `io()` connects with `reconnection: true`, backoff 1s–8s
- Heartbeat: `socket.emit('ping-server')` every 3s, server replies `pong-server`
- Kill button: `socket.emit('force disconnect')` → server calls `socket.disconnect(true)`
- Socket.IO built-in reconnection fires automatically with backoff

**Demo flow:**

1. Show the green status and ping/pong log running (real network traffic)
2. Say: "I'm going to break the connection on purpose. Watch what happens."
3. Press "Kill Connection"
4. Light turns red → log shows reconnection attempts with increasing intervals → light turns green again
5. Everything is real — no animation, no setTimeout

---

### Slide 14 — Security: wss:// and Authentication

**Type:** Diagram | **Time:** ~2 min

```
Security

ws:// is unencrypted. wss:// is encrypted.
Same idea as http:// vs https://.

Authentication happens during the first HTTP handshake — the client
sends a token in the headers. After the upgrade, the connection stays
open and the token is not sent again.

This means: if the token expires while the connection is open,
the server must actively close the connection and ask the client
to reconnect with a new token.
```

**Visual for ws:// vs wss://:**

Left-right split:

- Left (`ws://`): Shows message in plaintext — `{"user": "Alice", "message": "Hi!", "token": "abc123"}` with label "Anyone can read this"
- Right (`wss://`): Shows hex bytes — `17 03 03 00 9a 4b 2d 8f c3 a1 07 e2 91 3c d4 7f ...` with label "Encrypted"

> **Note:** Plan A is to use real Wireshark screenshots (capture ws:// and wss:// traffic). Plan B is to build the visualization directly into the demo. Screenshots can be added later.

**Visual for authentication:**

- Timeline: Handshake phase marked "Token sent here" → long line representing open connection → warning symbol at end "Token expired — server kicks client"

Verbal (to PM): "This is something to think about when designing the product — how do you handle a user whose login session expires while they're still connected?"

---

### Slide 15 — Message Patterns: Broadcast / Room / One-to-One

**Type:** Diagram | **Time:** ~3 min

```
Message Patterns

Broadcast — Send to everyone connected.
Like an announcement on a speaker — everyone in the building hears it.

Room — Send to a specific group.
Like a group chat — only people in that group receive the message.

One-to-One — Send to one specific person.
Like a private message — only you and that person can see it.
```

- Visual: Three groups of 6 user icons, same layout, different "lit up" patterns:
  - Broadcast: all 6 lit
  - Room: 3 lit (same-colored border), 3 dim
  - One-to-One: 1 lit, 5 dim

Verbal (to PM): "When you're designing a product, the first question is — when a user sends a message, who should receive it? That decides which pattern you use."

---

### Slide 16 — Demo: Message Patterns Interactive

**Type:** Live Demo | **Time:** ~3 min

**Layout:**

- Top: Dropdown (Broadcast / Room / One-to-One) + message input + Send button
- Bottom: 6 user cards with names, each labeled Room A or Room B
- Card fonts, avatars, room labels, and message bubbles enlarged for projection readability

**Implementation:** 6 user cards with flash animation. Server uses Socket.IO rooms.

**Demo flow:**

1. Select **Broadcast** → Send → all 6 cards flash/animate, show message
2. Select **Room** → pick Room A → Send → only Room A cards light up, Room B stays quiet
3. Select **One-to-One** → pick specific user → Send → only that one card lights up
4. Optional: show a simplified log of which socket IDs received the message (for technical audience)

---

### Slide 17 — WebSocket in the Real World

**Type:** Static | **Time:** ~2 min

```
WebSocket in the Real World

Chat apps — Slack, Discord, LINE
Collaborative editing — Google Docs, Figma
Live data — Stock tickers, sports scores
Gaming — Multiplayer online games
Notifications — Real-time push alerts
```

- Visual: Each category with recognizable product logo/icon
- Verbal example: "When you see someone else's cursor moving in Google Docs, that's WebSocket pushing the position data to your browser in real time."

---

### Slide 18 — When to Use / Not Use WebSocket

**Type:** Static | **Time:** ~2 min

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

- Visual: Green section (use) / Red section (don't use)
- The closing line is the key takeaway for PM

---

### Slide 19 — Live Demo: Chat Room

**Type:** Live Demo (Interactive) | **Time:** ~5 min

- Display QR code linking to `192.168.x.x:3000/chat`
- Audience scans, enters nickname, starts chatting
- Presenter's screen shows all messages in real time
- If time allows: split audience into different rooms, demonstrate Room pattern live
- Event name: `chat message` (with space, matching server.js and Socket.IO tutorial convention)

---

### Slide 20 — Summary

**Type:** Static | **Time:** ~1 min

```
What We Learned Today

HTTP is request-response. It wasn't designed for real-time.

WebSocket gives us a persistent, two-way connection with very low overhead.

A connection has a lifecycle — open it, keep it alive,
handle disconnections gracefully.

Choose the right message pattern for your use case
— broadcast, room, or one-to-one.
```

---

### Slide 21 — Q&A

**Type:** Static

```
Questions?
```

---

## Time Budget

| Slide | Content                              | Type      | Time        |
| ----- | ------------------------------------ | --------- | ----------- |
| 1     | Cover                                | Static    | —           |
| 2     | Opening question                     | Static    | 2 min       |
| 3     | HTTP model                           | Diagram   | 3 min       |
| 4     | Polling / Long Polling / SSE         | Diagram   | 3 min       |
| 5     | Demo: three methods compared         | Demo      | 3 min       |
| 6     | What is WebSocket                    | Diagram   | 2 min       |
| 7     | WebSocket handshake                  | Diagram   | 3 min       |
| 8     | HTTP request cost breakdown          | Diagram   | 2 min       |
| 9     | Demo: Polling vs WebSocket + counter | Demo      | 4 min       |
| 10    | Connection Lifecycle overview        | Diagram   | 3 min       |
| 11    | Heartbeat Ping/Pong                  | Diagram   | 2 min       |
| 12    | Disconnection & Reconnection         | Diagram   | 2 min       |
| 13    | Demo: Lifecycle visualization        | Demo      | 3 min       |
| 14    | Security: wss:// + auth              | Diagram   | 2 min       |
| 15    | Message patterns                     | Diagram   | 3 min       |
| 16    | Demo: Message patterns interactive   | Demo      | 3 min       |
| 17    | Real-world examples                  | Static    | 2 min       |
| 18    | When to use / not use                | Static    | 2 min       |
| 19    | Live demo: chat room                 | Demo      | 5 min       |
| 20    | Summary                              | Static    | 1 min       |
| 21    | Q&A                                  | Static    | Remaining   |
|       |                                      | **Total** | **~40 min** |

---

## Recommended Reading

### Must Read (Concepts)

- **RFC 6455** — Focus on Section 1 (Overview) and Section 4 (Opening Handshake)
- **MDN Web Docs — WebSocket** — Covers API, handshake, close codes clearly
- **High Performance Browser Networking (Ilya Grigorik)** — Free at hpbn.co, has a full WebSocket chapter. Good narrative structure for presentation reference

### Recommended (Deeper Understanding)

- **Socket.IO docs** — Especially "why not raw WebSocket" section; covers reconnection, rooms, namespaces
- **Ably / PieSocket blog posts** — "WebSockets vs Long Polling" comparisons with diagrams

### Hands-On Prep

- Socket.IO "Get Started" tutorial (builds a chat room — matches Slide 19 Demo)
- Wireshark basics (for ws:// vs wss:// screenshots on Slide 14)

---

## Key Concepts to Master

- HTTP keep-alive vs WebSocket persistent connection — different things
- WebSocket handshake is an HTTP Upgrade, not a separate protocol initiation
- Ping/Pong heartbeat: why it exists (proxies/NAT can silently kill idle connections)
- Reconnection with backoff: prevents thundering herd problem
- wss:// encryption + token authentication during handshake only
- Frame header overhead: HTTP headers = hundreds of bytes per request; WebSocket frame = as little as 2 bytes

---

## Open Items

- [ ] Wireshark screenshots for Slide 14 (ws:// vs wss:// comparison) — backup: build into demo
- [ ] Confirm local network IP for Slide 19 QR code
- [x] Build server (Express + Socket.IO, ~100 lines)
- [x] Build slides using frontend-slides skill in Claude Code
- [x] CSS optimized for laptop/projection (mobile breakpoints removed, fonts enlarged)
