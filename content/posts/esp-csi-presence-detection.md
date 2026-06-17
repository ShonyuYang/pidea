---
title: "WiFi 就是你的感測器 — ESP32 CSI 人體存在偵測完全指南"
date: 2026-06-17
draft: false
tags: ["ESP32", "WiFi CSI", "人體偵測", "智慧家居", "Home Assistant", "ESPHome"]
categories: ["專案推薦"]
description: "不用 PIR、不用雷達、不用攝影機 — 一顆 ESP32 就能感知房間裡有沒有人。WiFi CSI 技術讓無線信號本身成為最隱形的存在感測器。"
---

## 前言

智慧家居的「存在偵測」一直是個痛點。PIR 感測器便宜但只能偵測移動，你坐在沙發上看書它就當你不存在；mmWave 雷達精準但一顆要 $15–30；攝影機什麼都看得到，但你真的想在臥室裝一顆嗎？

如果我說，你家路由器每秒鐘發出的 WiFi 信號，本身就能當感測器呢？

這就是 **WiFi CSI（Channel State Information）** 的核心概念：分析 WiFi 子載波的振幅與相位變化，來判斷空間中是否有人體存在。而 Espressif 官方的 [esp-csi](https://github.com/espressif/esp-csi) 專案，讓這件事在 $5 的 ESP32 上就能實現。

## 專案概述

### 什麼是 CSI？

大多數人熟悉 RSSI（信號強度指標），但 RSSI 只告訴你信號「有多大聲」。CSI 則完全不同 — 它捕捉的是信號的「紋理」。

現代 WiFi 使用 OFDM 調變，資料同時透過多個子載波（不同頻率）傳送。當這些無線電波從發射端到接收端，會被牆壁、家具、人體反射，產生干涉圖案。CSI 記錄每個子載波的振幅和相位資訊 — 當有人走過房間，這些反射模式就會改變。

簡單來說：**RSSI 是音量，CSI 是音色。**

### 硬體需求

| 元件 | 用途 | 參考價格 |
|------|------|----------|
| ESP32 系列開發板 ×2 | 一個發射、一個接收 | ~$5–8/顆 |
| 外接偶極天線（建議） | 提升信號品質 | ~$1–2 |
| 三腳架或固定支架 | 離地 1.2–1.5m 架設 | 自備 |

**支援晶片排名**（CSI 效能由高到低）：
ESP32-C5 ＞ ESP32-C6 ＞ ESP32-C3 ≈ ESP32-S3 ＞ ESP32

總成本可以壓在 **$10–15 美元**以內，這是 mmWave 雷達模組價格的一半不到。

### 軟體生態

- **[esp-csi](https://github.com/espressif/esp-csi)**：Espressif 官方倉庫，包含 `csi_send`、`csi_recv`、`console_test` 等範例，以及 `esp_wifi_sensing` 狀態機元件
- **[ESPectre](https://github.com/ssieb/espectre)**：ESPHome 外部元件，一鍵整合 Home Assistant
- **[esp32_csi_human_presence](https://github.com/mzakharo/esp32_csi_human_presence)**：社群專案，直接暴露 HA binary sensor
- **[ESPresense](https://espresense.com/)**：基於 BLE 的房間級定位（互補方案）

## 技術亮點

### 1. 「隱形絆線」偵測模式

Hackster.io 上的 [深度實作文章](https://www.hackster.io/limengdu0117/esp-csi-diy-wifi-human-presence-detection-f80508) 詳細記錄了從失敗到成功的過程。作者用 Seeed Studio XIAO ESP32S3 搭建偵測陣列，發現關鍵不在軟體而在**物理擺放**：

- ❌ 把板子平放桌面 → 天線反射全被桌面吃掉，只看到雜訊
- ✅ 架高到 1.4m + 外接偶極天線 → 數據立刻乾淨，人體走過清晰可見

這就像在兩個 ESP32 之間拉了一條隱形的雷射線 — 有人穿過，波形立刻劇烈變化。

### 2. 從單線到陣列

單對收發器只能偵測兩點之間的「線」，盲區很大。解法是用 4 顆 ESP32 組成感測陣列：

```
    [TX] ←── 東北角
     ↙  ↘
  [RX2]  [RX3] ←── 牆壁中點
     ↘  ↙
    [RX1] ←── 西南角（對角線）
```

三條偵測線交叉覆蓋，就能把一整個房間變成感測區域。

### 3. 不只偵測移動，還能感知靜止

CSI 最厲害的地方在於它的靈敏度 — 不只能偵測走動，連**呼吸、咀嚼**這種微小動作都能感知。Espressif 官方甚至展示過 `esp-crab` 範例，用純 WiFi 信號做手指手勢辨識。

這意味著它理論上能解決 PIR 感測器最大的痛點：偵測靜止不動的人。

### 4. 零額外硬體成本的 OTA 升級

如果你家裡已經有 ESP32 跑其他 IoT 任務（溫濕度監控、LED 控制等），只要韌體支援，可以透過 OTA 升級直接加入 CSI 功能，完全不需要額外硬體。

### 5. Home Assistant 整合

透過 ESPectre 或社群專案，CSI 偵測結果可以直接暴露為 Home Assistant 的 binary sensor，搭配自動化實現：

- 人進房間 → 自動開燈
- 離開超過 5 分鐘 → 關閉空調
- 穿牆偵測 → 不同房間的聯動控制

## 心得與延伸

### 誠實面對：現階段的限制

讀完 Hackster.io 那篇深度文章後，作者的結論很誠實：**ESP-CSI 是一個令人驚嘆的 hack，但目前還太挑剔、太耗電，無法取代雷達模組。**

具體挑戰包括：

1. **環境敏感**：天線擺放角度、高度、周圍家具都會劇烈影響效果，需要現場校準（on-site calibration）
2. **功耗較高**：WiFi 收發持續運作，不適合電池供電場景
3. **演算法門檻**：從原始 CSI 數據到可靠的「有人/沒人」判斷，需要相當的信號處理知識
4. **穩定性**：WiFi 環境本身就是動態的，鄰居的路由器、微波爐都可能造成干擾

### 適合誰？

- 🎓 **學習者**：這是理解 OFDM、多路徑傳播、信號處理的絕佳實作專案
- 🔬 **研究者**：CSI 在室內定位、手勢辨識、健康監測領域有大量論文可以復現
- 🏠 **智慧家居玩家**：如果你願意花時間調校，它確實能提供比 PIR 更好的存在偵測
- 💰 **預算有限者**：$10 的成本讓它成為最便宜的存在偵測方案

### 延伸方向

- **CSI + mmWave 融合**：用 CSI 做粗略偵測，mmWave 做精確定位，互補優缺點
- **機器學習**：ESP32-S3 支援 AI 指令集，可以在裝置端跑輕量級 CNN 做活動辨識
- **多房間拓撲**：利用既有的 WiFi mesh 網路，每個節點同時當 CSI 感測器
- **穿牆偵測**：CSI 信號能穿牆，理論上一組設備就能監控相鄰房間

## 參考資料

- [ESP-CSI GitHub（Espressif 官方）](https://github.com/espressif/esp-csi)
- [ESP-CSI: DIY WiFi Human Presence Detection — Hackster.io 深度實作](https://www.hackster.io/limengdu0117/esp-csi-diy-wifi-human-presence-detection-f80508)
- [ESP-CSI 技術文件 — Espressif](https://docs.espressif.com/projects/esp-techpedia/en/latest/esp-friends/solution-introduction/esp-csi/esp-csi-solution.html)
- [Human Presence using WiFi Sensing CSI on ESP32 — Home Assistant 社群](https://community.home-assistant.io/t/human-presence-using-wifi-sensing-csi-on-esp32/791452)
- [ESPresense — 基於 BLE 的房間級定位](https://espresense.com/)
- [Reddit 討論串 — ESP32 WiFi CSI 人體偵測](https://www.reddit.com/r/esp32/comments/1qe4qk8/espcsi_diy_wifi_human_presence_detection/)
