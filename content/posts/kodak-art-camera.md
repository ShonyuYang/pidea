---
title: "Kodak 即可拍復活術 — 當 70 年代拍立得遇上 Raspberry Pi"
date: 2026-06-09
draft: false
tags: ["Raspberry Pi", "相機", "復古改裝", "藝術", "Maker"]
categories: ["專案推薦"]
description: "一台因專利戰爭而停產的 Kodak 拍立得相機，被 Maker 魔術師用 Raspberry Pi 賦予全新的數位藝術生命。"
---

## 前言

1976 年，Kodak 推出 EK 系列拍立得相機，正面挑戰 Polaroid 的即時攝影霸權。然而一場長達 15 年的專利訴訟，以 9.25 億美元的天價和解金收場——Kodak 被迫停產所有拍立得產品，1,650 萬台相機一夕間成為精美的廢鐵。

將近半世紀後，紐約的 Maker 魔術師 **Mario Marchese**（藝名 Mario the Maker Magician）從 98 歲的祖母手中收到了一台這樣的 Kodak Instant Camera。底片早已絕版，但他看到的不是廢物，而是機會——用 Raspberry Pi 讓這台相機重新「看見」世界，而且看見的是充滿機器人超現實主義的藝術世界。

## 專案概述

### 背景故事

Mario the Maker Magician 是一位在紐約活躍的全年齡層表演者，曾登上 **The Tonight Show with Jimmy Fallon**。他的表演融合魔術與自製機器人，用 Arduino 和各種 DIY 零件打造會變魔術的機械裝置，鼓勵孩子們動手創造。

這次的相機專案源自他的妻子 Katie Rosa Marchese（Reddit 帳號 @thegigiandthebear）分享的貼文。她的 98 歲祖母送給他們幾台老相機，其中一台是 70 年代的 Kodak Instant Camera。由於 Kodak 在專利訴訟後被迫停止生產底片，這台相機已經無法正常使用。

### 改裝內容

Mario 將 Raspberry Pi 塞入 Kodak 相機的經典外殼中，保留了原本的復古美學，同時賦予全新的數位攝影能力。根據影片展示，這不是一般的數位相機改裝——產出的影像帶有強烈的「機器人超現實主義」（robotic surrealism）風格，眼球是反覆出現的主題元素。

### 核心元件（推測）

| 元件 | 說明 |
|------|------|
| Raspberry Pi（Zero 2W 或更新型號） | 主控板，負責影像擷取與處理 |
| Pi Camera Module | 取代原本的底片機構 |
| Kodak Instant Camera 外殼 | 70 年代的經典造型，完美的復古容器 |
| 電池模組 | LiPo 電池供電（便攜使用） |
| 快門按鈕 | 可能沿用原機的快門機構 |

> ⚠️ 原帖為影片展示，未提供完整 BOM 或原始碼。以上元件清單為根據影片與類似專案推測。

## 技術亮點

### 1. 專利戰爭的遺產變成 Maker 素材

Kodak Instant Camera 的歷史本身就是一個精彩的故事。1976 年 Kodak 推出 EK 系列挑戰 Polaroid，Polaroid 立即提告。歷經 9 年審判，1985 年法院判決 Kodak 侵權，1986 年頒布禁令，Kodak 被迫立即停止生產。1991 年最終以 9.25 億美元和解——這在當時是美國史上最大的專利侵權賠償金。

這意味著：這些相機不只是「舊」，它們是被法律判決「處刑」的產品。底片永遠不會再生產。用 Raspberry Pi 復活它們，某種程度上是在修復一個歷史上的遺憾。

### 2. 機器人超現實主義的藝術風格

從影片中可以看出，這台相機產出的不是普通照片。Mario 的作品充滿了眼球、機械零件、和超現實的視覺元素。這可能是透過：

