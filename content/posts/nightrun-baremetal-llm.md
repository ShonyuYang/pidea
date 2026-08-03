---
title: "沒有 Linux 也能跑 AI — NightRun 讓 RPi 5 裸機直接對話大型語言模型"
date: 2026-08-03
draft: false
tags: ["Raspberry Pi 5", "LLM", "Rust", "UEFI", "bare-metal", "Llama", "Qwen"]
categories: ["專案推薦"]
description: "NightRun 是一個 Rust UEFI 應用程式，讓 Raspberry Pi 5 完全不需要作業系統就能開機跑 LLM——從插卡到對話只要幾秒鐘。"
---

## 前言

跑大型語言模型（LLM）的標準流程是什麼？裝 Linux、設定 Python 環境、下載模型、啟動推理框架⋯⋯光是環境就能折騰一個下午。但如果我告訴你，有個專案能讓 Raspberry Pi 5 **完全不裝任何作業系統**，插上 SD 卡開機就直接跟 AI 對話呢？

[NightRun](https://github.com/hardrave/NIGHTRUN) 就是這樣一個瘋狂的東西：一個用 Rust 寫的 UEFI 應用程式，讓你的機器**只做一件事**——跑 LLM。沒有 Linux、沒有 kernel、沒有瀏覽器、沒有網路堆疊。機器的韌體直接啟動 NightRun，模型載入 RAM，你就開始聊天。

## 專案概述

### 它是什麼？

NightRun 是一個 `no_std` Rust UEFI 應用程式。它不是跑在 Linux 上的程式，而是**取代了整個作業系統**。開機流程是這樣的：

1. 把 NightRun 燒到 SD 卡（RPi 5）或 USB 隨身碟（x86_64 PC）
2. 插上去開機
3. UEFI 韌體直接啟動 NightRun
4. 量化模型（1.3–2.4 GB）串流載入 RAM，邊讀邊驗 CRC-32
5. 儲存裝置封鎖——之後任何磁碟讀取都會觸發硬錯誤（故意的）
6. 出現聊天介面，純 CPU 推理，離線，永遠

### 支援的模型

| 模型 | 量化格式 | 大小 | 最低 RAM | 平台 |
|------|---------|------|---------|------|
| Llama 3.2 1B Instruct | Q8_0 | 1.3 GB | 4 GB | x86_64, Pi 5 |
| Llama 3.2 3B Instruct | Q4_K_M | 1.9 GB | 6 GB | x86_64, Pi 5 |
| Granite 4.1 3B | Q4_K_M | 2.0 GB | 6 GB | x86_64, Pi 5 |
| Qwen3 4B Instruct 2507 | Q4_K_M | 2.3 GB | 8 GB | x86_64, Pi 5 (8GB) |

### 材料清單

- Raspberry Pi 5（4GB/8GB/16GB 皆可，4GB 只能跑 1B 模型）
- microSD 卡（至少 8GB，裝模型用）
- 鍵盤（USB）
- 螢幕（HDMI）
- 一台有 Linux + Rust 的電腦（用來建置映像檔）

成本：如果你已經有 RPi 5，額外成本為零。

## 技術亮點

### 1. 真正的 no_std Rust

NightRun 不是「在 Linux 上跑的 Rust 程式」，而是直接在 UEFI Boot Services 上執行的 `no_std` 應用。它自己畫 framebuffer 終端機（用點陣字型）、自己處理 USB 鍵盤輸入（含游標編輯和捲動），甚至在 Pi 5 上自己控制風扇轉速——因為沒有別的東西能做這件事。

### 2. 手寫量化推理核心

推理引擎不是套用現成框架，而是**手寫的量化運算核心**：

- x86_64：AVX2 + FMA + F16C 指令集
- RPi 5（AArch64）：NEON 指令集
- 支援 Q8_0、Q4_K、Q6_K 權重格式，**直接在量化資料上運算**，不做反量化複製
- Prompt 批次處理（最多 64 tokens/pass）
- 生成迴圈完全不做記憶體分配

### 3. 嚴格的正確性驗證

這不是「跑起來就好」的 side project。NightRun 對正確性有近乎偏執的堅持：

- Greedy 輸出與 llama.cpp **逐 token 比對**，每個支援的模型家族、每次程式碼變更都要通過
- Tokenizer 用 Hugging Face 官方 tokenizer 產生的 fixture 測試
- Chat template 用 `apply_chat_template` 交叉驗證
- 規則：「如果核心修改破壞了一致性，那是核心的問題。」這條規則已經抓到真實的 bug

### 4. 儲存封鎖機制

模型載入完成後，NightRun 會**封鎖儲存裝置**。之後任何磁碟讀取都會觸發硬錯誤。這不是 bug，是 feature——確保推理過程中不會有任何意外的 I/O 干擾，也是一種安全機制。

### 5. 多核推理

透過 UEFI 的 MP Services 協定啟用多核心推理。在 Pi 5 的四核 Cortex-A76 上，這意味著所有核心都在為你的 LLM 工作，不會有任何資源被作業系統吃掉。

## 心得與延伸

### 為什麼這很酷？

表面上看，NightRun 解決的問題很小眾——誰需要不裝 OS 就跑 LLM？但它真正展示的是幾件深刻的事：

1. **作業系統的開銷比你想的大**：即使是最輕量的 Linux 發行版，kernel + systemd + 各種 daemon 也會吃掉可觀的 RAM 和 CPU 時間。在 4GB 的 Pi 5 上，這些資源對 LLM 來說是真金白銀。

2. **Rust 在系統程式設計的實力**：用 Rust 寫一個完整的 UEFI 應用（含 framebuffer 渲染、USB HID、多核排程、量化推理），而且是 `no_std`——這本身就是一個 Rust 系統程式設計的教科書級範例。

3. **LLM 推理的本質很單純**：剝掉所有抽象層之後，LLM 推理就是矩陣乘法。NightRun 證明你不需要 Python、不需要 PyTorch、甚至不需要 OS，就能讓模型跑起來。

### 可能的延伸

- **離線 AI 終端機**：搭配電池和小螢幕，做一台完全離線的 AI 對話裝置
- **教育用途**：學習 UEFI 開發、bare-metal 程式設計、量化推理的絕佳教材
- **嵌入式 AI**：概念上可以移植到其他有 UEFI 支援的嵌入式平台
- **安全應用**：完全離線、儲存封鎖、沒有網路堆疊——這是最安全的 LLM 部署方式

### 限制

- 目前只支援 RPi 5 和 x86_64 UEFI 機器
- 沒有網路功能（這也是 feature）
- 模型選擇有限（4 款），但都是主流模型
- 需要另一台電腦來建置映像檔

## 參考資料

- 🔗 原文討論：[Reddit r/raspberry_pi](https://www.reddit.com/r/raspberry_pi/comments/1v93afn/i_built_nightrun_boot_a_local_llm_directly_on_a/)
- 📦 GitHub Repo：[hardrave/NIGHTRUN](https://github.com/hardrave/NIGHTRUN)
- 📰 Hackster 報導：[Run a Local LLM on Raspberry Pi's Bare Metal](https://www.hackster.io/news/run-a-local-llm-on-raspberry-pi-s-bare-metal-linux-not-necessary-6c7e3817293f)
