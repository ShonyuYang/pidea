---
title: "USB-C 無用盒子 — 3D 列印一個最沒用卻最療癒的桌上小物"
date: 2026-06-13
draft: false
tags: ["3D列印", "Arduino", "伺服馬達", "USB-C", "桌上小物"]
categories: ["專案推薦"]
description: "用 3D 列印打造經典 Useless Box，USB-C 供電、免電池，翻開關就會被關回去的終極療癒裝置。"
---

## 前言

如果你曾經在辦公桌前發呆，想著「我需要一個完全沒有用的東西來讓自己開心」——恭喜你，這個專案就是為你設計的。

**Useless Box**（無用盒子）是 Maker 界的經典之作：你撥動開關，盒子裡伸出一根機械手指把開關撥回去，然後縮回去。就這樣。它做的事情就是把自己關掉。而這個現代版本用 USB-C 供電，免去換電池的煩惱，外殼完全 3D 列印，設計檔案免費下載。

## 專案概述

這個由 Tinkerer Designs 設計的 USB-C Useless Box，是經典無用機器的現代化詮釋：

### 核心規格

| 項目 | 規格 |
|------|------|
| 外殼 | 全 3D 列印（PLA） |
| 供電 | USB-C（5V），免電池 |
| 致動器 | SG90 / MG90S 微型伺服馬達 |
| 控制器 | Arduino Nano 或 ESP32-C3 |
| 設計檔 | MakerWorld 免費下載 STL |
| 預估成本 | ＜ NT$200（不含列印耗材） |

### 運作原理

1. 你撥動開關到「ON」
2. 伺服馬達驅動蓋子打開
3. 機械手指伸出，把開關撥回「OFF」
4. 手指縮回，蓋子關上
5. 回到步驟 1（如果你不死心的話）

整個機構只需要一到兩顆伺服馬達——單馬達版本用一顆同時控制蓋子和手指（透過連桿機構），雙馬達版本則分別控制，動作更流暢也更有「個性」。

### 材料清單

- 3D 列印外殼（約 50-80g PLA）
- 1-2 顆 SG90 或 MG90S 伺服馬達（NT$30-50/顆）
- Arduino Nano 或 ESP32-C3 Super Mini（NT$60-100）
- USB-C 母座模組（NT$10-20）
- 迷你撥動開關（NT$5）
- M2/M3 螺絲若干
- 杜邦線

## 技術亮點

### USB-C 的好處不只是潮

傳統 Useless Box 用 AA 或 9V 電池供電，但伺服馬達的瞬間電流需求容易讓電池快速衰減。USB-C 供電解決了這個問題：

- **穩定 5V 輸出**：伺服馬達運轉更順暢
- **無限續航**：插上就能玩，不用擔心電池沒電
- **通用充電線**：手機充電線就能用，不需要額外準備

### 進階玩法：加入手勢感測

GitHub 上有個 [veggerby/useless-box](https://github.com/veggerby/useless-box) 專案，用 ESP32-C3 搭配 APDS-9960 手勢感測器，讓盒子不只會關開關，還會：

- **偵測你的手靠近**，先偷偷打開蓋子「偷看」
- **根據手勢改變反應**，有時快速、有時猶豫
- **MicroPython 韌體**，程式碼更容易修改

這讓原本單調的「關開關」動作變成一場人機互動的小遊戲。

### 3D 列印小技巧

- 蓋子的鉸鏈處建議用 **0.2mm 層高**列印，確保活動順暢
- 伺服馬達安裝槽可能需要微調——不同廠牌的 SG90 尺寸略有差異
- 建議先不裝蓋子，測試伺服馬達角度和開關位置後再組裝

## 心得與延伸

Useless Box 看似無聊，其實是學習嵌入式系統的絕佳入門專案：

- **伺服馬達控制**：PWM 訊號、角度對應
- **狀態機設計**：偵測開關 → 開蓋 → 伸手 → 關開關 → 縮手 → 關蓋
- **機構設計**：如何讓 3D 列印零件精確配合

如果做完基礎版覺得不過癮，可以考慮：

1. **多動作模式**：像昨天介紹的 15 種效果版，加入隨機動作
2. **音效模組**：加 DFPlayer Mini，讓盒子在動作時發出哀嚎或碎唸
3. **OLED 眼睛**：在蓋子上加小螢幕，顯示表情
4. **多開關版本**：放 3-4 個開關，盒子要一個一個關掉

總成本不到兩百塊、組裝時間約一個下午，卻能帶來無限的「為什麼我要做這個」的存在主義思考。這大概就是 Maker 精神的精髓吧。

## 參考資料

- [MakerWorld - Useless Box with USB-C（Tinkerer Designs）](https://makerworld.com/en/models/1788930-useless-box-with-usb-c)
- [Thingiverse - Useless Box with Arduino/ESP32](https://www.thingiverse.com/thing:4728848)
- [GitHub - veggerby/useless-box（ESP32-C3 + 手勢感測版）](https://github.com/veggerby/useless-box)
- [Instructables - Arduino Useless Box](https://www.instructables.com/Arduino-Useless-Box/)