- **即時影像濾鏡**：使用 Python + OpenCV 或 Picamera2 的後處理功能
- **AI 風格轉換**：將拍攝的照片即時轉換為特定藝術風格
- **合成疊加**：將預設的機器人/眼球元素疊加到拍攝畫面上

這種「相機即藝術工具」的概念，讓攝影從記錄變成創作。

### 3. 復古外殼的工程挑戰

將現代電子元件塞入 70 年代的相機外殼，需要解決幾個問題：

- **空間規劃**：原本放底片的空間要容納 Pi、電池、相機模組
- **鏡頭對準**：Pi Camera 的感光元件要對準原本的鏡頭光路
- **散熱**：密閉的塑膠外殼中，Pi 的散熱需要考慮
- **供電**：需要足夠小的電池方案，同時提供合理的續航

### 類似專案參考

這個專案屬於「復古相機數位化」的 Maker 傳統，其他知名作品包括：

- **Rodak**（Alex Ellis）：將 1950 年代的 Kodak Brownie 改裝為 Pi Zero 數位相機，腰平取景
- **Optocam Zero**（Doruk Kumkumoğlu）：受 Kodak Charmera 鑰匙圈相機啟發的口袋型 Pi 相機
- **Canon Super 8 改裝**：用 Pi Zero 2W 將經典超八攝影機變成 4K 數位攝影機
- **百年相機數位化**（Hackaday.io）：將 100 年歷史的相機改裝為數位相機

## 心得與延伸

### 為什麼這個專案特別？

技術上，用 Pi 改裝舊相機並不新鮮。但這個專案的獨特之處在於三個層面：

1. **歷史深度**：不是隨便一台舊相機，而是一台因為美國史上最大專利訴訟而「被處刑」的產品。復活它本身就帶有某種歷史修復的意味。

2. **家族故事**：來自 98 歲祖母的禮物，三代人之間的連結。老物件不是被丟棄，而是被重新賦予意義。

3. **藝術取向**：不是追求畫質或功能，而是把相機變成藝術創作工具。「機器人超現實主義」的風格讓每張照片都是一件作品。

### 可以怎麼延伸？

- **加入熱感印表機**：Mario 之前就做過 Pi 驅動的熱感印表機即時相機，結合兩個專案可以做出真正的「即拍即印」復古體驗
- **AI 藝術濾鏡**：用輕量級的 Style Transfer 模型（如 TensorFlow Lite），讓每次拍攝自動套用不同的藝術風格
- **社群分享**：有 Reddit 留言提到用 AI 描述場景再生成圖片的概念——拍一張照片，讓 AI 重新詮釋它
- **展覽互動裝置**：這台相機非常適合放在 Maker Faire 或藝術展覽中，讓觀眾體驗「復古未來主義」的攝影

### 預算估算

| 項目 | 估計費用 |
|------|----------|
| Kodak Instant Camera（eBay 二手） | $15–40 USD |
| Raspberry Pi Zero 2W | $15 USD |
| Pi Camera Module V2/V3 | $25–35 USD |
| LiPo 電池 + 充電模組 | $10–15 USD |
| 其他零件（按鈕、線材） | $5–10 USD |
| **合計** | **$70–115 USD** |

## 參考資料

- [原始 Reddit 貼文](https://www.reddit.com/r/raspberry_pi/comments/1tzrn5n/raspberry_pi_powered_art_camera_built_into_a/)
- [Mario the Maker Magician 官網](https://www.mariothemagician.com/)
- [Mario the Maker Magician Instagram](https://instagram.com/mariothemagician)
- [Rodak — Kodak Brownie Pi 相機（GitHub）](https://github.com/alexellis/rodak)
- [Optocam Zero — Hackster.io](https://www.hackster.io/news/your-new-favorite-camera-is-a-raspberry-pi-15aedf614141)
- [Kodak vs. Polaroid 專利戰爭（Fstoppers）](https://fstoppers.com/historical/patent-war-changed-photography-forever-714881)
