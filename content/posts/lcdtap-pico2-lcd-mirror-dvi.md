---
title: "LcdTap：用 Pico 2 竊聽匯流排，把小 LCD 鏡像到大螢幕"
date: 2026-05-21
draft: false
tags: ["RP2350", "Pico 2", "PIO", "DVI", "SPI", "I2C", "嵌入式工具"]
categories: ["專案推薦"]
description: "一顆 Pico 2 被動竊聽 SPI/I2C 匯流排，即時把 ST7789、SSD1306 等小型 LCD 畫面鏡像輸出到 DVI 大螢幕，60 FPS 不影響原裝置。"
---

## 前言

做嵌入式開發的人一定遇過這種痛：你的裝置上只有一塊 1.3 吋的小螢幕，想截圖寫文件？拿手機拍反光又模糊。想錄影做教學？角度喬半天還是看不清楚。

日本開發者 shapoco 做了一個漂亮的解法——**LcdTap**。用一塊 Raspberry Pi Pico 2，被動竊聽 MCU 和 LCD 之間的匯流排通訊，即時把畫面透過 DVI 輸出到任何大螢幕上。不需要改韌體、不需要動原本的電路，接上線就能用。

## 專案概述

LcdTap 的核心概念是「被動竊聽」（bus sniffing）。它不參與 MCU 和 LCD 模組之間的通訊，只是旁聽 SPI 或 I2C 匯流排上的指令流，解析出顯示資料後更新自己的 framebuffer，再透過 PicoDVI 函式庫輸出 DVI 訊號。

### 硬體架構

| 元件 | 說明 |
|------|------|
| **主控** | Raspberry Pi Pico 2 (RP2350) |
| **輸入** | 被動竊聽 SPI 或 I2C 匯流排 |
| **輸出** | DVI（透過 PicoDVI / libdvi） |
| **外部電路** | SPI 高速模式需串列轉並列轉換電路 |

### 支援的 LCD 模組

- **SSD1306**：128×64 OLED，I2C 介面
- **ST7789**：240×240 TFT LCD，SPI 介面，已實測 60 FPS

### 成本估算

- Pico 2：約 NT$150
- DVI Breakout Board：約 NT$100–200
- 串列轉並列 IC + 被動元件：約 NT$50
- **總計約 NT$300–500**，比任何商用 SPI 分析儀都便宜

## 技術亮點

### PIO 狀態機的極限操作

RP2350 的 PIO（Programmable I/O）是這個專案的靈魂。SPI 匯流排的時脈高達 **62.5 MHz**，一般的 GPIO 中斷根本來不及處理。LcdTap 利用 PIO 狀態機直接在硬體層級接收 SPI 資料，完全不佔用 CPU 週期。

不過 PIO 狀態機的 pin 數量有限，無法直接同時處理 SPI 的所有訊號線。shapoco 的解法是加一顆外部串列轉並列轉換 IC，把 SPI 資料先轉成並列格式再餵給 PIO——這是在硬體限制下的巧妙工程妥協。

### 被動竊聽 vs. 主動攔截

市面上的邏輯分析儀（如 Saleae）也能抓 SPI 資料，但它們只是記錄原始波形，你還得自己解碼。LcdTap 直接理解 LCD 控制器的指令集（ST7789 的 RAMWR、SSD1306 的 page addressing），即時重建畫面。

而且因為是純被動竊聽，**對原裝置零影響**——不改變匯流排阻抗、不增加延遲、不需要修改韌體。這在除錯已出貨產品時特別有用。

### DVI 輸出的選擇

使用 PicoDVI 函式庫意味著輸出是標準 DVI 訊號，任何有 HDMI 輸入的螢幕都能直接使用。比起在 PC 上跑軟體解碼，這個方案延遲更低、也不需要額外的電腦。

## 心得與延伸

這個專案展示了 RP2350 作為「嵌入式瑞士刀」的潛力。PIO 狀態機讓它能處理各種非標準協定，而雙核心架構讓一個核心處理匯流排竊聽、另一個核心處理 DVI 輸出，互不干擾。

**可能的延伸方向：**

- 🔧 **支援更多 LCD 控制器**：ILI9341、GC9A01 等常見型號，擴大適用範圍
- 📸 **截圖功能**：加上按鈕或 UART 指令，把 framebuffer 存成 BMP/PNG 到 SD 卡
- 📡 **無線串流**：改用 Pico 2 W，透過 Wi-Fi 把畫面串流到電腦上
- 🎓 **教學利器**：在工作坊或課堂上，讓所有人都能看到小螢幕的內容

對於嵌入式開發者來說，這是一個成本低、實用性高的工具。下次寫文件需要螢幕截圖時，不用再拿手機拍了。

## 參考資料

- 🔗 Reddit 原文：https://www.reddit.com/r/embedded/comments/1tc5g85/mirroring_a_tiny_lcd_module_onto_a_large_monitor/
- 💻 GitHub：https://github.com/shapoco/lcdtap
- 📦 PicoDVI 函式庫：https://github.com/Wren6991/PicoDVI
