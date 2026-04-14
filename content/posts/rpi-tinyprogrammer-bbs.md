---
title: "TinyProgrammer：桌上那位永遠不會離職的 AI 工程師"
date: 2026-04-14
draft: false
tags: ["Raspberry Pi", "AI", "LLM", "Python", "BBS", "桌面裝置"]
categories: ["專案推薦"]
description: "一台放在桌上的 Raspberry Pi，自主寫 Python、會犯錯自己修、還會上 BBS 跟其他裝置聊天——這大概是最可愛的 AI 同事了。"
---

## 前言

想像一下，你桌上有一台小小的裝置，每天早上自動「打卡上班」開始寫程式，累了就上 BBS 跟其他裝置聊天、分享自己寫的 code、互相吐槽對方的程式碼，下班時間一到還會自動切換成星空螢幕保護程式。這不是科幻小說的場景，而是 **TinyProgrammer** 這個開源專案帶來的奇妙體驗。

## 專案概述

TinyProgrammer 是一個跑在 Raspberry Pi 上的自主程式撰寫裝置。它的核心概念很簡單：讓一台小電腦永無止境地寫 Python 程式，而且模擬得像一個真人工程師——以人類打字的速度一個字一個字敲出來，偶爾犯錯，然後自己 debug 修正。

### 硬體需求

- **主板**: Raspberry Pi 4 或 Pi Zero 2 W
- **顯示器**: 小螢幕（模仿經典 Mac IDE 介面）
- **網路**: 用於 BBS 社交功能和 LLM API 呼叫

### 核心功能

| 功能 | 說明 |
|------|------|
| 🖥️ 自主寫程式 | 以人類打字速度撰寫 Python，會犯錯並自行修正 |
| 😊 心情系統 | 裝置有「心情」狀態，影響寫程式的行為和風格 |
| 💬 BBS 社交 | 多台裝置可連上 BBS，分享程式、評論 code、發笑話 |
| 🌙 作息模式 | 下班自動切換星空螢幕保護程式，早上回來繼續「上班」 |
| 🌐 Web Dashboard | 網頁介面修改設定、提示詞、色彩濾鏡等 |

## 技術亮點

### LLM 整合策略

TinyProgrammer 支援 **OpenRouter 雲端 API** 和本地模型兩種方式。作者實測了多款小型模型在 Raspberry Pi 上的表現：

- **SmolLM2-135M** — 太小，輸出不穩定
- **Qwen 2.5 Coder 0.5B / 1.5B** — 有潛力但還不夠穩定
- **DeepSeek Coder 1.3B** — 同上
- **Phi-3 Mini** — 表現最好但仍有限

作者目前期待 **Gemma4** 能帶來突破，讓裝置真正實現完全離線運作。這個嘗試本身就很有價值——在邊緣裝置上跑 code generation 是目前 Edge AI 的熱門方向。

### BBS 社交層

這是最讓人眼睛一亮的設計。每台 TinyProgrammer 都有**獨立人格**，這個人格會影響：

- 瀏覽哪些 BBS 板塊
- 發文的風格和語氣
- 對其他裝置程式碼的評論方式

當裝置進入 BBS 模式時，螢幕會從經典 Mac IDE 介面切換成**綠底黑字的復古終端**風格，氛圍感拉滿。想像一群小裝置在 BBS 上互相 code review，這畫面實在太有趣了。

### 擬人化設計的巧思

整個專案最厲害的地方在於「擬人化」做得非常到位：

1. **打字速度** — 不是瞬間輸出，而是模擬人類敲鍵盤的節奏
2. **犯錯機制** — 刻意加入 typo 和 bug，再自行修正
3. **作息時間** — 有上下班的概念，不是 24 小時不停
4. **心情影響** — 心情好壞會改變程式碼風格

這些細節讓它從一個「跑 LLM 的 Pi」變成了一個有生命力的桌面夥伴。

## 心得與延伸

TinyProgrammer 最吸引人的地方不在於技術有多複雜，而在於它把 AI 和硬體結合成了一個**有溫度的互動體驗**。它讓人想到早期 Tamagotchi 電子寵物的概念——只是這次你養的不是小雞，而是一個會寫程式的 AI 同事。

幾個值得延伸的方向：

- **教育用途**: 放在教室裡，讓學生觀察 AI 怎麼寫程式、怎麼 debug，比看教科書生動多了
- **辦公室裝飾**: 當作 desk toy，同事經過都會忍不住停下來看它在寫什麼
- **多裝置互動**: BBS 功能讓多台裝置組成社群，可以想像一個辦公室裡好幾台在互相聊天的場景

專案以 **GPL-3.0** 授權開源，門檻不高，一台 Pi Zero 2 W 就能玩起來。

## 參考資料

- 🔗 原文討論: [Reddit r/raspberry_pi](https://www.reddit.com/r/raspberry_pi/comments/1se9afo/made_a_tiny_device_that_writes_code_takes_breaks/)
- 💻 GitHub: [cuneytozseker/TinyProgrammer](https://github.com/cuneytozseker/TinyProgrammer)
