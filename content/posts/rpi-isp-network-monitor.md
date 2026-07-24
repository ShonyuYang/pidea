---
title: "ISP 不要再騙我 — RPi 24/7 網路監控 Bot 自動蒐證"
date: 2026-07-24
draft: false
tags: ["raspberry-pi", "python", "網路監控", "telegram-bot", "sqlite", "自動化"]
categories: ["專案推薦"]
description: "用 Raspberry Pi 打造 24/7 網路品質監控站，自動測速、掃描裝置、生成趨勢圖表，還附贈 AI 吐槽評論，讓你下次打客服電話時有數據撐腰。"
---

## 前言

「我們這邊看起來一切正常喔。」

每次打電話給 ISP 客服，得到的永遠是這句話。你明明感覺到網路每週會莫名變慢好幾次，但沒有數據就是沒有話語權。Reddit 用戶 u/GoldBroccoli7073 受夠了這種「口說無憑」的窘境，用一台 Raspberry Pi 打造了 **netmon** — 一個 24 小時不間斷的網路品質監控機器人，自動蒐集證據，下次跟客服吵架時直接甩出圖表。

## 專案概述

**netmon** 是一個輕量級的 Python bot，設計哲學是「100% 自架、零雲端依賴」。所有數據都存在本地 SQLite 資料庫，只有你主動選擇發送的報告才會離開你的網路。

### 運作流程

| 週期 | 動作 | 工具 |
|------|------|------|
| 每小時 | 執行完整測速（下載/上傳/延遲/ISP/伺服器） | `speedtest-cli` |
| 每小時 | ARP 掃描區網，計算連線裝置數量 | `nmap` |
| 每小時 | 寫入本地資料庫 | SQLite |
| 每小時 | 發送簡短狀態更新 | Telegram API |
| 每 4 小時 | 生成 24 小時趨勢圖 + AI 吐槽評論 | matplotlib + OpenAI API |

### 技術堆疊

- **Python 3.13+**（透過 `uv` 管理）— 核心執行環境
- **SQLite** — 本地持久化，零外部依賴
- **speedtest-cli** — 測量頻寬和延遲
- **nmap** — ARP 掃描發現區網裝置
- **matplotlib** — 24 小時趨勢視覺化
- **OpenAI-compatible API** — 生成吐槽風格的分析報告（可選，支援本地 LLM）
- **Telegram Bot API** — 推送警報和圖表

### 硬體需求

只需要一台 Raspberry Pi（任何型號都行）加上網路連線。沒有額外的感測器、沒有特殊 HAT，純軟體方案。

## 技術亮點

### 1. ARP 掃描的巧妙設計

要掃描區網裝置需要 raw socket 權限（root），但作者不想讓整個 bot 以 root 執行。解法是設定 `nmap` 的 passwordless sudo，只開放最小權限：

```bash
# /etc/sudoers.d/netmon
your_user ALL=(ALL) NOPASSWD: /usr/bin/nmap
```

這樣 bot 以普通使用者身份執行，只在需要 ARP 掃描時透過 sudo 呼叫 nmap，安全又實用。

### 2. 數據關聯分析

這個專案最有價值的不只是「記錄網速」，而是**同時記錄裝置數量**。作者發現：

> 網速會在區網裝置數超過某個門檻時急劇下降，而且發生在特定時段。

這讓他能區分「是家裡太多裝置在搶頻寬」還是「ISP 在尖峰時段超賣線路」— 這個判斷光靠測速是做不到的。

### 3. AI 吐槽評論

每 4 小時的報告不只有冷冰冰的數字，還附帶一段 AI 生成的「吐槽式分析」：

> "someone's hogging the bandwidth again"

這個功能原本是開玩笑加的，但實際上非常有用 — 它把複雜的數據趨勢濃縮成一句人話，讓你一眼就能看出問題。而且支援任何 OpenAI-compatible API，包括本地跑的 LLM，不一定要花錢。

### 4. 社群驗證的實戰價值

Reddit 留言中有人分享了類似的成功案例：用 InfluxDB 記錄網速後，發現問題跟**下雨**有關。進一步追查發現是社區的同軸電纜絕緣層破損，雨水滲入導致訊號品質劣化。最後 ISP 派了大型工程車來整條街更換電纜 — 這一切都始於一個 Pi 上的監控腳本。

## 心得與延伸

### 為什麼這個專案值得關注

這不是又一個「用 Pi 測網速」的教學。netmon 的設計有幾個值得學習的地方：

1. **隱私優先**：所有數據留在本地，只有報告透過 Telegram 發送
2. **多維度監控**：同時追蹤速度和裝置數，能做交叉分析
3. **可操作的輸出**：不只是記錄，還能生成可以拿給客服看的圖表
4. **低維護設計**：設定完就忘了它，24/7 自動執行

### 延伸可能

- **加入 Grafana 儀表板**：SQLite 數據可以匯入 InfluxDB 或直接用 Grafana 的 SQLite plugin，做更漂亮的長期趨勢分析
- **多節點部署**：在不同房間放 Pi，比較 WiFi 死角
- **自動化客訴**：當連續 N 次測速低於合約速度時，自動生成投訴信件草稿
- **搭配 UPS**：加個小型 UPS 確保斷電時也能記錄到「ISP 斷線」事件

### 成本估算

| 項目 | 價格（USD） |
|------|------------|
| Raspberry Pi Zero 2 W | ~$15 |
| microSD 卡 | ~$8 |
| 電源供應器 | ~$10 |
| **總計** | **~$33** |

用不到一千台幣，就能建立一個專業級的網路品質監控站。

## 參考資料

- 🔗 原文：[Reddit - I built a Pi bot that watches my home network 24/7](https://www.reddit.com/r/raspberry_pi/comments/1v3t2uc/i_built_a_pi_bot_that_watches_my_home_network_247/)
- 📦 GitHub Repo：[Role1776/netmon](https://github.com/Role1776/netmon)（MIT License）
