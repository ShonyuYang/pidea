---
title: "你家上空飛過什麼？— RPi 本地空域感測站完全指南"
date: 2026-06-29
draft: false
tags: ["Raspberry Pi", "ADS-B", "SDR", "無人機", "Remote ID", "航空"]
categories: ["專案推薦"]
description: "用 Raspberry Pi + SDR 接收器打造個人空域雷達站，同時追蹤飛機 ADS-B 訊號與無人機 Remote ID 廣播"
---

## 前言

你有沒有抬頭看著天空，好奇那架飛機是從哪裡來、要飛去哪裡？或者注意到附近越來越多無人機嗡嗡作響，卻不知道是誰在操控？

一位 Reddit 用戶剛釋出了他的 v1 版本專案——用一台 Raspberry Pi 同時監控兩種完全不同的空域訊號：**傳統飛機的 ADS-B 廣播**和**無人機的 FAA Remote ID**。這不是依賴 Flightradar24 或 FlightAware 的雲端資料，而是你自己的接收器、你自己的資料、你自己的本地雷達站。

## 專案概述

### 什麼是 ADS-B？

**ADS-B（Automatic Dependent Surveillance-Broadcast）** 是現代航空管制的核心技術。每架商用飛機都會在 **1090 MHz** 頻率上持續廣播自己的位置、高度、速度和航班代號。這是公開的無加密訊號——任何人只要有正確的接收器就能收到。

地面接收器的有效範圍可達 **200 海里（約 370 公里）**，這意味著一台放在屋頂的小天線就能追蹤方圓數百公里內的所有航班。

### 什麼是 Remote ID？

**Remote ID** 是各國航空主管機關（如美國 FAA）對無人機的新規範。自 2023 年 9 月起，所有新製造的無人機都必須在飛行時透過 **2.4 GHz WiFi 或 BLE** 廣播自己的識別資訊，包括：

- 無人機序號或註冊號碼
- 即時位置與高度
- 操控者位置
- 起飛點座標
- 緊急狀態碼

這就像是無人機版的車牌——讓周圍的人能知道「這是誰的無人機」。

### 硬體需求

| 元件 | 用途 | 參考價格 |
|------|------|---------|
| Raspberry Pi 3/4/5 | 主控電腦 | ~$35–75 |
| RTL-SDR USB 接收器 | 接收 1090 MHz ADS-B 訊號 | ~$25–30 |
| 1090 MHz 天線 | ADS-B 專用天線 | ~$15–20 |
| USB WiFi 適配器（如 Alfa AWUS036N） | 接收 2.4 GHz Remote ID | ~$18–22 |
| 2.4 GHz 全向天線（5-9 dBi） | Remote ID 接收天線 | ~$10–15 |
| **合計** | | **~$100–160** |

如果你已經有 RPi 和 ADS-B 設備，加裝 Remote ID 偵測只需額外 **$25–30**。

### 軟體架構

這個專案的巧妙之處在於兩套系統完全獨立運作：

```
┌─────────────────────────────────────────┐
│            Raspberry Pi                  │
│                                          │
│  ┌──────────────┐  ┌──────────────────┐ │
│  │  dump1090     │  │  Remote ID       │ │
│  │  (ADS-B 解碼) │  │  Monitor         │ │
│  │  Port 8080    │  │  (WiFi 監聽)     │ │
│  └──────┬───────┘  └────────┬─────────┘ │
│         │                    │           │
│  ┌──────┴────────────────────┴─────────┐ │
│  │     本地 Web Dashboard               │ │
│  │     整合空域態勢顯示                  │ │
│  └─────────────────────────────────────┘ │
│                                          │
│  RTL-SDR ◄── 1090 MHz 天線 (飛機)       │
│  USB WiFi ◄── 2.4 GHz 天線 (無人機)     │
└─────────────────────────────────────────┘
```

- **dump1090 / readsb**：解碼 ADS-B 訊號，支援 Mode S 和 UAT 格式，具備多來源聚合與衝突處理
- **Remote ID 監聽器**：將 USB WiFi 設為 monitor mode，被動擷取 Remote ID 廣播封包
- **Web 介面**：在地圖上即時顯示飛機與無人機位置

