---
title: "不用買示波器 — STM32 一片板子就能看波形"
date: 2026-08-20
draft: false
tags: ["STM32", "示波器", "ADC", "DMA", "嵌入式", "DIY"]
categories: ["專案推薦"]
description: "用一片 STM32F207 Nucleo 開發板，不靠外部 ADC 晶片，直接用內建 ADC + DMA 做到 2 MSPS 取樣，搭配 Python GUI 即時顯示波形。"
---

## 前言

學嵌入式的人遲早會需要一台示波器，但入門款 Rigol DS1054Z 動輒台幣八千起跳，對學生或業餘玩家來說不是小數目。如果你手邊剛好有一片 STM32 Nucleo 板子，其實不用額外花錢 — MCU 內建的 ADC 就能當一台堪用的單通道示波器，而且整個專案從韌體到 PC 端 GUI 全部開源。

[BTTLab] 的這個專案用 STM32F207 的內建 12-bit ADC 達到 **2 MSPS** 取樣率，搭配 Python + matplotlib 前端即時畫波形。雖然跟專業示波器的 1 GSPS 差了三個數量級，但對低頻電路除錯、UART/I²C 協定分析、PWM 波形觀察來說綽綽有餘。

## 專案概述

### 硬體需求

| 元件 | 規格 | 備註 |
|------|------|------|
| 開發板 | NUCLEO-F207ZG | STM32F207 Cortex-M3 |
| 類比輸入 | PA6 (ADC1 CH6) | ⚠️ 僅接受 0–3.3V |
| PC 連接 | ST-LINK 虛擬 COM port | 不需額外 USB 轉 UART |
| 輸入保護 | Schottky 二極體箝位電路 | **必備**，否則燒 MCU |

整個硬體就是一片 Nucleo 板加上輸入保護電路，成本壓在台幣 500 元以內（如果你已經有板子的話，成本是零）。

### 軟體架構

```
訊號 → PA6 → ADC1 (TIM2 觸發) → DMA 環形緩衝區
                                      ↓
                              觸發偵測 (rising edge + 遲滯)
                                      ↓
                              UART DMA → PC Python GUI
```

**關鍵設計：CPU 完全不參與取樣。** TIM2 以固定間隔觸發 ADC 轉換，DMA 自動把結果搬進環形緩衝區，CPU 只在 half/full transfer 中斷時做觸發偵測和資料傳輸。

### 取樣規格

- **取樣率：** 2 MSPS（百萬次/秒）
- **解析度：** 12-bit（4096 階）
- **UART 鮑率：** 921600 baud（DMA 非阻塞傳輸）
- **觸發模式：** Rising edge，帶遲滯抗雜訊
- **Pre-trigger：** 支援觸發前波形擷取

## 技術亮點

### 1. 硬體定時取樣 — 不是軟體迴圈

很多 Arduino 示波器專案用 `analogRead()` 配 `delay()` 來取樣，時間精度完全靠 CPU 排程，抖動（jitter）嚴重。BTTLab 的做法是正統的：

- **TIM2** 產生固定頻率的觸發訊號
- ADC 設定為外部觸發模式，每收到一個 TIM2 事件就轉換一次
- CPU 完全不介入時序控制

這確保了取樣間隔的一致性，是做出可靠示波器的基本功。

### 2. 雙緩衝 DMA — 零丟失連續取樣

DMA 環形緩衝區分成前半和後半，利用 Half Transfer Complete 和 Full Transfer Complete 兩個中斷，實現類似雙緩衝的效果：一半在寫入新資料，另一半在處理和傳輸。這是 STM32 DMA 的經典用法，在音訊處理、資料記錄等場景都能直接套用。

### 3. 軟體觸發帶遲滯

觸發偵測不是單純比較「大於閾值」，而是加了遲滯（hysteresis）：訊號必須先低於下限、再升過上限才算一次有效觸發。這能有效過濾雜訊造成的假觸發，是示波器設計的基本但重要的細節。

### 4. ST HAL 移植性

韌體基於 ST HAL 撰寫，理論上可以移植到其他 STM32 系列。不過要注意：

- **STM32F4** 系列的 ADC 通常更快（可達 2.4 MSPS）
- **STM32H7** 系列支援 16-bit ADC，解析度更高
- **STM32F1** 系列（如常見的 F103）只有 1 MSPS，但仍堪用
- 每個系列的 ADC 時序、DMA 通道映射都不同，務必讀 datasheet

### 5. 系列化教學 — 三通道版本已上線

BTTLab 已經發布了 [三通道版本 stm32osc3ch](https://github.com/BTTLab/stm32osc3ch)，使用 STM32F207 的三個 ADC 在 **Triple Regular Simultaneous Mode** 下同時取樣三個通道，適合需要比較多組訊號的場景。

## 心得與延伸

### 這台「示波器」能做什麼？

老實說，2 MSPS、0–3.3V 輸入範圍的規格很有限。但以下場景完全夠用：

- **UART/SPI/I²C 協定除錯**（時脈通常在 MHz 以下）
- **PWM 波形觀察**（馬達控制、LED 調光）
- **感測器訊號檢查**（溫度、光線、加速度計的類比輸出）
- **電源紋波初步檢查**（配合適當的衰減電路）

### 不能做什麼？

- **RF 訊號分析** — 頻寬不夠
- **負電壓量測** — 需要外部偏移電路（影片有提到但未深入）
- **高壓量測** — 需要探棒和衰減電路
- **邏輯分析** — 雖然 12-bit ADC 可以看數位訊號，但專用邏輯分析儀更適合

### 可能的改進方向

1. **加入類比前端**：運算放大器做偏移和衰減，支援 ±10V 輸入
2. **換用 STM32H7**：16-bit ADC + 更高取樣率 + 乙太網路串流
3. **交錯取樣（Interleaved Mode）**：STM32F207 支援雙 ADC 交錯，理論上可達 6 MSPS
4. **換掉 matplotlib**：用 PyQt + pyqtgraph 做更流暢的即時顯示
5. **加入 FFT 頻譜分析**：Python 端加 `numpy.fft` 就能做基本頻譜顯示

### 學習價值

這個專案最大的價值不是做出一台多厲害的示波器，而是一次把 STM32 的核心周邊都練過一遍：

- **ADC** — 外部觸發、解析度設定、取樣時間
- **DMA** — 環形緩衝區、雙緩衝、中斷回調
- **Timer** — 觸發源產生、頻率計算
- **UART** — 高速非阻塞傳輸
- **中斷管理** — 優先級設定、ISR 設計

這些技能在任何嵌入式開發工作中都用得到。

## 參考資料

- 🎬 [BTTLab YouTube 影片](https://youtu.be/fAIbFj_k99g)
- 💻 [GitHub: stm32osc1ch（單通道版）](https://github.com/BTTLab/stm32osc1ch)
- 💻 [GitHub: stm32osc3ch（三通道版）](https://github.com/BTTLab/stm32osc3ch)
- 📰 [Hackaday 報導](https://hackaday.com/2026/08/19/simple-diy-stm32-oscilloscope-project/)
- 📖 [Pig-O-Scope — 早期 STM32 示波器先驅](http://github.com/pingumacpenguin/STM32-O-Scope/wiki)
