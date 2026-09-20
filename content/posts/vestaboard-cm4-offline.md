---
title: "你的翻頁時鐘不該聯網才能翻 — Vestaboard CM4 完全離線改造紀實"
date: 2026-09-20
draft: false
tags: ["Raspberry Pi", "CM4", "IoT", "Right to Repair", "逆向工程", "Split-Flap"]
categories: ["專案推薦"]
description: "一台要價 $999 的翻頁顯示器，韌體更新失敗後變磚。作者 root 了 CM4、逆向序列協議、移除雲端依賴，讓 8,448 片翻頁片重新自由翻動。"
---

## 前言

Vestaboard 是一塊讓人心動的產品——6×22 格、8,448 片機械翻頁片，掛在牆上像一座私人的火車站出發看板。你可以用手機推送天氣、股價、倒數日期，聽著翻頁片「嗒嗒嗒」地翻轉，那種實體回饋感是任何螢幕都給不了的。

但它有一個致命的設計：**所有訊息都必須經過 Vestaboard 的雲端伺服器**。

一位使用者在韌體更新失敗後，顯示器卡在「Status: Updating」的死亡畫面。Wi-Fi 持續斷線、雲端無法渲染內容、客服回覆緩慢。一台 $999 的硬體，因為伺服器端的問題變成了一面昂貴的牆壁裝飾。

他決定動手：root CM4、dump 韌體、逆向序列協議，最終讓這台顯示器完全離線運作。這不只是一次修復，更是一場 right-to-repair 的實踐。

## 專案概述

### 硬體架構

Vestaboard 的核心是一塊 **Raspberry Pi Compute Module**（早期版本用 CM3+ 8GB eMMC，新版已升級為 CM4）。CM 透過板載序列匯流排（`/dev/ttyAMA0`）與 22 組翻頁柱控制器通訊，每組控制 6 片翻頁片（實際協議預留了 7 片的空間，暗示可能有未發布的 7×22 版本）。

| 規格 | 數值 |
|------|------|
| 顯示格式 | 6 行 × 22 列 |
| 翻頁片總數 | 8,448 片 |
| 運算核心 | RPi CM3+/CM4 |
| 通訊協議 | UART 38400 baud, 8N1 |
| 可用顏色 | 8 色（黑/白/紅/橙/黃/綠/藍/紫） |
| 售價 | $999 起 |

### 原廠韌體問題

逆向工程揭露了原廠韌體的幾個驚人事實：

1. **5 個 Docker 容器同時運行**——在一塊 CM3+ 上跑 5 個容器，混合了 Python、Java 和 Bash，程式碼品質堪憂
2. **共用 SSH 金鑰漏洞**——rootfs 中包含一把 `vestaboard-root-key` SSH 私鑰，公鑰寫在 `/root/.ssh/authorized_keys`。SSH daemon 預設監聽 `0.0.0.0:22` 並允許 root 登入。如果這把金鑰在所有 Vestaboard 之間共用（極可能如此），**任何同網段的人都能取得 root 權限**
3. **大量遺留程式碼**——未使用的 legacy API 散落各處，冗餘邏輯隨處可見

### 離線化步驟

作者的改造流程：

1. **取出 CM** — 用酒精小心移除防拆貼紙（不觸發 tamper indicator）
2. **Dump 韌體** — 用另一塊 CM3+ 供電並掛載檔案系統，`dd if=/dev/sdb of=vestaboard_fs.bin bs=16M` 完整備份
3. **分析韌體** — 移除雲端依賴的 Docker 容器和服務
4. **寫入自訂韌體** — 加入自己的 SSH 金鑰、啟動腳本，移除雲端 call-home
5. **本地 API** — 透過逆向的序列協議，建立裝置級的本地 API

## 技術亮點

### 序列協議完全逆向

這是整個專案最精彩的部分。作者完整逆向了 CM 與翻頁控制器之間的通訊協議：

**封包結構：**
```
[0xFF 魔術位元] [2B 位址(柱ID)] [1B 旗標] [2B 載荷長度] [載荷...] [4B CRC32]
```

