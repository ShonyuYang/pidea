---
title: "14 歲少年的人形機器人夢 — RPi Zero 2W × 3D 列印打造 LitlMan"
date: 2026-07-15
draft: false
tags: ["raspberry-pi", "robot", "humanoid", "3d-printing", "servo", "python", "maker"]
categories: ["專案推薦"]
description: "14 歲的 Noah 花一個月從零設計 CAD、3D 列印、組裝伺服馬達，用 RPi Zero 2W 打造出會走路的人形機器人 LitlMan。"
---

## 前言

人形機器人不再是波士頓動力或特斯拉的專利。14 歲的 Noah 用一塊 Raspberry Pi Zero 2 W、一台 3D 印表機和一堆伺服馬達，花了一個月從零打造出名為 **LitlMan** 的人形機器人。沒有現成套件、沒有大學實驗室——只有 CAD 軟體、焊接鐵和無盡的除錯耐心。

這個專案之所以值得關注，不只是因為「14 歲做出人形機器人」這個標題夠吸睛，而是它完整展示了一個 Maker 從機構設計到軟體控制的全棧能力，並且把整個過程攤在社群面前，邀請所有人一起參與改進。

## 專案概述

### 硬體架構

| 元件 | 規格 | 用途 |
|------|------|------|
| 控制核心 | Raspberry Pi Zero 2 W | 主控制器、伺服驅動、軟體運行 |
| 驅動器 | 多顆伺服馬達 | 各關節動作控制 |
| 結構件 | 3D 列印（PLA/PETG） | 全部由 Noah 自行 CAD 設計 |
| 通訊 | Wi-Fi（板載） | 遠端控制與除錯 |

### 設計哲學

