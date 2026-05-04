---
title: "不用攝影機也能偵測物體？BLE Channel Sounding 的奇妙應用"
date: 2026-05-04
draft: false
tags: ["BLE", "nRF54L15", "機器學習", "Channel Sounding", "Nordic", "物體偵測"]
categories: ["專案推薦"]
description: "用兩片 nRF54L15 開發板和 BLE 5.4 Channel Sounding，搭配 SVM 分類器實現無攝影機物體偵測與手勢辨識。"
---

## 前言

提到物體偵測，大多數人第一反應是攝影機加上電腦視覺。但如果告訴你，兩片藍牙開發板放在鞋盒裡，就能偵測盒子裡有沒有蘋果、甚至判斷你的手臂往左還是往右，你會不會覺得很瘋狂？

這個專案利用了 BLE 5.4 新引入的 **Channel Sounding (CS)** 功能，把原本設計給測距用的無線電訊號，變成了一個低成本的「雷達」系統。

## 專案概述

### 硬體配置

- **2 × Nordic nRF54L15 開發板** — 一片發射、一片接收
- **鞋盒** — 沒錯，就是一個普通鞋盒當作實驗環境
- CS 測量間隔約 **50ms**，即每秒 20 次掃描

### 運作原理

BLE Channel Sounding 和一般藍牙通訊不同 — 裝置之間交換的不是資料封包，而是**純正弦波**。接收端會測量 **72 個頻率通道**上的相位偏移和訊號衰減。

當環境中有物體存在時，無線電波會被反射、吸收或散射，導致這些相位和振幅數據產生變化。作者的關鍵洞察是：**這些變化雖然人類難以解讀，但機器學習模型可以從中找到規律。**

### 軟體架構

- **Python 工具鏈**：開源專案 [waves](https://github.com/skig/waves)
- **ML 模型**：scikit-learn 的 SVM 分類器
- **特徵向量**：每次測量產生 72 個通道 × (相位 + 衰減) 的數據矩陣

## 技術亮點

### Channel Sounding ≠ 一般 BLE

> Unlike "normal" BLE communication where devices exchange bytes of data, in the CS procedures devices exchange pure unmodulated sine waves.

這是 BLE 5.4 規範中相對冷門的功能，原本的設計目的是精確測距（例如車鑰匙防中繼攻擊）。但作者反其道而行，把測距副產品當成感測數據來用，這種思路非常有創意。

### 72 通道 = 72 個「感測器」

每次 CS 測量掃描 72 個頻率通道，每個通道提供相位和振幅兩個維度的資訊。這等於用一對藍牙晶片，同時獲得了 144 維的環境感測數據，資訊密度相當可觀。

### 誠實面對限制

作者很坦誠地指出了目前的瓶頸：

> The approach is very unreliable and very sensitive to small changes in the surroundings... it mainly happens because radio waves may reflect off surrounding surfaces.

多路徑傳播（multipath propagation）是無線電感測的老問題。環境中任何微小變化 — 人走過、門開關 — 都可能影響結果。這也是為什麼目前只能在受控環境（鞋盒）中展示。

## 心得與延伸

### 為什麼這個專案值得關注？

1. **成本極低**：兩片開發板，不需要攝影機、LiDAR 或任何額外感測器
2. **隱私友善**：不拍照、不錄影，純粹用無線電波感測
3. **BLE 5.4 的新可能**：Channel Sounding 目前主要用於測距，這個專案展示了完全不同的應用方向

### 可能的改進方向

- **更強的 ML 模型**：目前只用了 SVM，如果換成 CNN 或 Transformer 處理時序數據，辨識準確率可能大幅提升
- **多對裝置**：用 3-4 對開發板從不同角度測量，可以緩解多路徑問題
- **Edge ML**：把模型部署到 nRF54L15 上（Cortex-M33，支援 TFLite Micro），實現完全離線的感測系統
- **智慧家庭應用**：房間佔用偵測、跌倒偵測等場景，不需要攝影機就能保護隱私

### 相關技術比較

| 技術 | 成本 | 隱私 | 精確度 | 範圍 |
|------|------|------|--------|------|
| BLE CS 感測 | 低 | 高 | 低 | 短距 |
| mmWave 雷達 | 中 | 高 | 高 | 中距 |
| 攝影機 + CV | 中 | 低 | 很高 | 中距 |
| UWB 感測 | 中 | 高 | 中 | 中距 |

BLE CS 的優勢在於：**幾乎所有未來的藍牙裝置都會支援 5.4**，這意味著感測能力可以「免費」搭載在既有的藍牙基礎設施上。

## 參考資料

- 原文討論：[Reddit - BLE Channel Sounding for Object Detection](https://www.reddit.com/r/embedded/comments/1t2yj8j/ble_channel_sounding_for_object_detection_and/)
- 開源工具：[skig/waves - GitHub](https://github.com/skig/waves)
- [Nordic nRF54L15 產品頁](https://www.nordicsemi.com/Products/nRF54L15)
- [Bluetooth SIG - Channel Sounding 介紹](https://www.bluetooth.com/learn-about-bluetooth/feature-enhancements/channel-sounding/)
