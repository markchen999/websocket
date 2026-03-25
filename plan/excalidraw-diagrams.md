# Excalidraw Diagram Creation Guide

## Overview
This document captures lessons learned from creating multiple Excalidraw diagrams via the MCP server for the WebSocket presentation.

## Diagrams Created
1. `assets/http-request-response.png` — HTTP Request-Response (Slide 3)
2. `assets/polling-comparison.png` — Short Polling / Long Polling / SSE comparison (Slide 5)
3. `assets/websocket-bidirectional.png` — WebSocket bidirectional connection (Slide 6)
4. `assets/websocket-handshake.png` — WebSocket handshake flow (Slide 7)

---

## MCP Server Setup
- Two Docker containers: `amazing_goldwasser` (MCP backend) and `mcp-excalidraw-canvas` (frontend)
- The MCP server runs inside Docker, so file exports go to `/app/` inside the container
- Must use `docker cp` to move exported files to the host filesystem

### Export Workflow
```
1. mcp__excalidraw__export_to_image → /app/filename.png
2. docker cp amazing_goldwasser:/app/filename.png → local assets/ path
3. Read the file to visually verify the result
```

---

## Key Lessons

### 1. Vertical Lines — Use Thin Rectangles, Not Lines
**Problem:** The `line` type with `roughness: 1` produces slanted lines because Excalidraw's hand-drawn engine adds horizontal wobble to the points (e.g., `[0,0]` to `[100, 360]` instead of `[0,0]` to `[0, 360]`).

**Even `roughness: 0` does NOT fix this** — the points are already skewed at creation time.

**Solution:** Use a thin filled rectangle instead:
```json
{
  "type": "rectangle",
  "x": 99, "y": 60,
  "width": 2, "height": 310,
  "strokeColor": "#1a1a1a",
  "backgroundColor": "#1a1a1a",
  "strokeWidth": 1,
  "roughness": 0
}
```
This guarantees a perfectly vertical line.

### 2. Arrows — Bound vs Unbound
**Problem:** When using `startElementId` / `endElementId` to bind arrows to shapes, Excalidraw auto-routes them to element edges. If two arrows are bound to the same two elements, they overlap at the same y-position.

**Solution:** For arrows that need specific vertical positioning (e.g., Request arrow above, Response arrow below), do NOT bind them. Use manual coordinates instead:
```json
// Request arrow (upper)
{"type": "arrow", "x": 260, "y": 118, "width": 300, "height": 0, ...}
// Response arrow (lower, 44px below)
{"type": "arrow", "x": 560, "y": 162, "width": -300, "height": 0, ...}
```

### 3. Null Values Cause Errors
**Problem:** Setting `"startArrowhead": null` throws a validation error.

**Solution:** Simply omit the parameter entirely. Only include `startArrowhead` when you need an arrowhead (e.g., `"startArrowhead": "arrow"` for bidirectional arrows).

### 4. Arrow Direction via Negative Width
- Arrows pointing **right**: positive `width` (e.g., `"width": 300`)
- Arrows pointing **left**: negative `width` (e.g., `"width": -300`)
- The arrow starts at `(x, y)` and the width determines direction

### 5. White-Background Labels Over Arrows (Penetrating Effect)
To create the visual effect of arrows passing through labels:
1. Draw the full-width arrow first
2. Place a rectangle with `"backgroundColor": "#ffffff"` on top (later elements have higher z-index)
3. Place the text on top of the rectangle

This creates the illusion that the arrow goes behind the label.

### 6. Export Path Restrictions
- The MCP server only allows exports to `/app/` inside the Docker container
- Any path outside `/app/` triggers: `"Path traversal blocked"`
- Set `EXCALIDRAW_EXPORT_DIR` env var to change the allowed base directory
- Workaround: export to `/app/`, then `docker cp` to host

### 7. Opacity for Semi-Transparent Elements
Use the `opacity` parameter (0–100) on rectangles for color bars or highlights:
```json
{
  "type": "rectangle",
  "backgroundColor": "#4361ee",
  "opacity": 50,
  "roughness": 0
}
```

### 8. Batch Creation Order = Z-Index
Elements created later in `batch_create_elements` appear on top. Plan your element order:
1. Background elements (timelines, color bars)
2. Arrows
3. Label rectangles (with white fill to cover arrows)
4. Text (on top of everything)

---

## Color Palette Used
| Color | Hex | Usage |
|-------|-----|-------|
| Accent Blue | `#4361ee` | Borders, WebSocket elements, titles |
| Text Dark | `#1a1a1a` | Body text, labels |
| Text White | `#ffffff` | Text on filled backgrounds |
| Green | `#22c55e` | Success states, "connection open" |
| Red | `#ef4444` | Error states, "connection closed", arrows |
| Gray | `#999999` | Muted text, "No data" responses |
| Amber | `#f59e0b` | Summary text |
| Light Gray | `#cccccc` | Timeline reference lines |

## Font Settings
- `fontFamily: "virgil"` (or `1`) — hand-drawn style, matches Excalidraw aesthetic
- Title text: `fontSize: 28`
- Body text: `fontSize: 20`
- Small labels: `fontSize: 14–16`
- `roughness: 1` for hand-drawn elements, `roughness: 0` for precision elements (lines, color bars)
