---
title: "當形式驗證遇上微控制器：Ada SPARK + Zephyr 讓你在 Pico 2W 上寫出零缺陷韌體"
date: 2026-05-26
draft: false
tags: ["Ada", "SPARK", "Zephyr", "Pico 2W", "RTOS", "形式驗證", "嵌入式"]
categories: ["專案推薦"]
description: "一個週末的 vibe-coding 產物：在 Zephyr RTOS 上跑 Ada SPARK，讓 ARM Cortex-M 微控制器也能享受編譯期形式驗證。"
---

## 前言

在嵌入式開發的世界裡，大多數人選擇 C 或 Rust。但有一群人堅持用一種誕生於 1983 年的語言 —— Ada，以及它的形式驗證子集 SPARK —— 來寫微控制器韌體。這不是復古情懷，而是因為 SPARK 能在**編譯期就證明你的程式不會有執行期錯誤**。

最近有位開發者在一個週末內，成功將 GNAT 的 Ravenscar 輕量任務排程移植到 Zephyr RTOS 上，讓 Ada task 直接映射為 Zephyr thread。他用這套工具在 Raspberry Pi Pico 2W 上向 Home Assistant 回報水質感測器數據。整個專案開源在 GitLab 上，而且已經支援四款開發板。

## 專案概述

**light-tasking-zephyr** 是一個 GNAT Ravenscar 輕量任務執行環境（Light Tasking Runtime），將 Ada 的任務模型「接地」到 Zephyr 的核心 API 上：

| Ada 概念 | Zephyr 對應 |
|---------|------------|
| Ada Task | `k_thread` |
| `Ada.Real_Time` | `k_uptime_get` / `k_usleep`（1µs 解析度）|
| Protected Object | `k_mutex` + 優先權天花板協議 |

### 已驗證的開發板

| 開發板 | SoC | 核心 |
|-------|-----|------|
| `rpi_pico` | RP2040 | Cortex-M0+ |
| `rpi_pico2_w` | RP2350 | Cortex-M33 +FPU |
| `frdm_mcxa156` | MCXA156 | Cortex-M33 +FPU |
| `nucleo_h563zi` | STM32H5 | Cortex-M33 +FPU |

### 生態系統

這不只是一個 runtime，而是一整套 Ada 綁定生態系統。作者在 `close-hauled/crates/` 下維護了多個 sibling crate：

- **zephyr_sockets** — GNAT.Sockets 風格的網路介面
- **zephyr_mqtt** — MQTT 客戶端
- **zephyr_wifi** — WiFi STA / Soft-AP + DHCPv4
- **zephyr_gpio / zephyr_uart / zephyr_i2c / zephyr_spi / zephyr_adc** — 各種驅動綁定
- **zephyr_nvs** — Flash 鍵值儲存

每個 crate 都有獨立的 CI pipeline，在 Renode 模擬器上對 RP2040 進行煙霧測試。

## 技術亮點

### 1. 形式驗證不是空談

Ada SPARK 的殺手級功能是 **靜態證明**（Static Proof）。你可以在程式碼中加入前置條件、後置條件和不變量，SPARK 的證明器會在編譯期驗證這些條件永遠成立。這意味著：

- **零除法？不可能。** 證明器會在編譯期抓到
- **陣列越界？不存在。** 靜態分析確保索引永遠合法
- **整數溢位？編譯失敗。** 除非你能證明它不會發生

對於監控 pH 和 TDS 這類感測器數據的應用，這種保證特別有價值 —— 你不想因為一個 off-by-one 錯誤而誤報水質異常。

### 2. ISA 無關的架構設計

runtime 的核心是 ISA-agnostic 的。Zephyr 負責所有硬體抽象，Ada 側只看到 `k_*` API。要支援新的 Cortex-M 核心，只需在 `target_options.gpr` 中加一個 `PLATFORM` 值，選擇對應的 `light-cortex-*` GNAT runtime。

### 3. Alire + CMake 的混合建構

建構流程巧妙地結合了兩個世界：
- **Alire**（Ada 的套件管理器）負責拉取 `gnat_arm_elf` 交叉編譯工具鏈
- **Zephyr/CMake** 負責頂層建構
- `cmake/AdaRuntime.cmake` 在 Zephyr 建構過程中驅動 `gprbuild`

這種混合建構讓你可以用 `alr exec -- west build -b rpi_pico2_w .` 一行指令完成從工具鏈拉取到韌體編譯的全部流程。

### 4. Renode CI 全自動測試

每個 crate 的 pipeline 都會在 Renode 模擬器上建構並執行 Ada 應用程式，針對 RP2040 的平台描述進行煙霧測試。pipeline 失敗就擋 merge —— 綠色 pipeline 代表「在模擬設備上建構並通過測試」。

runtime 本身的 pipeline 更是跑四板矩陣測試（rpi_pico、rpi_pico2_w、frdm_mcxa156、nucleo_h563zi），每塊板子都有對應的 Renode 平台描述檔。

## 心得與延伸

### 為什麼這很重要？

嵌入式開發正在經歷一場「安全語言革命」。Rust 在這方面獲得了大量關注，但 Ada/SPARK 其實是這個領域的老前輩 —— 航空電子、鐵路信號、醫療設備早就在用了。這個專案的意義在於**把企業級的形式驗證工具帶到 maker 級的硬體上**。

一塊 Pico 2W 不到 10 美元，但你現在可以在上面跑跟波音 787 飛控系統同等級的程式語言。這個落差本身就很有趣。

### 可能的延伸

- **Cortex-R / Cortex-A 移植**：目前僅支援 M-profile，但架構上是可擴展的
- **RISC-V 支援**：Zephyr 已經支援 RISC-V，Ada 工具鏈也有 RISC-V 後端
- **更多感測器綁定**：ADC crate 已經就緒，可以輕鬆加入更多感測器驅動
- **安全關鍵 DIY 應用**：結合 SPARK 的證明能力，適合 DIY 醫療監控或環境監測

### 適合誰？

如果你是嵌入式開發者，對程式碼品質有極高要求，或者單純好奇「除了 C 和 Rust 之外還有什麼選擇」，這個專案值得一看。入門門檻不低（需要學 Ada），但回報是**數學級別的正確性保證**。

## 參考資料

- 原始碼：https://gitlab.com/close-hauled/light-tasking-zephyr
- Ada 入門：https://ada-lang.io
- Zephyr RTOS：https://zephyrproject.org
- Alire（Ada 套件管理器）：https://alire.ada.dev
- SPARK 形式驗證：https://www.adacore.com/about-spark
