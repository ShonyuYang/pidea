---
title: "戴上護目鏡練飛 — RPi 4 × 5.8GHz VTX 把模擬器變成真實 FPV 體驗"
date: 2026-10-22
draft: false
tags: ["Raspberry Pi", "FPV", "無人機", "模擬器", "Composite Video", "5.8GHz", "EdgeTX", "Lua"]
categories: ["專案推薦"]
description: "用 RPi 4 跑 FPV 模擬器，透過 AV 端子接 5.8GHz VTX 廣播到真實護目鏡，搭配真實遙控器操控 — 不需要電腦，不需要接線，練飛體驗無限接近實機。"
---

## 前言

學 FPV 無人機最大的門檻不是買機器，而是「摔得起」。一台 5 吋穿越機摔一次可能就是幾百塊零件費，所以老手都會建議新手先用模擬器練到手感穩定再上天。

問題是，大多數 FPV 模擬器需要一台 PC、一條 USB 線接遙控器、然後盯著螢幕飛。這跟實際戴著護目鏡、拿著遙控器、畫面有類比雜訊的「真實飛行感」差了十萬八千里。

克羅埃西亞的開發者 juricabi 想到一個瘋狂但優雅的解法：**把 Raspberry Pi 4 變成一台「虛擬四軸」**。模擬器畫面從 AV 端子輸出，接到 5.8GHz VTX（影像發射器），用真正的 FPV 護目鏡接收；操控用真正的 EdgeTX 遙控器透過 ExpressLRS WiFi 傳輸。整套系統不需要 PC、不需要接線到遙控器，練飛體驗無限接近實機。

## 專案概述

### StickTime 的前世今生

StickTime 原本是一個跑在 EdgeTX 遙控器螢幕上的 Lua 3D FPV 模擬器 — 對，就是那個只有 480×272 解析度的小螢幕。它能在遙控器上跑 AI 對戰、自由飛行、計時賽，完全不需要外接任何設備。

StickTime Pi 則是把這個模擬器「搬」到 Raspberry Pi 上，但不是簡單的移植。作者寫了一個原生 C 語言主機程式，實作了 EdgeTX 的 Lua API，讓原本的遊戲腳本 `StickTime.lua` **完全不用修改**就能在 Pi 上跑。這是一個精心設計的相容層，不是暴力重寫。

### 系統架構

```
┌─────────────┐    WiFi/USB/BLE    ┌──────────────┐
│  EdgeTX 遙控器 │ ─────────────────→ │              │
│  (ExpressLRS)  │                    │  RPi 4       │
└─────────────┘                    │              │
                                    │  StickTime   │
                                    │  Pi (C)      │
                                    │              │
                                    │  Lua 5.3     │
                                    │  Game Script │
                                    │              │
                                    └──┬───────┬───┘
                                       │       │
                              AV 端子   │       │ GPIO4
                              (TRRS)    │       │ (1kΩ)
                                       ▼       ▼
                                    ┌──────────────┐
                                    │  5.8GHz VTX  │
                                    │  (25mW)      │
                                    │  SmartAudio   │
                                    └──────┬───────┘
                                           │ 5.8GHz
                                           ▼
                                    ┌──────────────┐
                                    │  FPV 護目鏡   │
                                    │  / 監視器     │
                                    └──────────────┘
```

### 材料清單

| 零件 | 用途 | 估價 |
|------|------|------|
| Raspberry Pi 4 (任何 RAM) | 主機 | ~$35–55 |
| 5.8GHz VTX（含天線） | 影像發射 | ~$10–20 |
| 3.5mm TRRS 線材 | AV 輸出接 VTX | ~$2 |
| 1kΩ 電阻 | VTX 控制線 | ~$0.1 |
| microSD 卡 (≥8GB) | 系統映像 | ~$5 |
| 5V/3A 電源 | 供電 | ~$8 |
| **選配：** 3D 列印外殼 | 整合 Pi + VTX + 風扇 | 耗材成本 |

**總成本約 $60–90**（假設你已經有 FPV 護目鏡和遙控器）。

## 技術亮點

### 1. 零修改的遊戲相容層

這是整個專案最令人印象深刻的設計。StickTime Pi 不是「把遊戲移植到 Pi」，而是「在 Pi 上重建遙控器的執行環境」。

作者用 C 語言實作了 EdgeTX 的 Lua API，包括繪圖函數、輸入讀取、音效系統。Lua 5.3 引擎使用 EdgeTX 的數值設定（整數精度），確保遊戲邏輯完全一致。結果是 `StickTime.lua`（2,657 行、106KB）一行都不用改就能跑。

這個設計的好處是：遊戲更新和 Pi 主機更新完全獨立。遊戲從 EdgeTX 社群取得更新，主機程式從 GitHub Releases 更新，互不干擾。

### 2. 極低延遲的渲染管線

模擬器最怕延遲。StickTime Pi 的渲染管線刻意避開所有中間層：

