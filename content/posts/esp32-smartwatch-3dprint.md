---
title: "自己的手錶自己印：開源 ESP32 智慧手錶完全攻略"
date: 2026-04-03
draft: false
tags: ["ESP32", "3D列印", "智慧手錶", "穿戴裝置", "開源硬體"]
categories: ["專案推薦"]
description: "從 PCB 到外殼全部開源，一支 ESP32 智慧手錶的 DIY 完整指南"
---

## 前言

市面上的智慧手錶功能越來越多，但真正屬於「你」的有幾支？當一位 Reddit 用戶在 r/arduino 貼出自己 3D 列印的 ESP32 智慧手錶時，368 個讚和一片叫好聲說明了一切 — 大家想要的不只是手腕上的螢幕，而是一個可以自己掌控、自己改造的穿戴裝置。

這個專案把硬體設計、3D 列印外殼、韌體程式碼全部免費開源，讓任何有 3D 印表機和焊接工具的人都能從零打造一支屬於自己的智慧手錶。

## 專案概述

這支 ESP32 智慧手錶的核心理念是 **「完全可複製」**：所有設計檔案、程式碼、組裝說明都公開在網路上，不需要任何商業授權。

### 硬體架構

- **主控**：ESP32（支援 Wi-Fi + Bluetooth）
- **顯示**：小尺寸 TFT 或 OLED 螢幕
- **外殼**：3D 列印，STL 檔案公開下載
- **電源**：鋰電池供電，USB 充電
- **開發環境**：Arduino IDE，上手門檻低

### 功能特色

- 時間顯示（可自訂錶面）
- Wi-Fi 連線同步時間
- 藍牙配對手機通知
- USB 直接燒錄韌體，不需外接底座
- 標準錶帶相容（可換裝個人風格）

### 預估成本

根據類似專案的 BOM 估算，整支手錶材料費約 **$30–70 美元**，取決於螢幕和電池的選擇。相比之下，一支 Apple Watch SE 要 $249 起跳。

## 技術亮點

### 1. ESP32 作為穿戴裝置核心的可行性

ESP32 本來是為 IoT 設計的，但它的低功耗模式（deep sleep 僅 10μA）、內建 Wi-Fi/BT、充足的 GPIO，讓它意外地適合穿戴裝置。這個專案證明了不需要專用的穿戴晶片，一顆通用 MCU 就能撐起一支日用手錶。

### 2. 3D 列印外殼的設計考量

手錶外殼是 3D 列印專案中最考驗設計功力的品類之一：

- **尺寸精度**：手腕上的東西差 0.5mm 就會不舒服
- **結構強度**：日常碰撞在所難免，壁厚和卡扣設計很關鍵
- **美觀度**：畢竟是要戴出門的，後處理（打磨、噴漆）也是學問

作者提供了完整的 STL 檔案，使用者可以直接列印，也可以用 FreeCAD 或 Fusion 360 修改成自己喜歡的造型。

### 3. Arduino 生態系的優勢

選擇 Arduino IDE 開發意味著：

- 海量的現成 library（NTP 校時、BLE 通訊、OLED 驅動…）
- 社群支援豐富，遇到問題容易找到解答
- 進入門檻低，不需要嵌入式系統背景也能上手

## ESP32 智慧手錶生態一覽

這個專案並不孤單，ESP32 智慧手錶已經形成了一個小型生態系：

| 專案 | 特色 | 狀態 |
|------|------|------|
| **Open-SmartWatch** | 圓形 TFT、加速度計、GPS 版本開發中 | 活躍，有 Discord 社群 |
| **Bellafaire ESP32-Smart-Watch** | 230+ stars、Android companion app | 成熟，多版本迭代 |
| **MutantW V2** | ESP32-S3、1.7 吋 IPS、日用級完成度 | Instructables 有完整教學 |
| **DRM Watch 3** | Sharp Memory LCD、14 天續航 | 2025 年新作，$70 材料費 |

這些專案互相參考、互相啟發，形成了一個健康的開源穿戴裝置社群。

## 心得與延伸

### 為什麼要自己做手錶？

坦白說，自製手錶在功能上很難跟商業產品比。但 DIY 手錶的價值不在功能多寡：

1. **學習價值**：一個專案就涵蓋了 PCB 設計、嵌入式程式、3D 建模、電源管理
2. **客製化自由**：想加感測器就加，想改錶面就改，不受廠商限制
3. **維修權**：壞了自己修，不用寄回原廠等兩週
4. **成就感**：戴著自己做的手錶出門，那種感覺是買不到的

### 可能的改進方向

- **加入 E-Paper 螢幕**：大幅提升續航，陽光下可讀性更好
- **整合 ESPHome**：讓手錶變成智慧家庭的遙控器
- **PCB 小型化**：用 ESP32-S3 模組取代 WROOM，進一步縮小體積
- **加入計步器**：BMA400 加速度計只要幾塊錢，就能實現基本健康追蹤

### 適合誰？

- 想學嵌入式系統但不知道做什麼專案的初學者
- 有 3D 印表機閒置想找東西印的 Maker
- 對商業智慧手錶的封閉生態感到不滿的人
- 單純覺得「自己做手錶」很酷的人

## 參考資料

- [原始 Reddit 貼文（r/arduino）](https://www.reddit.com/r/arduino/comments/1ren9u8/my_free_opensource_3dprinted_esp32_smart/)
- [Bellafaire ESP32-Smart-Watch（GitHub）](https://github.com/Bellafaire/ESP32-Smart-Watch)
- [Open-SmartWatch 官網](https://open-smartwatch.github.io/)
- [MutantW V2 教學（Instructables）](https://www.instructables.com/MutantW-V2-DIY-ESP32-S3-Smartwatch-That-You-Can-We/)
- [DRM Watch 3（Hackster.io）](https://www.hackster.io/drfailov/diy-esp32-wearable-drm-watch-3-suitable-for-daily-use-94150b)