**翻頁流程：**
```
對每一柱 (1-22):
  → SetTarget(柱ID, 6字元資料)  ← ACK
  → Arm(柱ID)                   ← ACK

→ Go(位址: 1)                    ← ACK
// 每個封包之間需要 ~40ms 延遲，否則控制器會失去同步
```

6 個封包類型（SetTarget / Arm / Go / Ping / WriteRegister / ReadRegister）涵蓋了顯示器的完整控制能力。有了這份協議文件，任何人都可以用任意語言直接驅動翻頁片，完全繞過原廠軟體堆疊。

### 開源生態系的力量

這次逆向催生了 **FiestaBoard** 這個開源專案——一個完整的 Vestaboard 替代平台：

- **插件系統**：天氣、股票、交通、運動、衝浪條件等即時資訊源
- **自架部署**：Docker 一鍵啟動，或直接燒錄 RPi 映像檔
- **Home Assistant 整合**：HA Add-on 一鍵安裝，MQTT 自動發現
- **多機型支援**：Flagship (22×6)、Note (15×3)、Note 陣列

一台被雲端綁架的 $999 顯示器，現在可以用一塊 $15 的 RPi Zero 2W 當控制器，跑完全本地的開源軟體。

### 安全漏洞揭露

作者在 2022 年 3 月向 Vestaboard 回報了共用 SSH 金鑰的漏洞，但廠商未予回應。這意味著：

- 同一區域網路內的攻擊者可以 SSH 進入任何 Vestaboard
- 可修改韌體、植入後門、取得持久性網路存取
- 甚至可能讓裝置變磚

這是 IoT 安全的經典反面教材：硬體做得精美，軟體卻連最基本的金鑰管理都沒做好。

## 心得與延伸

### Right to Repair 的最佳案例

這個專案完美詮釋了為什麼 right-to-repair 如此重要。一台 $999 的硬體：

- 韌體更新失敗 → 變磚
- 雲端服務中斷 → 無法使用
- 公司倒閉 → 永久報廢

當作者 root 了 CM4、移除雲端依賴後，這台顯示器才真正成為**他的**。不再受制於伺服器狀態、不再擔心服務終止、不再需要網路連線就能翻頁。

### 延伸可能

- **自製 split-flap 顯示器**：如果 $999 太貴，[scottbez1/splitflap](https://github.com/scottbez1/splitflap) 是一個 ESP32 驅動的 DIY 方案，成本可控在 $200 以內
- **CM4 升級**：原廠 CM3+ 效能有限，換上 CM4 可以跑更複雜的本地 AI 模型（例如用 LLM 生成每日金句）
- **整合智慧家庭**：透過 FiestaBoard 的 Home Assistant 插件，讓翻頁顯示器成為家中的資訊中樞——門鈴響了翻「有人來了」、洗衣機洗完翻「衣服好了」

### 給 IoT 廠商的啟示

> 「硬體做得很好，外觀和手感都是一流的。可惜韌體就不能這麼說了。」——逆向工程作者

5 個 Docker 容器跑在一塊 CM3+ 上、共用 SSH 金鑰、冗餘的遺留程式碼——這些問題在一台 $999 的產品上出現，實在令人失望。IoT 產品的軟體品質和安全性，不應該是事後才想到的東西。

## 參考資料

- 原文 Reddit 貼文：https://www.reddit.com/r/raspberry_pi/comments/1waa0pn/rooted_my_vestaboards_compute_module_4_and_made/
- Vestaboard 逆向工程 GitHub：https://github.com/EngineOwningSoftware/Vestaboard-Reverse-Engineering
- 序列協議文件：https://github.com/EngineOwningSoftware/Vestaboard-Reverse-Engineering/blob/main/Protocol.md
- FiestaBoard 開源替代平台：https://github.com/Fiestaboard/FiestaBoard
- DIY Split-Flap 顯示器：https://github.com/scottbez1/splitflap
- Vestaboard Local API 官方文件：https://docs.vestaboard.com/docs/local-api/endpoints/
