---
title: "不靠 Linux 也能 3D 渲染！大學生在 RPi Zero W 上打造裸機高斯散射渲染器"
date: 2026-06-15
draft: false
tags: [Raspberry Pi, Bare-Metal, Gaussian Splatting, ARMv6, 3D Rendering, OS Development]
categories: ["專案推薦"]
description: "大學生從零打造 ARMv6 裸機作業系統，在沒有 Linux、沒有 GPU 驅動的 Raspberry Pi Zero W 上實現即時 3D 高斯散射渲染。"
---

## 前言

當大多數人想到 3D 渲染，腦中浮現的是高階 GPU、數十 GB 的顯存、以及龐大的軟體堆疊。但如果告訴你，有人在一塊售價不到 300 元台幣的 Raspberry Pi Zero W 上，**完全不依賴 Linux 作業系統**，就實現了近年電腦圖學界最火紅的 Gaussian Splatting 3D 渲染技術呢？

這個來自大學電腦科學課程的專案，不只是一個渲染器 — 它是一整套從零打造的裸機（Bare-Metal）ARMv6 作業系統，搭配完整的軟體浮點運算與 3D 渲染管線。

## 專案概述

**Baremetal Gaussian Splat Renderer** 是由 GitHub 用戶 jkasalt 與團隊成員在大學課程中完成的專案。他們的目標很明確也很瘋狂：在 Raspberry Pi Zero W（BCM2835 SoC，單核 ARM1176JZF-S @ 1GHz）上，不借助任何作業系統，直接在硬體上跑 Gaussian Splatting 渲染。

整個系統架構包含：

- **自製裸機運行環境**：從 ARM 組合語言的 boot code 開始，建立最小化的 ARMv6 執行環境
- **自訂記憶體配置器**：沒有 OS 提供的 malloc，一切記憶體管理自己來
- **Mailbox 介面的 Framebuffer 設定**：透過 VideoCore GPU 的 mailbox 協定取得畫面緩衝區
- **UART 除錯支援**：在沒有螢幕輸出的開發階段，靠串列埠 log 來除錯
- **完整的 3D 高斯散射渲染管線**：軟體實作的 splatting 演算法

專案原始碼以 MIT 授權釋出，編譯只需要 ARM 交叉編譯工具鏈（arm-none-eabi-gcc），產出的 `kernel.img` 直接放上 SD 卡就能開機執行。

## 技術亮點

### 🔥 沒有硬體浮點運算單元

RPi Zero W 的 ARM1176JZF-S 核心**不具備硬體 FPU**。這意味著 Gaussian Splatting 中大量的浮點數學運算 — 包括高斯分佈計算、矩陣變換、深度排序 — 全部必須透過軟體模擬完成。團隊在 `src/math.c` 中實作了完整的軟體浮點數學函式庫。

### 🧠 單核心即時渲染

沒有 GPU 計算能力可用，所有渲染工作都跑在那顆 1GHz 的單核心上。要在這樣的限制下排序並渲染數千個 splat，對演算法優化的要求極高。

### 🛠️ 從 Boot Code 到 Framebuffer

專案的程式碼結構清楚展示了裸機開發的完整面貌：

| 檔案 | 功能 |
|------|------|
| `src/boot.S` | ARM 組合語言開機程式碼 |
| `src/main.c` | 主程式進入點 |
| `src/gpu.c` | Framebuffer 與 mailbox 介面 |
| `src/splat.c` | 高斯散射渲染器核心 |
| `src/math.c` | 軟體浮點數學運算 |
| `src/alloc.c` | 記憶體配置器 |

### 🎥 有影片為證

團隊也釋出了實際運行的展示影片（YouTube），可以看到 Pi Zero W 上即時渲染 3D 高斯散射場景的效果。

## 心得與延伸

這個專案在 Reddit r/raspberry_pi 社群獲得了 352 個讚、99% 的好評率，留言區的討論也非常精彩：

> 「沒有硬體 FPU 還能做到即時渲染？太瘋狂了。每幀能渲染多少個 splat？」

> 「你們為了渲染高斯散射而自己寫了一個 OS，這既瘋狂又令人敬佩。」

這個專案完美展示了幾個重要的學習方向：

1. **裸機開發的魅力**：拿掉 OS 這層抽象之後，你必須真正理解硬體如何運作
2. **限制激發創意**：在 1GHz 單核、無 FPU、無 GPU 的極端限制下，反而逼出了對演算法和系統架構的深度理解
3. **Gaussian Splatting 的可移植性**：這項技術不一定需要高階硬體，概念上可以在任何有 framebuffer 的系統上實現

如果你對裸機開發、嵌入式系統、或是 3D 渲染技術有興趣，這個專案的原始碼非常值得一讀。

## 參考資料

- 🔗 [GitHub 原始碼](https://github.com/jkasalt/rpi-baremetal-gaussian-splat)
- 🎬 [展示影片 (YouTube)](https://www.youtube.com/watch?v=MQBINyJqFRw)
- 💬 [Reddit 原文討論](https://www.reddit.com/r/raspberry_pi/comments/1tz4et2/baremetal_gaussian_splat_renderer/)
- 📖 [Gaussian Splatting - Wikipedia](https://en.wikipedia.org/wiki/Gaussian_splatting)