## 技術亮點

### 1. 兩種訊號，兩個世界

ADS-B 和 Remote ID 覆蓋的是完全不同的空域層次：

| | ADS-B（飛機） | Remote ID（無人機） |
|---|---|---|
| **頻率** | 1090 MHz | 2.4 GHz WiFi / BLE |
| **偵測範圍** | 100–250+ 英里 | 1–5 英里 |
| **高度覆蓋** | 500 英尺以上 | 地面至 ~500 英尺 |
| **目標** | 商用/私人飛機 | 消費級/商用無人機 |

兩者結合起來，你就擁有了從地面到高空的**完整空域態勢感知**。

### 2. Feed 聚合與衝突處理

原文特別提到系統支援 **ADS-B / Mode S / UAT feed aggregation with merge conflict handling**。這代表它不只是被動接收，還能：

- 同時處理多種航空訊號格式
- 當多個來源提供同一架飛機的資料時，智慧合併並解決衝突
- 這在多接收器部署時特別有用

### 3. 完全本地運作

與 FlightAware、Flightradar24 等雲端服務不同，這個專案的核心理念是**本地優先**：

- 資料不需上傳到任何第三方伺服器
- 不需要網路連線也能運作（只需要 GPS 時間同步）
- 你完全掌控自己的資料
- 適合隱私敏感或網路受限的場景

### 4. 天線放置的學問

接收效果的關鍵不在設備多貴，而在天線放得多高。根據 DroneAware 社群的經驗：

- **屋頂或閣樓安裝**可以讓偵測範圍提升 **3 倍**
- 如果你已經有 ADS-B 天線桿，直接在上面加裝第二根 2.4 GHz 天線
- 1090 MHz 天線建議用專用的 ADS-B 天線而非通用 DVB-T 天線，增益差異顯著

## 心得與延伸

### 為什麼這個專案值得關注？

1. **法規趨勢**：Remote ID 是全球趨勢，歐盟、日本等地也在推動類似規範。現在建立偵測能力，未來只會越來越有用。

2. **社群力量**：FlightAware 的 43,000+ 個 ADS-B 接收站證明了「群眾感測」模式的可行性。Remote ID 的偵測網路正在走同樣的路。

3. **安全應用**：對於機場附近居民、活動場地管理者、關鍵基礎設施守護者，無人機偵測不再是奢侈品。

### 延伸玩法

- **整合 Home Assistant**：當偵測到無人機進入特定區域時觸發自動化（開燈、發通知、啟動攝影機）
- **歷史資料分析**：記錄長期飛行資料，分析航線模式、噪音時段、無人機活動熱區
- **多節點部署**：在不同位置放置多台 RPi，透過網路聚合資料，擴大覆蓋範圍
- **加入 SSTV / APRS 接收**：同一台 SDR 還能解碼業餘無線電影像傳輸和 APRS 位置封包
- **結合 ESP32**：用 ESP32 做低成本的 Remote ID 專用偵測節點，再回傳給 RPi 中控

### 成本效益分析

一套商業級空域監控系統動輒數萬美元。這個 DIY 方案用不到 $160 就能達到類似效果——雖然精度和可靠性無法與商業設備相比，但對個人愛好者和學習目的來說綽綽有餘。

## 參考資料

- [原文 Reddit 貼文](https://www.reddit.com/r/raspberry_pi/comments/1ufj6vv/using_a_raspberry_pi_as_a_local_airspace_sensor/)
- [DroneAware — 為 FlightAware Pi 加裝無人機偵測](https://droneaware.io/flightaware.html)
- [Raspberry Pi 官方飛行追蹤教學](https://www.raspberrypi.com/tutorials/build-your-own-raspberry-pi-flight-tracker/)
- [RTL-SDR ADS-B 教學](https://raspberry.tips/en/raspberrypi-tutorials/raspberry-pi-ads-b-flight-radar)
- [AeroScope — ADS-B 接收器 DIY 指南](https://aeroscope.live/adsb-receiver-setup)
- [RemoteIDReceiver — 開源 Remote ID 監控平台（GitHub）](https://github.com/cyber-defence-campus/RemoteIDReceiver)
