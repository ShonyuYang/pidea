---
title: "用 Raspberry Pi 和光束分裂器打造桌上全息顯示器"
date: 2026-06-01
draft: false
tags: ["raspberry-pi", "hologram", "3d-printing", "pepper-ghost", "display"]
categories: ["專案推薦"]
description: "一顆 7cm 的光束分裂器立方體、一塊方形螢幕、一台 Pi，就能在桌上變出漂浮影像——這是 19 世紀劇場魔術的 DIY 復刻。"
---

## 前言

全息投影聽起來像科幻片的專利，但其實背後的原理——**佩珀爾幽靈（Pepper's Ghost）**——早在 1862 年就在倫敦劇院驚豔觀眾了。核心概念極其簡單：一片透明玻璃 + 一個明亮光源，就能讓影像看起來「漂浮」在空中。

Reddit 使用者 silvercoated1 在待業期間把這個 160 年前的劇場把戲搬上了桌面，用 Raspberry Pi 4、一塊 4 吋方形螢幕和一顆光束分裂器立方體（beam splitter cube），做出了一台可以當媒體播放器、時鐘、甚至未來跑 Doom 的全息顯示器。

## 專案概述

### 核心架構

這個專案的硬體堆疊意外地簡潔：

| 元件 | 規格 | 用途 |
|------|------|------|
| 運算核心 | Raspberry Pi 4/5 | 跑顯示程式 |
| 螢幕 | HyperPixel 4.0 Square（4 吋觸控） | 投射影像源 |
| 光學元件 | 70mm 光束分裂器立方體 | 產生全息反射效果 |
| 外殼 | 3D 列印件 | 固定螢幕與立方體的相對位置 |
| 電源 | 5V/5A USB-C | 供電 |
| 相機（選配） | Pi Camera Module 3 | 手勢互動 |

光束分裂器立方體是整個專案的靈魂——它不像傳統 Pepper's Ghost 用一片傾斜的壓克力板，而是用一顆實心的分光立方體坐在螢幕正上方，讓反射影像看起來懸浮在立方體內部。效果比斜板方案更立體、更有「科幻感」。

### 軟體

作者使用的是開源專案 [OpenGhost](https://github.com/xanderchinxyz/OpenGhost)，基於 Python 的 **py5** 圖形庫（Processing 的 Python 移植版），搭配 PyOpenGL 渲染 3D 線框圖形。目前 repo 內含幾個展示程式：

- **boids.py** — 群體行為模擬（鳥群演算法）
- **boids_finger_tracking.py** — 加上手指追蹤的互動版本
- **lorenz_attractor.py** — 洛倫茲吸引子的混沌視覺化

## 技術亮點

### 1. 光束分裂器 vs 傳統 Pepper's Ghost

傳統做法是把一片 45° 傾斜的透明壓克力板放在螢幕前方，影像反射到觀眾眼中。缺點是：視角受限、需要遮住邊框、壓克力板容易刮花。

光束分裂器立方體（beam splitter cube）是兩片三角稜鏡膠合而成的光學元件，內部鍍有半反射膜。放在螢幕上方時，螢幕光線一半穿透、一半反射，觀眾從任何角度都能看到「漂浮」在立方體中的影像。50mm 和 70mm 兩種尺寸在 AliExpress 上都買得到，價格約 $10-30 美元。

### 2. 手勢互動

搭配 Pi Camera Module 3 和 MediaPipe 手部追蹤，可以用手指控制立方體內的 3D 物件旋轉。OpenGhost 的 `boids_finger_tracking.py` 已經實作了這個功能——手指移動會影響 boids 群體的行為方向。

### 3. 完全開源 + 3D 列印

所有 STL 檔案都在 GitHub repo 的 `/stl_files` 目錄下，用一般 FDM 印表機就能印。搭配 4 顆 M2.5x14mm 螺絲就能組裝完成，不需要任何額外的機械加工。

## 心得與延伸

這個專案最吸引人的地方不是技術難度——它真的不高——而是**視覺衝擊力與成本的反差**。一顆 $15 的光束分裂器立方體加上你可能已經有的 Pi，就能做出讓朋友以為你花了幾千塊的效果。

### 可以怎麼玩

- **AI 助手介面**：作者自己提到想做一個 Power Rangers Zordon 風格的 AI 臉部顯示，搭配本地 LLM 跑語音對話
- **Winamp 視覺化器**：音樂視覺化效果在全息立方體裡會特別酷
- **智慧家庭 HUD**：顯示天氣、行事曆、通知，放在桌上就是一個未來感十足的桌面小物
- **教育展示**：分子結構、太陽系模型、數學曲面——全息效果讓抽象概念變得可觸摸

### 進階改造方向

如果想要更強的 3D 效果，可以參考 [HoloForge Particles](https://github.com/nathan-ortiz/holoforge-particles) 專案，它用 PyOpenGL 渲染發光線框加粒子光暈，視覺效果更接近科幻電影裡的全息投影。

另一個有趣的方向是用 **四面 Pepper's Ghost**（金字塔形結構）取代單面立方體，讓四個方向都能看到全息影像。不過光束分裂器立方體的優勢在於體積小、光學品質好、不怕碰撞。

## 參考資料

- [Reddit 原文](https://www.reddit.com/r/raspberry_pi/comments/1tsa7bb/cube_holographic_display_using_pi_4_and_beam/)
- [OpenGhost GitHub](https://github.com/xanderchinxyz/OpenGhost)（⭐ 269）
- [XDA Developers 報導](https://www.xda-developers.com/this-awesome-raspberry-pi-project-uses-the-peppers-ghost-illusion-to-make-a-holographic-display/)
- [HoloForge Particles](https://github.com/nathan-ortiz/holoforge-particles) — 進階全息粒子效果
