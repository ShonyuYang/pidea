---
title: "瑪利歐穿越回 16 位元 — Super FX 晶片硬跑超級瑪利歐 64"
date: 2026-09-29
draft: false
tags: ["SNES", "Super FX", "Super Mario 64", "復古遊戲", "demake", "homebrew"]
categories: ["專案推薦"]
description: "Modder Game of Tobi 用 SNES 卡帶裡的 Super FX 協處理器，把 N64 的超級瑪利歐 64 硬搬回 16 位元時代。"
---

## 前言

「Can it run Doom?」已經是老梗了。2026 年的新問題是：**Can it run Mario 64?**

自從超級瑪利歐 64 的原始碼被反編譯公開後，這款 N64 經典被移植到各種不可思議的平台——瀏覽器、3DS、甚至 Game Boy Color。但這次的挑戰不一樣：不是把遊戲搬到更強的硬體，而是**往回推一個世代**，讓 1990 年的 Super Nintendo 跑 1996 年的 3D 遊戲。

Modder [Game of Tobi](https://www.youtube.com/@GameofTobi) 做到了。用 SNES 卡帶裡的 Super FX 協處理器，渲染出超級瑪利歐 64 的 3D 多邊形世界。

## 專案概述

### Super FX 是什麼？

Super FX 是任天堂與英國遊戲公司 Argonaut Games 合作開發的協處理器晶片，直接焊在 SNES 卡帶電路板上。它本質上是一顆 **16 位元 RISC 處理器**，專門用來加速 SNES 原生硬體做不到的 3D 多邊形運算。

| 規格 | 數值 |
|------|------|
| 架構 | 16-bit RISC（GSU — Graphical Support Unit） |
| 時脈 | 10.7 MHz（初代）/ 21.47 MHz（GSU-2） |
| 用途 | 3D 多邊形渲染、進階 2D 特效 |
| 代表作 | Star Fox、Yoshi's Island、DOOM |
| 開發代號 | MARIO（Mathematical, Argonaut, Rotation & I/O） |

有趣的冷知識：晶片開發代號叫「Super Mario FX」，導致坊間長期流傳任天堂曾計畫在 SNES 上做 3D 瑪利歐遊戲的都市傳說。三十年後，有人真的把它做出來了。

### SMFX：把 N64 瑪利歐搬回 SNES

Game of Tobi 的做法不是直接移植 SM64 的程式碼（SNES 的 65C816 CPU 和 N64 的 MIPS R4300i 完全不同架構），而是：

1. **全新引擎**：為 Super FX 從頭撰寫 3D 渲染引擎
2. **原版素材**：使用 SM64 的原始 3D 模型、貼圖和關卡資料
3. **畫家演算法**：由後往前逐層繪製（back-to-front），因為 Super FX 沒有 Z-buffer 硬體
4. **關卡裁切**：2 MB 記憶體上限迫使部分關卡需要刪減

結果？瑪利歐真的在 SNES 上跑起來了——雖然畫面粗糙、幀率不高，但多邊形世界、縱深感和貼圖全都在。

### 技術限制

這不是完美移植，而是一場硬體極限的探索：

- **幀率**：遠低於 N64 版的 30 fps，評論者估計 <10 fps
- **記憶體**：Super FX 卡帶最大 2 MB ROM，SM64 原版是 8 MB
- **無 Z-culling**：每一幀都要從最遠的物件畫到最近，效能代價很高
- **貼圖品質**：受限於記憶體和頻寬，材質明顯「方塊化」

## 技術亮點

### 卡帶就是升級套件

SNES 的天才設計在於：卡帶插槽不只是資料介面，而是完整的匯流排擴展。遊戲廠商可以在卡帶裡塞任何客製晶片——Super FX、SA-1、DSP-1——每一款遊戲都能帶著自己的「GPU」。

這個概念在光碟時代消失了。CD-ROM 只是儲存媒介，沒辦法在光碟裡塞一顆晶片。SNES 卡帶的可擴展性，某種程度上比後來的主機更前衛。

### Super FX 3 的前世今生

2025 年，Limited Run Games 重新發行 SNES 版 DOOM 實體卡帶時，用了一顆 **Raspberry Pi RP2350**（跑在 150 MHz）來模擬 Super FX 晶片，稱為「Super FX 3」。這意味著現代的 RP2040/RP2350 微控制器已經能完全取代 90 年代的客製 ASIC，而且性能更好。

Game of Tobi 的 SMFX 專案如果未來發布 ROM，搭配 Super FX 3 卡帶硬體或模擬器，玩家就能在真正的 SNES 上體驗。

### 從 Star Fox 到 Mario 64

Super FX 最知名的作品是 1993 年的 Star Fox——它證明了 SNES 可以做 3D 遊戲，但那是軌道射擊，場景相對簡單。SMFX 把挑戰拉到另一個層次：開放式 3D 平台跳躍，需要自由視角、複雜地形和大量物件。這大概是 Super FX 被推到的最極端用途。

## 心得與延伸

### 如果任天堂當年真的做了呢？

N64 在 1996 年帶來了真正的 3D 革命，但 SNES 的 Super FX 其實在 1993 年就展示了 3D 的可能性。如果任天堂選擇繼續投資 SNES + Super FX 路線，而不是開發全新的 N64 主機，遊戲產業會走向完全不同的方向。

不過正如評論者指出，<10 fps 的 3D 平台跳躍遊戲在商業上是不可行的——任天堂對品質的堅持不會允許這樣的產品上市。SMFX 更像是一個「技術上可以，商業上不行」的精彩示範。

### Demake 文化的魅力

SMFX 不是唯一的「逆向移植」專案。Game of Tobi 之前還把 Minecraft 塞進了 SNES，包含一個通往地獄的傳送門。這類 demake 專案的價值不在實用性，而在於它們揭示了硬體的真正潛力——那些原廠工程師可能從未想像過的用途。

### 等待發布

Game of Tobi 表示會在專案穩定後公開發布。屆時你可以用 SNES 模擬器（如 bsnes、Snes9x）或搭配 FXPak Pro 等燒錄卡在真機上體驗。

## 參考資料

- 🔗 原文：[Hackaday — Using The SNES Super FX Chip To Run Super Mario 64](https://hackaday.com/2026/09/28/using-the-snes-super-fx-chip-to-run-super-mario-64/)
- 🎮 作者頻道：[Game of Tobi — YouTube](https://www.youtube.com/@GameofTobi)
- 📰 報導：[XDA — Someone got Super Mario 64 running on the SNES](https://www.xda-developers.com/someone-got-super-mario-64-running-on-the-snes-using-the-super-fx-chip/)
- 📖 技術背景：[Wikipedia — Super FX](https://en.wikipedia.org/wiki/Super_FX)
