---
title: "自己造一台無反相機 — RPi 5 × IMX283 一吋感測器 DIY 街拍神器"
date: 2026-08-01
draft: false
tags: ["Raspberry Pi 5", "IMX283", "相機", "3D 列印", "攝影", "OneInchEye"]
categories: ["專案推薦"]
description: "攝影師 Malcolm Wilson 花了 10 小時，用 RPi 5、Sony IMX283 一吋感測器和 Mamiya C220 腰平觀景窗，打造一台有對焦峰值、即時直方圖和 Wi-Fi 傳檔的 3D 列印數位相機。"
---

## 前言

你有沒有想過，一台「真正能拍照」的數位相機，其實可以自己做？

不是那種拿 Pi Camera Module 拍出來糊成一片的玩具，而是搭載 **Sony IMX283 一吋感測器**、有**對焦峰值**、**即時直方圖**、**手動快門速度**和 **Wi-Fi 傳檔**的完整相機。攝影師 [Malcolm Wilson](https://camerahacksbymalcolmjay.substack.com/) 就做到了——而且從構想到完成只花了大約 **10–12 小時**。

他帶著這台自製相機出門街拍旅行，拍出來的照片讓人完全忘記這是一台 Raspberry Pi 驅動的 DIY 裝置。

## 專案概述

### 設計理念

Malcolm 自稱是個 90 年代小孩，對方方正正、賽博龐克風格的科技產品有特殊情感。他厭倦了千篇一律的黑色相機和無止盡的復古底片機改造，決定做一台「讓攝影重新變得有趣」的東西。

最終成品是一台**腰平觀景窗**風格的相機——就像經典的 Hasselblad 或 Mamiya 中片幅相機那樣，低頭往下看取景，然後按下快門。

### 核心硬體

| 元件 | 型號 | 用途 | 參考價格 |
|------|------|------|----------|
| 主控板 | Raspberry Pi 5 | 運算核心、影像處理 | ~$80 |
| 感測器 | OneInchEye V2（IMX283） | 1 吋 CMOS 影像感測器 | ~$179 |
| 電源 | Geekworm X1200 + 2× 18650 | 電池供電模組 | ~$30 |
| 螢幕 | 4 吋 HDMI 顯示器 | 即時取景 | ~$25 |
| 觀景窗 | Mamiya C220 TLR 腰平觀景窗 | 復古取景體驗 | 二手 ~$30–50 |
| 快門按鈕 | 瞬時開關（Momentary Switch） | GPIO26 觸發拍攝 | ~$2 |
| 機身 | Protopasta Steel PLA 3D 列印 | 金屬質感外殼 | ~$15 |
| 鏡頭 | Fujinon C-mount（25mm f/1.4、9mm f/1.4） | 拍攝鏡頭 | 二手 ~$30–80 |
| **合計** | | | **~$390–460** |

### 軟體功能

相機運行一個自訂的 Python 腳本，整合了以下功能：

- **PNG 影像擷取** — 無損格式保存
- **對焦峰值（Focus Peaking）** — 即時顯示合焦區域
- **即時直方圖** — 曝光輔助
- **手動快門速度控制** — 完整曝光控制
- **自動 ISO** — 智慧感光度調節
- **Wi-Fi 影像傳輸** — 拍完直接傳到手機

這些功能在 Malcolm 平常使用的 Sony A7IV 上都有，而他成功在一台 RPi 5 上全部復刻了。

## 技術亮點

### 一吋感測器的跨越

Raspberry Pi 官方的 HQ Camera 使用 1/2.3 吋的 Sony IMX477 感測器，雖然在 Pi 生態系已經算很好，但面積只有 IMX283 的約 **1/4**。OneInchEye V2 把 IMX283 一吋感測器封裝在一塊相容 RPi 的 PCB 上，透過 **4-lane MIPI-CSI** 介面連接，還附帶 IMU（ICM42688-P）和溫度感測器（TMP117）。

這塊開源感測器板由台灣的 [Will Whang](https://github.com/will127534/OneInchEye) 設計，在 Tindie 上販售，KiCad 設計檔案完全開源（MIT License）。

> 感測器面積大 → 進光量多 → 低光表現好 + 動態範圍大 + 淺景深

### Steel PLA 的教訓

Malcolm 選用 Protopasta Steel PLA 來列印外殼，這種線材含有金屬粉末，手感和視覺效果都很獨特。但他學到了一個慘痛教訓：**Steel PLA 是導電的**。他因此燒掉了一塊 Raspberry Pi。

解決方案：列印一個**標準 PLA 內殼**來隔離電子元件，再套上 Steel PLA 外殼。雙層結構反而讓相機更有質感。

### 腰平觀景窗的巧思

觀景窗是這台相機最有靈魂的部分。Malcolm 把一個 Mamiya C220 TLR 的腰平觀景窗拆下來，用摩擦配合固定在 3D 列印的頂板上。4 吋 HDMI 螢幕放在觀景窗下方，從上往下看就能看到即時取景畫面。

這種拍攝方式讓你自然地低頭看螢幕，被攝者不會注意到你在拍照——完美適合街拍。

### 鏡頭系統

感測器板預設使用 C-mount 接口。有人在 Hackaday 留言區批評 C-mount 鏡頭選擇有限且價格不合理，Malcolm 的回應是：他做了一個 **0.5× 焦距縮減器（Focal Reducer）**，支援 Pentax PK 和 M42 接口鏡頭。這意味著大量便宜的老鏡頭都能用上。

IMX283 的裁切係數約 2.7×，所以：
- 8mm 鏡頭 → 等效約 22mm（超廣角）
- 12mm → 等效約 32mm（人文廣角）
- 25mm → 等效約 68mm（人像中焦）

## 心得與延伸

### 為什麼這個專案值得關注

1. **它真的能拍照** — 不是概念驗證，Malcolm 帶著它去街拍、去棚拍，出片品質讓 PetaPixel（全球最大獨立攝影媒體）都專文報導。

2. **成本不到 $500** — 一台有一吋感測器的商用相機（如 Sony RX100 系列）要 $800–1300，這台 DIY 版本功能更客製化，價格只要一半。

3. **完全可復刻** — STL 檔案公開（CC BY-NC-SA 4.0）、感測器板開源（MIT）、Python 程式碼公開。你今天就可以開始做。

4. **Pi 生態系的 LEGO 效應** — 電池模組、螢幕、感測器板、鏡頭轉接環⋯⋯每個零件都是現成的，像樂高一樣組合。

### 可能的延伸方向

- **加入 RAW 格式支援** — IMX283 支援 12-bit 輸出，理論上可以儲存 DNG 格式
- **觸控螢幕觸碰對焦** — 換一塊觸控螢幕，實現點哪對哪
- **AI 場景辨識** — RPi 5 的算力足以跑輕量級模型，自動調整拍攝參數
- **影片錄製** — IMX283 支援影片輸出，可以擴展為 Vlog 相機
- **換用 RPi CM5** — 更小的體積、更強的算力，可以做出更緊湊的機身

### OneInchEye 開源生態

這個專案的核心——OneInchEye 感測器板——本身就是一個值得關注的開源硬體專案。它由台灣開發者 Will Whang 設計，PCB 設計檔（KiCad）、BOM、定位檔全部開源，甚至可以直接丟給 JLCPCB 做 PCBA。如果你有辦法自己買到 IMX283 感測器晶片，連感測器板都可以自己焊。

## 參考資料

- 🔗 原文（Reddit）：https://www.reddit.com/r/raspberry_pi/comments/1vb8uyf/took_my_diy_camera_for_street_photography_on_a/
- 📝 Build Guide：https://camerahacksbymalcolmjay.substack.com/p/build-guide-mini-hasselblad-style
- 📝 開發日誌：https://camerahacksbymalcolmjay.substack.com/p/building-a-camera-from-scratch
- 📰 PetaPixel 報導：https://petapixel.com/2025/08/28/this-awesome-diy-digital-camera-has-a-waist-level-viewfinder/
- 📰 Hackaday 報導：https://hackaday.com/2025/08/25/theres-nothing-mini-about-this-mini-hasselblad-style-cameras-sensor/
- 🔧 OneInchEye 開源感測器板：https://github.com/will127534/OneInchEye
- 🛒 OneInchEye V2（Tindie）：https://www.tindie.com/products/will123321/oneincheye-v20/
