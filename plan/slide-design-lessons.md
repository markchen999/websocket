# Slide Design Lessons

## Overview
Lessons learned from iterating on the HTML presentation slides, particularly around layout, whitespace, and visual hierarchy.

---

## Problem: Too Much Whitespace in Slides

### Root Cause
The default `.slide-content` used `justify-content: center`, which vertically centered content and left large gaps above and below when content was short.

### Solution Applied
```css
.slide-content {
    justify-content: flex-start;
    padding-top: clamp(3rem, 8vh, 6rem);
}
```
Content starts from top with controlled padding instead of floating in the middle.

---

## Problem: Comparison Cards Too Small

### Iterations
1. **Card layout** (side-by-side boxes) — Cards looked tiny on full-screen slides, ~30% of viewport
2. **Flex-grow approach** — Tried making cards fill remaining space with `flex:1`. Left card stretched absurdly tall while right card stayed tiny. Looked broken.
3. **Horizontal bar chart** (final solution) — Replaced cards entirely with stacked horizontal bars spanning full width

### Final Approach: Stacked Bar Chart
Instead of two side-by-side cards, use full-width horizontal bars where **width = bytes**:

```
HTTP Request:  [TCP Handshake 25%][Headers: 200-800 bytes  ~67%][Body 8%]
WebSocket:     [Header: 2-6 bytes][Payload]  (tiny compared to HTTP)
```

**Why this works:**
- The width ratio itself tells the story — no need for extra explanation
- Full `width: 100%` eliminates left/right whitespace
- Bar height `clamp(6rem, 16vh, 10rem)` makes text readable at presentation distance
- Text inside bars uses `var(--h3-size)` — presentation-grade font size

### Key Design Decisions
- **HTTP bar spans 100% width** — it represents the full overhead
- **WebSocket bar is intentionally short** — the visual gap IS the message
- **Text lives inside the bars** — no separate labels needed
- Bottom summary is one line, not paragraphs

---

## Typography Rules for Slides

### Font Size Hierarchy
| Element | Variable | Actual Size |
|---------|----------|-------------|
| Slide title | `--h2-size` | `clamp(1.8rem, 4vw, 3rem)` |
| Bar text / section headers | `--h3-size` | `clamp(1.3rem, 3vw, 2rem)` |
| Labels above bars | `--body-size` | `clamp(1rem, 1.8vw, 1.4rem)` |
| Annotations below bars | `--small-size` | `clamp(0.8rem, 1.2vw, 1rem)` |

### Lesson: "We Have Space"
On a full-screen slide, small text is useless. When in doubt:
- Use `--h3-size` for anything that needs to be read from a distance
- Use `line-height: 1.6–1.8` for multi-line text inside bars
- Add separators (colon, dash) between text segments that would otherwise run together
- Keep `font-weight` consistent within a bar segment — mixing bold/normal in small spaces creates visual noise

---

## Layout Patterns That Work

### Full-Width Bar Chart
```html
<div style="display:flex;width:100%;height:clamp(6rem,16vh,10rem);border-radius:8px;overflow:hidden;">
    <div style="flex:0 0 25%;background:...;">Segment A</div>
    <div style="flex:1;background:...;">Segment B (fills rest)</div>
    <div style="flex:0 0 8%;background:...;">Segment C</div>
</div>
```

### Short Bar (for contrast)
```html
<div style="display:flex;height:clamp(6rem,16vh,10rem);">
    <div style="width:clamp(200px,20vw,320px);...;">Small segment</div>
    <div style="width:clamp(160px,18vw,280px);...;">Another small segment</div>
</div>
```
The fact that it doesn't fill the width IS the visual point.

---

## Global CSS Changes Made

| Property | Before | After | Why |
|----------|--------|-------|-----|
| `--content-gap` | `clamp(1rem, 2.5vw, 2.5rem)` | `clamp(1.2rem, 3vw, 3rem)` | More breathing room |
| `.slide-content justify-content` | `center` | `flex-start` | Eliminate bottom whitespace |
| `.slide-content padding-top` | (none) | `clamp(3rem, 8vh, 6rem)` | Controlled top spacing |
| `.card, .container max-width` | `min(90vw, 1000px)` | `min(95vw, 1200px)` | Wider content area |
| `img max-height` | `min(50vh, 400px)` | `min(65vh, 550px)` | Larger images |
| `.diagram, .flow, .compare-blocks` | (no max-width) | `max-width: 90vw` | Diagrams use full width |

---

## Anti-Patterns to Avoid

1. **Don't use `flex:1` to stretch cards vertically** — It works for equal-height columns but looks absurd when one card has 3x more content than the other
2. **Don't put paragraphs on slides** — If you need a sentence to explain the visual, the visual isn't working
3. **Don't constrain width with `max-width: 220px`** on elements inside full-screen slides — use viewport units or percentages
4. **Don't mix font-weight within tight spaces** — bold title + normal subtitle in a 3rem-tall bar is unreadable; keep it consistent
5. **Don't use `line` type in Excalidraw for straight lines** — use thin rectangles with `roughness: 0`
