# Design Iteration Log — Slide 2 & Polling Comparison Diagram

## Slide 2: Opening Question → Agenda

### Claude 原始設計 v1：全寬橫條風格
把 Slide 8（bar chart）的視覺語言直接套用到 Slide 2，三段式橫條（Hey! | ? | Hey!）佔滿頁面寬度。

```
┌──────────┬─────────────────┬──────────┐
│  Hey! 👋 │        ?        │  Hey! 👋 │
│  (blue)  │   (light bg)    │  (grey)  │
└──────────┴─────────────────┴──────────┘
```

**使用者評價：** 「太差勁了，Slide 8 之所以用長條圖是因為它適合。」

**問題：** Bar chart 適合數據對比（Slide 8 的 HTTP vs WebSocket overhead），但不適合開場問題頁。視覺元素應該服務於內容，不是為了風格統一而強套。

---

### Claude 原始設計 v2：Agenda + 左邊框
改為 5 列 Agenda，每列有 `border-left: 4px solid var(--accent)` + 淺藍背景。

**使用者調整：** 移除左邊框（`border-left`），只保留淺藍背景 + 數字 + 標題。

**最終結果（已上線）：**
- 5 列全寬橫排，無邊框
- 左側大數字（accent 色，`--h2-size`），右側標題（`--h3-size`）
- 淺藍背景 `rgba(67, 97, 238, 0.06)` + `border-radius: 8px`
- Section label 改為 `01 — AGENDA`

### 教訓
1. **視覺元素要匹配內容性質** — bar chart 適合數據對比，不適合 agenda/問題頁
2. **簡潔優先** — 左邊框在 agenda 列表中是多餘的裝飾，移除後更乾淨
3. **不要盲目複製成功頁面的風格** — Slide 8 的成功來自 bar chart 恰好適合表達 overhead 對比，不代表所有頁面都該用 bar chart

---

## Polling Comparison Diagram (Excalidraw SVG)

### Claude 原始設計 v1：簡易箭頭風格
模仿原始 PNG 的風格，用獨立箭頭 + 文字標籤，沒有 Client/Server 方塊和垂直時間線。

```
Short Polling       Long Polling        SSE
──────→             ──────→             Connect →
← No data           ⏳ waiting...      ← Server event
──────→             ← New message!      ← Server event
← No data           ──────→            ← Server event
...                  ⏳ waiting...      ✗ Client can't send
```

**問題：** 風格與 Slide 3 的 HTTP 時序圖不一致。

---

### Claude 原始設計 v2：時序圖風格（最終版）
改為跟 Slide 3 一樣的時序圖結構：

- 三欄各有 Client/Server 圓角矩形（邊框 #4361ee）
- 垂直生命線（#1a1a1a）
- 箭頭 #4361ee，標籤用白底方框包住
- 顏色系統：No data=#999999, New message!=#22c55e, 總結=#f59e0b, 警告=#ef4444

### 使用者調整 vs Claude 設計（Excalidraw 元素對比）

元素總數不變：78 個（text=32, rectangle=23, line=6, arrow=17）

位置微調（使用者手動移動）：

| 元素 | Claude 設計 | 使用者調整 | 差異 |
|------|-------------|-----------|------|
| Col1 Client rect | x=0 | x=-10 | -10px |
| Col1 Client line | x=65 | x=49 | -16px |
| Col1 Server line | x=355 | x=364 | +9px |
| Col2 Client line | x=585 | x=573 | -12px |
| Col2 Server line | x=875 | x=892 | +17px |
| Col3 Client line | x=1105 | x=1097 | -8px |
| Col3 Server line | x=1395 | x=1402 | +7px |
| Short Polling title | x=80 | x=93 | +13px |
| Long Polling title | x=600 | x=633 | +33px |
| SSE title | x=1200 | x=1220 | +20px |

**觀察：**
- 使用者微調了垂直生命線的 x 位置，讓它們更精確地對齊 Client/Server 方塊的中心
- 標題文字做了小幅水平位移，改善了與欄位的對齊
- 箭頭、標籤方框、文字內容和顏色全部保持不變
- 整體結構和設計方向沒有被推翻

### 教訓
1. **風格一致性很重要** — 同一份簡報內，同類型的圖（時序圖）應該用一致的視覺語言
2. **生成的位置只是起點** — Excalidraw 的手繪風格讓精確對齊變困難，使用者手動微調是必要的
3. **用 batch_create 建立骨架，讓使用者做最後微調** — 這個工作流程比較高效

---

## CSS 清理紀錄

移除的 class（僅 Slide 2 使用）：
- `.bubble-container`
- `.bubble`
- `.bubble--sent`
- `.bubble--received`
- `.question-mark`

從 edit mode selector 移除：`.bubble`