Noah 選擇 **全自主設計** 路線——不用現成的人形機器人框架（如 [Modular Biped](https://github.com/makerforgetech/modular-biped) 或商用 TonyPi），而是從頭在 CAD 軟體中設計每一個結構件，再 3D 列印組裝。這意味著他必須同時處理：

- **機構設計**：關節自由度配置、結構強度、重心分佈
- **電子整合**：伺服馬達佈線、電源分配、Pi Zero 的 GPIO 規劃
- **軟體控制**：伺服校準、步態演算法開發

### 目前進度

專案目前處於 **伺服校準與步態開發** 階段。Noah 正在研究如何讓 LitlMan 穩定行走，這也是人形機器人最困難的部分之一。

## 技術亮點

### 為什麼選 Pi Zero 2 W？

Pi Zero 2 W 是人形機器人控制器的有趣選擇。它的四核 Cortex-A53 @ 1GHz 提供了比 Arduino 更強的運算能力（可以跑 Python、甚至輕量 ML 模型），但功耗只有約 0.4W 待機 / 1.8W 滿載，對電池供電的機器人來說至關重要。

不過，Pi Zero 2 W 的 GPIO 並不原生支援硬體 PWM 多通道輸出。要控制多顆伺服馬達，通常需要：

1. **PCA9685 I²C 伺服驅動板**：16 通道 PWM，透過 I²C 只佔兩個 GPIO
2. **pigpio 軟體 PWM**：利用 DMA 產生精確的 PWM 信號
3. **gpiozero 的 Servo 類別**：Noah 的 GitHub 顯示他使用了這個方案

```python
from gpiozero import Servo
servo = Servo(pin, initial_value=0, 
              min_pulse_width=0.5/1000,  # 500μs
              max_pulse_width=2.5/1000)  # 2500μs
servo.value = 0.5  # -1 到 1 之間
```

### 步態演算法的挑戰

讓雙足機器人走路是機器人學中經典的難題。核心挑戰在於：

- **靜態穩定性**：重心投影必須落在支撐腳的範圍內
- **動態穩定性**：行走時重心會不斷移動，需要即時補償
- **ZMP（Zero Moment Point）控制**：確保翻倒力矩為零

對於小型伺服驅動的人形機器人，常見的入門策略是：

1. **預錄關鍵影格**：手動調整每個關節角度，錄製一組行走動作序列
2. **正弦波步態**：用不同相位的正弦波驅動各關節，產生週期性步態
3. **逆運動學（IK）**：計算腳掌目標位置對應的各關節角度

社群中有人建議 Noah 試試 **PiZZA**（一個 Pi Zero 的即時作業系統），因為 Linux 的非即時排程可能導致伺服控制的 jitter，影響步態穩定性。

### Noah 的學習路徑

從 Noah 的 [GitHub](https://github.com/NoahMSchool/RaspberryPiEpq) 可以看到他的學習軌跡：

1. 基礎 GPIO 控制（LED 閃爍）
2. PWM 與伺服馬達控制
3. ADC 與感測器讀取（I²C 協定）
4. 多伺服同步控制
5. Godot 模擬器建模（先在虛擬環境測試）
6. OpenSCAD 模組化結構設計

這個從簡到繁的學習路徑，本身就是一份很好的「如何從零開始做機器人」教材。

## 心得與延伸

### 這個專案教會我們什麼

**年齡不是限制，工具才是。** 十年前，一個 14 歲的孩子要做人形機器人，光是機構加工就是不可能的任務。但現在有了平價 3D 印表機（Ender-3 不到 200 美元）、免費 CAD 軟體（Fusion 360 教育版、OpenSCAD）、和 15 美元的 Pi Zero 2 W，硬體門檻已經低到可以在臥室裡完成。

**公開學習的勇氣。** Noah 把半成品丟到 Reddit，直接說「我需要步態演算法的建議」。這種態度比技術本身更值得學習——在 Maker 社群，展示失敗和求助往往比展示完美作品得到更多有價值的回饋。

### 可能的改進方向

如果 Noah 想讓 LitlMan 走得更穩，以下是一些社群建議的方向：

| 改進 | 難度 | 說明 |
|------|------|------|
| 加裝 IMU（MPU6050） | ⭐⭐ | 即時感測傾斜角度，回饋補償 |
| 換用 PCA9685 驅動板 | ⭐ | 釋放 GPIO、更精確的 PWM |
| 加大腳掌面積 | ⭐ | 增加支撐多邊形面積，提升靜態穩定性 |
| 嘗試 PiZZA RTOS | ⭐⭐⭐ | 消除 Linux 排程 jitter |
| 導入 MicroPython + Pico 副控 | ⭐⭐ | Pi 負責高階邏輯，Pico 負責即時伺服控制 |

### 類似專案比較

| 專案 | 控制器 | 特色 |
|------|--------|------|
| **LitlMan** | Pi Zero 2 W | 全自主 CAD 設計、青少年 Maker |
| [Modular Biped](https://github.com/makerforgetech/modular-biped) | Pi + Arduino | 開源框架、模組化設計 |
| [Bipedal Companion](https://www.thingiverse.com/thing:6784507) | Pi + Arduino | 開源、自主導航 |
| TonyPi (Hiwonder) | Pi 5 | 商用套件、AI 視覺 |

LitlMan 的獨特價值在於它是 **從零開始的學習記錄**，而不是套件組裝或框架修改。

### 估算成本

| 項目 | 估計費用 |
|------|----------|
| Raspberry Pi Zero 2 W | ~$15 |
| 伺服馬達 ×8-12 顆 | ~$30-60 |
| PCA9685 驅動板 | ~$5 |
| 3D 列印耗材 | ~$10-15 |
| 電池與電源模組 | ~$10-15 |
| 螺絲、跳線等雜料 | ~$5-10 |
| **總計** | **~$75-120** |

一百美元以內就能開始你的人形機器人之旅。

## 參考資料

- [Reddit 原文 — Meet LitlMan](https://www.reddit.com/r/raspberry_pi/comments/1usgism/meet_litlman_my_raspberry_pi_humanoid_robot/)
- [Noah 的 GitHub — RaspberryPiEpq](https://github.com/NoahMSchool/RaspberryPiEpq)
- [Modular Biped 開源框架](https://github.com/makerforgetech/modular-biped)
- [gpiozero Servo 文件](https://gpiozero.readthedocs.io/en/stable/api_output.html)
- [Bipedal Walking Robot 步態教學](https://zbotic.in/bipedal-walking-robot-servo-layout-and-gait-programming/)
