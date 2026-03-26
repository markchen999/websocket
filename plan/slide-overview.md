# Slide Overview — 每頁傳達什麼

| #   | 標題                         | 類型 | 這頁要傳達什麼                                                                                                                                      |
| --- | ---------------------------- | ---- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Cover                        | 靜態 | 主題名稱、講者、日期                                                                                                                                |
| 2   | Agenda                       | 靜態 | 五大段落預覽，讓觀眾知道接下來的結構                                                                                                                |
| 3   | How HTTP Works               | 圖解 | HTTP 是一問一答的模型，問完就斷線。它天生不是為即時通訊設計的                                                                                       |
| 4   | What if we want real-time?   | 圖解 | 人們試過三種方法：Short Polling（浪費）、Long Polling（卡住）、SSE（單向）。都不理想                                                                |
| 5   | Demo: 三種方法對比           | Demo | 讓觀眾親眼看到三種方法的流量差異。Short Polling 瘋狂刷 request，SSE 安靜但只能單向                                                                  |
| 6   | HTTP vs WebSocket            | 圖解 | WebSocket 是一條持久的雙向通道。直接並排對比：HTTP 每次都要重新連線，WebSocket 握手一次就持續開著                                                   |
| 7   | The Cost of One HTTP Request | 圖解 | 一次 HTTP 來回要 282 bytes，WebSocket 只要 54 bytes。Polling 每分鐘浪費 ~8.4KB 的 header 只為了問「有新訊息嗎？」                                   |
| 8   | Demo: Polling vs WebSocket   | Demo | 全場最關鍵的 Demo。左邊 Polling 的計數器瘋狂跳，右邊 WebSocket 幾乎不動。同樣的訊息，開銷差距一目瞭然                                               |
| 9   | Connection Lifecycle         | 圖解 | WebSocket 連線有四個階段：Opening → Open → Closing → Reconnecting。給觀眾一個全貌                                                                   |
| 10  | Heartbeat Ping/Pong          | 圖解 | 怎麼知道對方還在？Server 定期送 Ping，Client 回 Pong。沒回就代表斷了。不做心跳的話 proxy 可能會靜默切掉連線                                         |
| 11  | Disconnection & Reconnection | 圖解 | 斷線分兩種：正常關閉（沒事）和意外斷線（要重連）。重連策略是越等越久，避免所有人同時打爆 server                                                     |
| 12  | Demo: Lifecycle 視覺化       | Demo | 按一個按鈕真的斷開連線，觀眾看到狀態燈變紅、log 顯示重連倒數、最後自動恢復。全部是真實的網路行為                                                    |
| 13  | Security                     | 圖解 | ws:// 明文傳輸，wss:// 加密。認證只在握手時發生一次，token 過期了 server 要主動踢人。對 PM 說：「這是產品設計要考慮的點」                           |
| 14  | Message Patterns             | 圖解 | 三種模式：Broadcast（全部人收到）、Room（只有同群組收到）、One-to-One（只有指定的人收到）。對 PM 說：「設計產品時第一個問題是：誰該收到這則訊息？」 |
| 15  | Demo: 訊息模式互動           | Demo | 切換模式發訊息，6 張使用者卡片對應亮起。Broadcast 全亮、Room 半亮、One-to-One 只亮一個                                                              |
| 16  | Real World                   | 靜態 | 誰在用 WebSocket？Slack、Google Docs、股票報價、線上遊戲、推播通知。讓觀眾把今天學的跟日常經驗連起來                                                |
| 17  | The Cost of WebSocket        | 靜態 | WebSocket 不是免費的。每個連線佔 server 資源、擴展比 HTTP 難（stateful）、要處理斷線重連、基礎設施要額外配置。好東西但有代價                        |
| 18  | When to Use / Not Use        | 靜態 | 判斷標準：需要 server 主動推資料、雙向頻繁溝通、延遲很重要 → 用 WebSocket。普通表單、一次性查詢、資料不常變 → 用 HTTP。結論：HTTP 能做的就用 HTTP   |
| 19  | Live Demo: 聊天室            | Demo | 壓軸互動。觀眾掃 QR code 連進來即時聊天，親身體驗 WebSocket。如果時間夠，現場分 Room                                                                |
| 20  | Summary                      | 靜態 | 四句話收尾：HTTP 是一問一答、WebSocket 是持久雙向、連線有生命週期要管理、選對訊息模式                                                               |
| 21  | Q&A                          | 靜態 | Questions?                                                                                                                                          |
