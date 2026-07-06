---
title: "烤肉不用顧火 — RPi × 433MHz 無線溫度計打造即時 BBQ 儀表板"
date: 2026-07-06
draft: false
tags: ["raspberry-pi", "laravel", "websocket", "bbq", "iot", "gpio"]
categories: ["專案推薦"]
description: "八年三代進化的開源 BBQ 溫度監控系統，用 C daemon 攔截 433MHz 射頻訊號，Laravel 13 + WebSocket 即時推送溫度曲線到手機。"
---

## 前言

烤肉最怕什麼？不是調味失敗，是**離開烤架那十分鐘肉就焦了**。

商用 BBQ 溫度計不便宜，而且多數是封閉系統——你只能用它的 app、看它的介面、接受它的限制。如果你手邊剛好有一台 Raspberry Pi 和一支 Maverick ET-732 無線溫度計，這個開源專案讓你用一個 C daemon + 一套 Laravel 全端應用，把溫度監控完全掌握在自己手上。

更厲害的是，這不是一個週末 hack——作者從 2018 年做到 2026 年，歷經三個大版本，每次都用當時最新的技術棧重寫。這是一個 maker 專案**活了八年**的故事。

## 專案概述

### 系統架構

```
┌─────────────────┐  433MHz RF   ┌──────────────────┐
│ Maverick ET-732 │ ───────────► │   maverick.c     │  C daemon + pigpio
│   無線溫度計     │              └────────┬─────────┘
└─────────────────┘                       │ 寫入讀數
                                          ▼
                                 ┌──────────────────┐
                                 │  SQLite 資料庫    │
                                 └────────┬─────────┘
                                          │
                                          ▼
                                 ┌──────────────────┐
                                 │  Laravel 13 App  │  Livewire + Filament UI
                                 └────────┬─────────┘
                                          │ WebSocket
                                          ▼
                                 ┌──────────────────┐
                                 │  Laravel Reverb   │  即時推送
                                 └────────┬─────────┘
                                          │
                                          ▼
                                 ┌──────────────────┐
                                 │  瀏覽器 / PWA     │  手機、平板、桌機
                                 └──────────────────┘
```

### 核心元件

| 元件 | 角色 |
|------|------|
| `maverick.c` | 讀取 GPIO pin 15，解碼 ET-732 的 RF 封包，透過 Artisan 命令寫入資料庫 |
| `maverick.sh` | daemon 的啟動/停止/狀態控制腳本 |
| `maverick-fake.sh` | 本地開發用的模擬器，不需要實體硬體也能測試 |
| `MaverickService` | PHP 服務層，控制 daemon 生命週期 |
| `LiveCookBroadcast` | 透過 Reverb 推送即時圖表更新事件 |
| `TemperatureAlertService` | 評估溫度讀數，觸發瀏覽器推播通知 |

### 硬體需求

- Raspberry Pi（任何有 GPIO 的型號）
- Maverick ET-732 無線 BBQ 溫度計
- 433MHz RF 接收晶片（連接到 GPIO BCM pin 15）
- 就這樣，三樣東西

### 軟體堆疊

- **後端**：PHP 8.3+、Laravel 13、Livewire 4、Filament 5
- **即時通訊**：Laravel Reverb（WebSocket）
- **前端**：Chart.js 即時溫度曲線
- **資料庫**：SQLite（輕量、免設定）
- **部署**：Caddy + PHP-FPM + systemd

## 技術亮點

### 1. 用 C 解碼 433MHz 射頻訊號

這個專案最硬核的部分不是 Laravel，而是那個 `maverick.c`。Maverick ET-732 用 433MHz 無線頻段傳輸溫度數據，作者寫了一個 C daemon，透過 `pigpio` 函式庫直接讀取 GPIO 腳位的射頻訊號，解碼出食物探針和烤架探針的溫度讀數。

這意味著你**不需要**原廠的接收器或 app——Pi 直接攔截空中的射頻封包。

### 2. WebSocket 即時推送

