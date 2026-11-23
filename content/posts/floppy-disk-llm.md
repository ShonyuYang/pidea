---
title: "軟碟裝得下 AI — 367K 參數 Transformer 塞進 1.44MB 的極限挑戰"
date: 2026-11-23
draft: false
tags: ["LLM", "DOS", "復古電腦", "模型壓縮", "Turbo C", "486"]
categories: ["專案推薦"]
description: "一個用 Turbo C 2.0 寫成、跑在 486DX 上的迷你 Transformer，整個程式加模型只佔 435KB，塞進一張軟碟還有剩。"
---

## 前言

「大型語言模型」這個詞已經深入人心，但當有人把一個 Transformer 壓到能塞進一張 1.44MB 軟碟片時，「大型」二字就顯得格外諷刺了。

開發者 [WollyMonjac](https://github.com/WollyMonjac/Tiny-2.1) 發佈的 **Tiny 2.1** 是一個能在 MS-DOS 環境下運行的問答式 Transformer 模型。它用 Turbo C 2.0 撰寫，目標平台是 486DX 或 Pentium 等級的古董 PC，整個程式加模型權重只佔約 435KB — 放進一張格式化過的 1.44MB 軟碟片後，空間還綽綽有餘。

更有趣的是：這個 Transformer 本身是由 Anthropic 的 Claude Fable 5.2 撰寫，訓練資料則由 Opus 5.5 生成。換句話說，這是一個 AI 造出來的 AI，而且它跑在比你阿公的電腦還老的硬體上。

## 專案概述

### 架構規格

| 項目 | 規格 |
|------|------|
| 參數量 | 367,000（367K） |
| 網路架構 | 4 層 Transformer，隱藏維度 96 |
| 權重格式 | 8-bit 整數（每行一個縮放因子） |
| 模型大小 | 380KB |
| 程式 + 模型 | ~435KB |
| 記憶體需求 | 540KB 傳統記憶體 |
| 目標平台 | 486DX / Pentium，MS-DOS |
| 開發語言 | Turbo C 2.0 |
| 訓練資料 | ~3,500 個事實，多種問法 |
| 授權 | MIT |

### 運作方式

Tiny 2.1 採用逐字母讀寫的方式處理輸入與輸出 — 沒有 subword tokenizer，沒有 BPE，就是最原始的 character-level 處理。模型被訓練了約 3,500 個事實，每個事實都用多種不同的問法來訓練，讓它能理解同一個問題的不同表達方式。

在已訓練的知識範圍內，它的回答相當準確。但超出範圍？它會像所有 LLM 一樣，以無比的自信胡說八道。例如問它 17 × 23 等於多少，它會毫不猶豫地回答 96。

### 介面

DOS 環境下的介面出乎意料地精緻：全螢幕 neon 風格 UI，包含對話歷史視窗、即時神經網路活動監視器、輸入框，底部還有 F 鍵快捷鍵。最復古的彩蛋是 — 它會透過 PC Speaker 發出機器人語音來「唸」出回答。

## 技術亮點

### 極限壓縮的藝術

現代 LLM 動輒數十億甚至數千億參數，光模型檔就要幾十 GB。Tiny 2.1 用 367K 參數做到了一個堪用的 Q&A 系統，這中間的差距是六個數量級。

關鍵在於幾個設計決策：

1. **8-bit 量化**：每個權重只佔一個 byte，搭配每行一個浮點縮放因子。這比現代常見的 4-bit 量化還粗暴，但在這個尺度下完全夠用。
2. **Character-level 處理**：拋棄了所有現代 tokenizer 的複雜性，直接一個字母一個字母處理。這大幅減少了詞彙表的大小。
3. **固定知識範圍**：不追求通用性，只專注在 3,500 個事實上。這讓小模型能在有限的容量內達到不錯的準確度。
4. **Turbo C 2.0 原生編譯**：沒有 Python runtime、沒有 CUDA、沒有任何現代依賴。一個 `.EXE` 加一個 `.BIN` 就是全部。

### AI 寫 AI

這個專案最引人注目的面向之一：程式碼由 Claude Fable 5.2 撰寫，訓練資料由 Opus 5.5 生成。開發者 WollyMonjac 扮演的更像是「導演」角色 — 定義需求、驗證結果、調整方向，但實際的程式碼和資料都是 AI 產出的。

這引發了社群不少討論：如果創作者本人無法完全解釋模型的每一行程式碼，這算不算真正的「創作」？但從另一個角度看，能用 AI 工具做出一個跑在 30 年前硬體上的 AI，本身就是一種新型態的工程能力。

### 自己訓練自己的模型

專案在 GitHub 上以 MIT 授權釋出了完整的訓練工具鏈：

```bash
cd training
python3 gen2.py          # 預覽生成的問答對
python3 train2.py train 60 more.pt --resume tiny2.pt --lr 5e-4 --decay 0.3 --eval-every 20
python3 train2.py eval more.pt
python3 train2.py export more.pt TINY.BIN
```

你可以在 `training/facts/` 資料夾裡加入自己的事實，重新訓練後匯出新的 `TINY.BIN`，就能打造一個專屬於你的知識領域的迷你 AI。訓練只需要 Python 3、NumPy 和 PyTorch，一般 CPU 就跑得動。

## 心得與延伸

### 為什麼這很重要

Tiny 2.1 當然不是要取代 ChatGPT。它的價值在於提出一個根本性的問題：**AI 到底需要多大？**

當我們把所有注意力都放在「更大的模型、更多的 GPU、更多的資料」時，這個專案提醒我們：對於特定的、有限的任務，一個 380KB 的模型就夠了。這個思路在邊緣裝置、嵌入式系統、離線環境中有巨大的實用價值。

### 延伸可能

- **博物館互動展品**：在一台真正的 486 電腦上跑 AI 問答，讓參觀者體驗「復古未來」
- **教學工具**：完整的訓練流程只需要 CPU，非常適合用來教 Transformer 的基本原理
- **嵌入式 FAQ 系統**：在資源極度受限的環境中提供基本的知識問答
- **客製化知識庫**：把你的產品手冊、FAQ 訓練進去，做成一個離線可用的迷你助手
- **行為藝術裝置**：一台老電腦、一張軟碟片、一個會聊天的 AI — 這本身就是一件關於科技演進的藝術品

### 限制

老實說，3,500 個事實的容量在實際應用中非常有限。而且 character-level 的處理方式意味著它對中文等非拉丁語系的支援基本為零。但作為一個概念驗證和教學工具，它的價值遠超過它的能力。

## 參考資料

- 🔗 原文：[Not-So-Large Language Model Fits On A Floppy Disk — Hackaday](https://hackaday.com/2026/11/18/not-so-large-language-model-fits-on-a-floppy-disk/)
- 📦 GitHub：[WollyMonjac/Tiny-2.1](https://github.com/WollyMonjac/Tiny-2.1)
- 📰 報導：[Tiny AI runs on 486 DOS PCs from a floppy disk — Notebookcheck](https://www.notebookcheck.net/Tiny-AI-runs-on-486-DOS-PCs-from-a-floppy-disk.1456755.0.html)
