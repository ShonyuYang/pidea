---
title: "不切書脊也能掃全本 — RPi 氣動翻頁掃書機 Kiro 實測紀錄"
date: 2026-08-16
draft: false
tags: ["Raspberry Pi", "書籍數位化", "OCR", "自動化", "3D 列印"]
categories: ["專案推薦"]
description: "一台用 Raspberry Pi 3B+、Camera Module 3 Wide 和氣吸翻頁機構打造的自動掃書機，300 頁只要 3.5 小時，不用拆書就能產出文字 EPUB。"
---

## 前言

「掃書最痛苦的不是掃描本身，而是翻頁。」

作者 adldotori 在電子書社群做了一輪調查，發現阻止人們把實體書數位化的最大理由不是設備、不是軟體，而是**手動翻幾百頁實在太累了**。市面上的商用自動掃書機（例如 Treventus）動輒數十萬台幣起跳，一般人根本碰不起。

於是他用一片 Raspberry Pi 3B+、一顆廣角鏡頭、幾個伺服馬達和一組氣吸翻頁機構，打造了名為 **Kiro** 的家用掃書機原型——而且已經在韓國三戶家庭進行 7 天免費試用。

## 專案概述

### 硬體架構

| 元件 | 規格 |
|---|---|
| 主控板 | Raspberry Pi 3 Model B+ |
| 鏡頭 | Camera Module 3 Wide（廣角版） |
| 翻頁機構 | 伺服馬達驅動臂 + 氣吸（vacuum suction）翻頁 |
| 機身 | 3D 列印 + 結構件 |

### 工作流程

```
放書 → 氣吸翻頁 → 廣角鏡頭拍攝跨頁
  → 左右頁分割 → 頁面曲率校正 → 傾斜校正 → 色彩校正
  → OCR 文字辨識 → 輸出 EPUB
```

整個流程全自動，使用者只需要把書放上去、按下開始。300 頁的書大約需要 **3.5 小時**完成。

### 核心設計：氣吸翻頁

這是整個專案最關鍵也最困難的部分。作者自己說：

> 「Making one page turn is relatively easy. Making hundreds of page turns reliably without skips or double turns is the real challenge.」

翻一頁不難，難的是翻幾百頁都不出錯——不跳頁、不黏頁。氣吸（vacuum suction）機構用負壓吸起紙張邊緣，配合伺服臂完成翻頁動作，這種方式比摩擦滾輪更不容易損傷紙張。

## 技術亮點

### 1. 非破壞性掃描

市面上很多高速掃描方案（包括 AI 公司大量收購稀有書籍的做法）都需要切開書脊，把書頁拆散才能送進自動進紙器。Kiro 的設計完全保留書籍原狀，掃完還是一本完整的書。

### 2. 影像後處理管線

拍攝跨頁後的處理步驟相當完整：
- **左右頁分割**：一次拍攝兩頁，軟體自動切割
- **頁面曲率校正**：書頁在攤開時不可能完全平整，軟體補償彎曲變形
- **傾斜校正**：修正拍攝角度偏差
- **色彩校正**：統一不同光線條件下的色調

### 3. 翻頁失敗偵測

當書頁沒有頁碼時，系統使用**影像相似度比對**來偵測翻頁是否失敗（連續兩張照片太像就代表沒翻成功），這比單純靠頁碼更通用。

### 4. 直接輸出 EPUB

不只是掃描成圖片 PDF，而是經過 OCR 後產出**文字版 EPUB**，可以在任何電子書閱讀器上重新排版、搜尋、標註。

## 目前限制

作者很誠實地列出了現階段的問題：

- ⏱️ 300 頁需要 3.5 小時（商用機器通常幾分鐘就搞定）
- 📖 銅版紙、受損書籍、異常裝訂、大開本仍有困難
- 👀 有些書仍需要偶爾人工檢查或重試
- 🌏 目前只在韓國提供試用租借，尚未開放國際

## 心得與延伸

### 為什麼這個專案值得關注？

書籍數位化是一個被嚴重低估的需求。圖書館、檔案館、獨立出版社、甚至只是想把家裡幾十年的藏書電子化的普通人，都面臨同樣的痛點：商用方案太貴，手動方案太累。

Kiro 用一片 RPi 3B+（大約台幣 1,200）加上一些機構件就做到了全自動掃描，雖然速度還比不上商用設備，但成本差了兩個數量級。

### 可能的改進方向

- **升級到 RPi 5 + Camera Module 3**：更快的處理速度和更好的影像品質
- **雙鏡頭同時拍攝左右頁**：省去分割步驟，提高解析度
- **加入 AI 增強 OCR**：用 LLM 做 OCR 後校正，提升辨識率
- **開源設計**：社群中已有人詢問是否會開源，如果能開放設計圖和軟體，會大幅降低複製門檻

### 類似專案

- **[Linear Book Scanner](https://linearbookscanner.org/)**：另一個開源自動掃書機，書本在機器上來回移動，用真空吸頁
- **[DIY Book Scanner 社群](https://diybookscanner.org/)**：自製掃書機的大本營，有各種 V 型書架設計
- **Treventus ScanRobot**：商用標竿，一台要價數百萬台幣，但翻頁速度和精度是業界頂尖

## 參考資料

- 原文：[I built a Raspberry Pi book scanner that turns pages without cutting the binding](https://www.reddit.com/r/raspberry_pi/comments/1voyops/i_built_a_raspberry_pi_book_scanner_that_turns/)
- Linear Book Scanner：https://linearbookscanner.org/
- Raspberry Pi Camera Module 3：https://www.raspberrypi.com/products/camera-module-3/
