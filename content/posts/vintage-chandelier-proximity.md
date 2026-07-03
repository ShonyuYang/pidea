---
title: "奶奶的吊燈會說話 — 8 顆 ToF 感測器讓古董燈具活過來"
date: 2026-07-03
draft: false
tags: ["ESP32", "Raspberry Pi", "VL53L1X", "ToF", "DotStar", "互動裝置", "LED"]
categories: ["專案推薦"]
description: "用 8 顆 VL53L1X 飛時測距感測器、ESP32 和 Raspberry Pi，將奶奶的古董吊燈改造成會隨手勢發光、發聲的互動藝術裝置。"
---

## 前言

家裡總有幾件「捨不得丟但也不知道怎麼用」的老東西。這位 maker 選擇了最浪漫的做法——把奶奶留下的古董吊燈，改造成一座會對你的手勢做出反應的互動燈光音效裝置。手伸過去，燈光跟著流動，音效隨距離交叉淡入淡出。復古的外殼裡，藏著 8 顆 ToF 感測器、一條 DotStar LED 燈帶、一顆 ESP32 和一台 Raspberry Pi。

這不只是「加個 LED」的改裝，而是一個完整的雙控制器互動系統設計。

## 專案概述

### 系統架構

整個裝置分為兩層控制：

| 層級 | 硬體 | 職責 |
|------|------|------|
| 感測 + 燈光 | ESP32 | 讀取 8 顆 VL53L1X ToF 感測器、驅動 DotStar LED 燈帶 |
| 音訊處理 | Raspberry Pi | 接收感測器數據、觸發音效、處理交叉淡入淡出 |

### 材料清單

- **感測器**：8× VL53L1X ToF 飛時測距模組（I²C 介面，量測範圍約 4m）
- **控制器**：ESP32 開發板
- **LED**：DotStar（APA102）LED 燈帶（SPI 介面，支援高刷新率）
- **音訊主機**：Raspberry Pi（處理音效觸發與混音）
- **載體**：一盞有故事的古董吊燈

### 運作流程

1. 8 顆 VL53L1X 分布在吊燈不同位置，各自偵測手部距離
2. ESP32 高速輪詢所有感測器，即時更新 DotStar LED 的顏色與亮度
3. ESP32 同時將距離數據傳送給 Raspberry Pi
4. RPi 根據各感測器的距離值，觸發對應音效並做交叉淡入淡出（crossfade）
5. 手越靠近，燈光越亮、音效越清晰；手移開，一切漸漸歸於寧靜

## 技術亮點

### 8 路 ToF 感測器的 I²C 管理

VL53L1X 的預設 I²C 位址都一樣，要同時跑 8 顆需要動點手腳。常見做法有兩種：

- **XSHUT 引腳逐一喚醒**：開機時先把所有感測器 XSHUT 拉低，再逐顆喚醒並重新指派 I²C 位址
- **I²C 多工器**（如 TCA9548A）：用硬體切換通道

不管哪種方式，8 顆感測器的輪詢速度都是關鍵——VL53L1X 單次測距約 20-50ms，8 顆輪詢下來延遲會累積。ESP32 的雙核心在這裡派上用場，一個核心跑感測器，另一個核心刷 LED。

### DotStar vs NeoPixel

選用 DotStar（APA102）而非更常見的 NeoPixel（WS2812B）是個有意思的決定。DotStar 用 SPI 通訊，不像 NeoPixel 需要精確的時序控制，在多工環境下更穩定。缺點是多一條線（clock + data vs data only），但對這種需要即時反應的互動裝置來說，穩定性比省線重要。

### 雙控制器分工

為什麼不全部用 RPi 做？因為 RPi 跑 Linux，做不到微秒級的即時 GPIO 控制。ESP32 負責硬即時的感測與 LED 驅動，RPi 負責它擅長的音訊處理與混音。這種「MCU + SBC」的分工架構在互動裝置中非常實用：

- **ESP32**：確定性高、反應快、功耗低
- **RPi**：運算力強、音訊生態成熟（ALSA、PulseAudio、pygame）

兩者之間可以用 UART 或 USB Serial 傳遞數據，簡單又可靠。

### 交叉淡入淡出音效

不是「碰到就播音效」那麼簡單。交叉淡入淡出（crossfade）意味著多個音效可以同時播放，根據手的距離動態調整各自的音量。這需要：

- 多通道混音（RPi 的 CPU 綽綽有餘）
- 距離到音量的映射曲線（線性？對數？指數？每種感覺都不一樣）
- 平滑的音量過渡，避免爆音（pop/click）

## 心得與延伸

這個專案最打動人的不是技術規格，而是「用科技延續記憶」的概念。一盞不再通電的古董吊燈，變成了一件互動藝術品——它不只是被保存，而是被賦予了新的生命。

### 延伸想法

- **加入 MQTT**：讓手勢數據上傳到 Home Assistant，觸發其他智慧家庭場景
- **換成 VL53L5CX**：8×8 多區域 ToF 感測器，一顆就能偵測 64 個區域，大幅簡化硬體
- **加入手勢辨識**：用 ESP32 做簡單的手勢分類（揮動、靠近、停留），觸發不同模式
- **Web UI**：在 RPi 上跑一個設定頁面，讓使用者自訂燈光效果和音效對應

### 預估成本

| 項目 | 價格（USD） |
|------|------------|
| VL53L1X × 8 | ~$24 |
| ESP32 開發板 | ~$5 |
| DotStar LED 燈帶（1m） | ~$10 |
| Raspberry Pi 3/4 | ~$35-55 |
| 古董吊燈 | 無價 |
| **合計** | **~$75-95**（不含吊燈） |

## 參考資料

- [原文 Reddit 貼文](https://www.reddit.com/r/raspberry_pi/comments/1ukv86j/converted_my_grandmas_vintage_chandelier_into_a/)
- [VL53L1X 資料手冊](https://www.st.com/resource/en/datasheet/vl53l1x.pdf)
- [DotStar（APA102）介紹](https://learn.adafruit.com/adafruit-dotstar-leds)
