# Demo Real Data Lessons

## Overview
2026-03-26 session: fixing Slide 5 (Three Methods Compared) and Slide 8 (Polling vs WebSocket) demos to use entirely real data instead of hardcoded/estimated values. Also added Slide 18 (The Cost of WebSocket).

---

## Principle: Demo Data Must Be Real

Presentation demos lose credibility the moment an audience member opens DevTools and sees the numbers don't match. Every value shown on screen should come from an actual API response, not a JS constant.

### What Was Hardcoded (Before)
| Item | Hardcoded Value | Problem |
|------|----------------|---------|
| HTTP header size | `350` bytes | Actual headers vary per response |
| Short Polling status | `"204"` when empty | Server always returns 200; 204 was faked in frontend |
| WebSocket handshake | `"~180B"` | Tilde prefix admits it's a guess |
| SSE initial bytes | `350` | Same fake header estimate |

### How to Get Real Response Size from Fetch API
```javascript
// Add to base DemoController class
getHeaderSize(r) {
  let s = `HTTP/1.1 ${r.status} ${r.statusText}\r\n`.length;
  r.headers.forEach((v, k) => { s += k.length + v.length + 4; }); // "key: value\r\n"
  s += 2; // trailing \r\n
  return s;
}

// Usage in fetch handler
fetch(url).then(async (r) => {
  const headerSize = this.getHeaderSize(r);
  const text = await r.text();          // read as text first
  const bodySize = text.length;
  const totalSize = headerSize + bodySize;
  const data = JSON.parse(text);        // then parse
  // display totalSize, r.status, etc.
});
```

**Note:** Must use `r.text()` then `JSON.parse()` — can't call both `r.json()` and `r.text()` on the same Response (body is consumed once).

**Limitation:** EventSource (SSE) doesn't expose response headers, so initial connection size can't be measured. Set to 0 and only count actual event data.

---

## Pattern: `?since=` for Incremental Polling

### Problem
`/api/poll` returned ALL accumulated messages every time. This caused:
- First request much larger than subsequent ones (confusing to audience)
- Response payload growing over time (unrealistic)

### Solution
```javascript
// Server
app.get("/api/poll", (req, res) => {
  const since = req.query.since !== undefined ? parseInt(req.query.since) : null;
  const filtered = since !== null ? messages.filter((m) => m.time > since) : messages;
  res.json({ messages: filtered });
});

// Client
this.spSince = Date.now();  // start from now, ignore old messages
fetch(BASE + '/api/poll?since=' + this.spSince)
```

### JavaScript Gotcha: `0` is Falsy
Original code: `const since = parseInt(req.query.since) || 0` then `since ? filter : all`

When `since=0`, `parseInt("0")` returns `0`, which is falsy, so it fell through to returning all messages. Fix: explicitly check `!== undefined` and `!== null`.

---

## CORS: Live Server vs Same-Origin

Opening the HTML via VS Code Live Server (e.g. `http://127.0.0.1:5500`) while API runs on `http://localhost:3000` causes all fetch calls to fail with CORS errors, showing `ERR` in the demo.

**Solution:** Always open the presentation via `http://localhost:3000/websocket-presentation.html` since Express already serves static files with `express.static(__dirname)`.

---

## Presentation-Specific Patterns

### Font Sizes for Projection Demos
Demo areas need larger fonts than regular web UIs. Adjusted values:

| Element | Before | After |
|---------|--------|-------|
| Column label (SHORT POLLING, etc.) | `clamp(0.8rem, 1.2vw, 1rem)` | `clamp(1.1rem, 2vw, 1.6rem)` |
| Network log rows | `clamp(0.75rem, 1.1vw, 0.95rem)` | `clamp(1rem, 1.6vw, 1.35rem)` |
| Counter bar | `clamp(0.8rem, 1.2vw, 1rem)` | `clamp(1rem, 1.6vw, 1.35rem)` |

### Wasted Counter: Making Waste Visible
A "Reqs: 19" counter doesn't tell the audience how many were pointless. Adding a red **Wasted: 18** counter next to it creates immediate understanding:
```html
<span class="label" style="color:#ef4444">Wasted:</span>
<span class="value" id="demo5-sp-wasted" style="color:#ef4444">0</span>
```
Red color draws the eye and creates contrast against the default accent-colored values.

### Real-World HTTP Status Codes
Most polling APIs return **200 with an empty array** when there's no new data, not 204. Reason: 204 means "No Content" (no response body), which breaks `response.json()`. Showing the real 200 is more honest — use the Wasted counter or row highlighting to differentiate empty vs data responses.

### Disable Dev Features Before Presenting
Inline edit (pencil icon) is useful during preparation but confusing during a live talk. Comment out both the HTML elements and the JS instantiation:
```html
<!-- <div class="edit-hotzone" id="editHotzone"></div> -->
```
```javascript
// const editor = new InlineEditor();
```

---

## New Slide Creation Checklist

When adding a new slide to this presentation:

1. **HTML section** — Use `class="slide slide--light"` or `slide--dark` depending on content type
2. **Section number** — Increment from previous: `<div class="section-num reveal">14 — TRADE-OFFS</div>`
3. **Reveal animation** — Add `reveal` class to each content block; use `transition-delay` for stagger
4. **Nav dots** — Auto-generated from `section.slide` count, no manual update needed
5. **Slide ID** — Use descriptive ID like `id="slide-cost"` rather than just numbers (numbers shift when slides are inserted)
6. **Grid layout for multiple items** — 2x2 grid works well for 4 equal-weight points:
   ```css
   display: grid;
   grid-template-columns: 1fr 1fr;
   gap: clamp(0.8rem, 1.5vw, 1.2rem);
   ```

---

## `formatBytes` Helper

Consistent size display across all demos:
```javascript
formatBytes(b) {
  return b < 1024 ? b + "B" : (b / 1024).toFixed(1) + "KB";
}
```
Use this everywhere instead of manual string concatenation (`frameSize + 'B'`) or `.toLocaleString()` to keep units consistent.
