---
title: "礦機不用顧 — RPi 3 自癒式 USB 礦機管理系統"
date: 2026-07-03
draft: false
tags: ["Raspberry Pi", "Python", "Flask", "tmux", "watchdog", "自動化", "挖礦"]
categories: ["專案推薦"]
description: "用 Raspberry Pi 3 打造自動修復的 USB 礦機管理系統，watchdog 偵測故障、USB 電源循環重啟、Flask 儀表板即時監控，終結每天手動重啟的噩夢。"
---

## 前言

挖礦最浪漫的部分是「讓機器幫你賺錢」，最不浪漫的部分是「機器每天當機好幾次，你得手動重啟」。這位作者用 5 台 Moonlander 2 USB 礦機掛在一台 Raspberry Pi 3 上，原本每天要跑去按好幾次重啟。後來他受不了了，花時間寫了一套自癒式管理系統——watchdog 自動偵測故障、USB hub 電源循環自動重啟、Flask 儀表板即時監控，從此再也不用 babysit 礦機。

這不是什麼高深的 DevOps 架構，但它解決了一個真實的痛點，而且用的全是 Linux 生態系裡最基本的工具。

## 專案概述

### 系統架構

```
┌─────────────────────────────────┐
│         Flask Dashboard         │
│   (狀態 / 溫度 / 算力 / 日誌)    │
└──────────────┬──────────────────┘
               │ HTTP
┌──────────────┴──────────────────┐
│        Raspberry Pi 3           │
│  ┌──────────┐  ┌─────────────┐  │
│  │ Watchdog │  │ tmux 管理器  │  │
│  │  監控迴圈 │  │ (5 sessions) │  │
│  └────┬─────┘  └──────┬──────┘  │
│       │               │         │
│  ┌────┴───────────────┴──────┐  │
│  │    USB Hub 電源控制        │  │
│  └────┬───┬───┬───┬───┬─────┘  │
│       │   │   │   │   │         │
│      M1  M2  M3  M4  M5        │
│    (Moonlander 2 USB 礦機)      │
└─────────────────────────────────┘
```

### 核心元件

| 元件 | 用途 |
|------|------|
| Raspberry Pi 3 | 主控制器，跑所有管理服務 |
| 5× Moonlander 2 | USB ASIC 礦機 |
| USB Hub | 帶電源控制的 USB 集線器 |
| Watchdog 腳本 | 定期檢查礦機狀態，偵測離線或異常 |
| Flask Dashboard | 網頁介面，顯示即時狀態 |
| tmux | 管理每台礦機的獨立終端 session |

### 運作流程

1. 每台礦機在獨立的 tmux session 中運行
2. Watchdog 腳本定期輪詢各礦機狀態（算力、溫度、回應時間）
3. 偵測到異常（離線、算力歸零、無回應）→ 觸發修復流程
4. 修復流程：停止礦機程序 → USB hub 電源循環 → 等待裝置重新列舉 → 重啟礦機
5. Flask Dashboard 即時顯示所有狀態，方便遠端查看

## 技術亮點

### USB 電源循環（Power Cycling）

這是整個系統的核心技巧。當 USB 礦機當掉時，光是軟體重啟通常沒用——你需要斷電再上電，讓 USB 裝置重新列舉。在 Linux 上可以透過幾種方式做到：

- **uhubctl**：直接控制支援 per-port power switching 的 USB hub
- **sysfs**：寫入 `/sys/bus/usb/devices/.../authorized` 來停用/啟用裝置
- **GPIO 控制繼電器**：用 RPi 的 GPIO 控制繼電器切斷 USB hub 電源

不管哪種方式，關鍵是要等待足夠的時間讓裝置完全斷電，再等 USB 重新列舉完成後才啟動礦機程序。

### tmux 多 Session 管理

用 tmux 管理多個礦機程序是個聰明的選擇：

- 每台礦機一個 session，互不干擾
- 可以隨時 attach 進去看即時輸出
- 程序崩潰不會影響其他 session
- Watchdog 可以透過 `tmux send-keys` 重啟特定礦機

```bash
# 建立 session
tmux new-session -d -s miner1 'cgminer --config miner1.conf'

# 重啟特定礦機
tmux send-keys -t miner1 C-c
sleep 2
tmux send-keys -t miner1 'cgminer --config miner1.conf' Enter
```

### Flask 即時儀表板

用 Flask 做監控儀表板是 RPi 專案的經典搭配。輕量、易寫、Python 生態豐富。關鍵功能：

- **即時狀態**：各礦機的線上/離線狀態、當前算力、溫度
- **歷史記錄**：故障次數、重啟次數、累計運行時間
- **日誌檢視**：直接在網頁上看 watchdog 的操作日誌
- **手動控制**：必要時可以從網頁手動觸發重啟

### Watchdog 設計要點

一個好的 watchdog 不只是「偵測到問題就重啟」，還需要：

- **去抖動**：偶爾的瞬間異常不應該觸發重啟，連續 N 次異常才行動
- **冷卻期**：重啟後要等一段時間再開始監控，避免重啟風暴
- **最大重試次數**：同一台礦機短時間內重啟太多次，應該停止並告警
- **日誌記錄**：每次操作都要記錄，方便事後分析

## 心得與延伸

這個專案的價值不在挖礦本身（Moonlander 2 的算力在 2026 年大概只能挖到電費的零頭），而在於它展示了一套完整的「無人值守設備管理」思路。同樣的架構可以套用到任何需要自動修復的場景：

### 延伸應用

- **3D 列印農場**：監控多台印表機，偵測到失敗自動停止並通知
- **IoT 閘道器**：管理多個感測器節點，自動重啟離線裝置
- **家用伺服器**：監控 Docker 容器或服務，自動重啟崩潰的服務
- **網路設備**：監控路由器/交換機，偵測到斷線自動重啟 PoE 埠

### 可以加的功能

- **Telegram/Line 通知**：故障和修復時推送訊息
- **Grafana + InfluxDB**：長期數據視覺化
- **MQTT 整合**：接入 Home Assistant 統一管理
- **自動更新**：定期拉取最新的礦機軟體版本

### 預估成本

| 項目 | 價格（USD） |
|------|------------|
| Raspberry Pi 3 | ~$35 |
| USB Hub（帶電源控制） | ~$15-30 |
| Moonlander 2 × 5 | 已停產，二手價不定 |
| **管理系統成本** | **~$50-65** |

## 參考資料

- [原文 Reddit 貼文](https://www.reddit.com/r/raspberry_pi/comments/1ul85em/built_a_selfhealing_mining_management_system_on_a/)
- [uhubctl — USB hub 電源控制工具](https://github.com/mvp/uhubctl)
- [tmux 官方文件](https://github.com/tmux/tmux/wiki)
- [Flask 快速入門](https://flask.palletsprojects.com/)