v3 最大的升級是加入 Laravel Reverb 做 WebSocket 即時推送。烤肉時打開手機瀏覽器，溫度曲線會**自動更新**，不用手動重新整理。溫度超出設定範圍時，瀏覽器會跳出推播通知——即使你在客廳看電視也不會錯過。

通知頻率可設定冷卻時間（1、3、5、10、15 分鐘），避免被轟炸。

### 3. PWA 支援

整個 web app 可以安裝為 Progressive Web App，從手機主畫面一鍵開啟，體驗接近原生 app。在戶外烤肉時特別實用——不用開瀏覽器找書籤，直接點 icon 就看到溫度。

### 4. 完整的烹飪記錄系統

不只是即時監控，每次烤肉都會被完整記錄：
- **烹飪歷史**：瀏覽所有完成的烹飪，含日期、標題、描述
- **互動式圖表**：可以刪除異常讀數、標記特定時間點的筆記
- **烤爐管理**：追蹤不同烤爐的使用情況
- **統計頁面**：累計總烹飪時間

### 5. 八年三代的技術演進

| 版本 | 年份 | 技術棧 |
|------|------|--------|
| v1 | 2018 | 基礎版本 |
| v2 | 2023 | 中期改版 |
| v3 | 2026 | Laravel 13 全面重寫 + WebSocket + PWA |

205 個 commits，從一個簡單的溫度讀取腳本，進化成一個功能完整的全端應用。這種持續迭代的精神，比任何單次的華麗 demo 都更值得學習。

### 6. 開發體驗友善

作者顯然很在意 DX（開發者體驗）：
- `composer setup` 一鍵完成所有安裝
- `composer dev` 同時啟動 web server、WebSocket server、log viewer 和 Vite
- `maverick-fake.sh` 模擬器讓你**不需要實體硬體**就能在筆電上開發測試
- Passkey（WebAuthn）無密碼登入支援

## 心得與延伸

### 為什麼這個專案特別？

市面上不缺 BBQ 溫度監控方案，但多數是「讀溫度 → 顯示數字」的簡單實作。這個專案的價值在於：

1. **射頻解碼**：直接攔截 433MHz 訊號，不依賴原廠 SDK 或 API
2. **全端工程**：從 C daemon 到 Laravel 後端到 Chart.js 前端，完整的技術棧
3. **長期維護**：八年不是一個小數字，代表作者真的在用這個東西
4. **生產品質**：systemd 服務、Cloudflare DDNS、推播通知——這不是玩具

### 延伸想法

- **多溫度計支援**：目前綁定 ET-732，但 433MHz 射頻解碼的思路可以套用到其他無線溫度計
- **Home Assistant 整合**：已有人做了 [HA 整合](https://github.com/dennyreiter/HAMaverickTempProbe)，可以把溫度數據接入智慧家庭系統
- **AI 預測**：累積足夠的烹飪數據後，可以訓練模型預測「這塊肉還要多久才熟」
- **多 Pi 聯網**：如果你有多個烤爐，可以用多台 Pi 各自監控，數據匯聚到同一個 dashboard

### 適合誰？

- 認真對待 BBQ 的 maker（尤其是低溫慢烤 low-and-slow 愛好者）
- 想學 Laravel 全端開發的人（這是一個結構清晰、測試完整的真實專案）
- 對 433MHz 射頻解碼有興趣的硬體玩家

## 參考資料

- **GitHub Repo**：[produktive/Maverick.bbq](https://github.com/produktive/Maverick.bbq)
- **Reddit 原文**：[r/raspberry_pi — Introducing Maverick.bbq v3](https://www.reddit.com/r/raspberry_pi/comments/1ugr3jb/introducing_maverickbbq_version_3_an_opensource/)
- **Laravel 版討論**：[r/laravel — Maverick.bbq v3](https://www.reddit.com/r/laravel/comments/1ugr6wc/introducing_maverickbbq_version_3_an_opensource/)
- **HeaterMeter**：[另一個知名的 RPi BBQ 控制器](https://www.raspberrypi.com/news/heatermeter-open-source-barbecue-controller/)