- **沒有 X Window、沒有 Wayland、沒有瀏覽器** — 直接用 DRM/KMS page flip 寫入 framebuffer
- **搖桿輸入在每一幀繪製前才讀取** — 最小化輸入延遲
- **Composite 輸出使用 progressive scan**（240p/288p）— 沒有隔行掃描的閃爍問題
- **音效自行混音** — 馬達聲、嗶聲、音樂都不經過系統音效層

Web 介面上可以即時看到每幀的渲染延遲和搖桿延遲數據。

### 3. 類比訊號的「缺陷」反而是優點

FPV 老手都知道，類比訊號有雜訊、有斷訊、有多路徑干擾 — 這些「缺陷」在模擬器裡反而成了最好的訓練。當你的護目鏡畫面開始出現雪花，你的大腦會自動學會在雜訊中保持方向感，這是 HDMI 接螢幕永遠練不到的。

StickTime Pi 讓你用跟實機完全相同的訊號鏈路練飛：**同一副護目鏡、同一支遙控器、同一個 VTX 頻段**。唯一的差別是天上沒有真正的飛機。

### 4. VTX 智慧控制

Pi 的 GPIO4 透過一顆 1kΩ 電阻接到 VTX 的控制端子，就能模擬飛控板的 SmartAudio/Tramp 協議。在 Web 介面上可以直接切換頻段、頻道、發射功率和 Pit Mode，不需要按 VTX 上的小按鈕。Pi 會自動偵測 VTX 使用的協議版本（SmartAudio 1.0/2.0/2.1 或 Tramp）。

### 5. 遙控器連接方式多元

| 連接方式 | 說明 |
|----------|------|
| ExpressLRS WiFi Joystick | 最推薦，無線、低延遲 |
| TBS Crossfire WiFi | 同上，Crossfire 生態系 |
| USB Joystick | 有線，零延遲 |
| ELRS BLE Joystick | 藍牙，方便但延遲略高 |
| 遊戲手把 / 鍵盤 | 入門替代方案 |

選單操作也能用搖桿完成，所以護目鏡戴上就不用拿下來。

### 6. 一個靜態執行檔搞定一切

整個 StickTime Pi 是一個靜態編譯的 aarch64 執行檔。不依賴任何動態函式庫，不需要特定版本的 Pi OS。更新就是換一個檔案，回滾也是換回來。SD 卡的 FAT 分區上放著所有設定（`sticktime.txt`）、賽道、存檔和音樂，拔出來用電腦就能編輯。

## 心得與延伸

### 為什麼這個專案值得關注

FPV 模擬器不是新東西 — Velocidrone、Liftoff、Uncrashed 都是成熟的 PC 模擬器。但它們都需要一台有獨顯的電腦，而且畫面是透過螢幕看的，跟戴護目鏡飛的感覺完全不同。

StickTime Pi 的價值在於它填補了一個很具體的空缺：**用最少的硬體，最大程度還原真實 FPV 飛行的感官體驗**。$60 的成本、一張 SD 卡刷好就能用、不需要任何軟體安裝 — 這對 FPV 社群來說是一個門檻極低的練習方案。

### 軟體工程的啟示

作者選擇「重建執行環境」而非「移植遊戲」的策略非常值得學習。這讓遊戲和主機完全解耦，各自可以獨立演進。這跟 Docker 容器化的哲學有異曲同工之妙 — 不是改應用程式，而是改執行環境來適應應用程式。

### 可能的延伸

- **多人對戰**：多台 Pi 各自發射不同頻道，幾個人戴護目鏡一起飛
- **錄影回放**：把模擬器的飛行軌跡錄下來，在護目鏡裡回放分析
- **搭配真實場景**：把 Pi 放在飛行場地，VTX 訊號覆蓋整個場地，就像有一台隱形的練習機
- **教學用途**：FPV 教練可以用一台 Pi 讓學員用自己的護目鏡和遙控器練習，不用擔心摔機

### 注意事項

- VTX 是無線電發射器，使用前請確認當地法規。在台灣，5.8GHz 頻段的使用需注意 NCC 規範
- 練飛時遙控器仍會發射訊號，**務必拔掉真機的電池或拆下螺旋槳**
- VTX 天線必須在通電前接好，否則可能燒毀 VTX
- 建議 VTX 功率設定為 25mW，室內使用綽綽有餘

## 參考資料

- 🔗 GitHub Repo：[juricabi/sticktime-pi](https://github.com/juricabi/sticktime-pi)
- 🔗 原始 StickTime 模擬器：[juricabi/sticktime](https://github.com/juricabi/sticktime)
- 📺 示範影片：[YouTube](https://www.youtube.com/watch?v=Khq31t_4pqc)
- 🔗 Reddit 討論：[r/raspberry_pi](https://www.reddit.com/r/raspberry_pi/comments/1xbo5ry/)
- 📰 FabScene 報導：[ラズパイをドローンのシミュレーターに](https://fabscene.com/new/make/sticktime-pi-raspberry-pi-fpv-simulator-analog-goggles/)
- 📄 授權：GPL-2.0
