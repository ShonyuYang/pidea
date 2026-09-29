---
title: "你的照片不該住在別人家 — RPi 5 開源家用雲端 Keephaven 完全解析"
date: 2026-09-29
draft: false
tags: ["Raspberry Pi 5", "NixOS", "self-hosted", "私有雲", "Immich", "Jellyfin"]
categories: ["專案推薦"]
description: "一個人寫的完整家用雲端 OS，六個自架服務、一鍵燒錄、簽章更新、災難復原——全部開源免費。"
---

## 前言

你的照片存在 Google Photos，影片放 Netflix，音樂聽 Spotify，有聲書用 Audible，新聞靠 Feedly——每個月繳一堆訂閱費，資料散落在五六間公司的伺服器裡。哪天任何一家改價、關服、砍功能，你就只能乖乖配合。

如果告訴你，一台 RPi 5 加一顆 NVMe SSD，就能把這些全部搬回家呢？

Keephaven 就是這樣的專案：一個人獨力開發的完整家用雲端 OS，2026 年 9 月 25 日正式開源。不是「教你裝 Docker」的教學文，而是**燒進去就能用的完整系統映像檔**。

## 專案概述

### 這是什麼？

Keephaven 是一個基於 NixOS 的完整作業系統映像檔，專為 RPi 5 設計。燒錄到 NVMe SSD 後開機，三分鐘內自動建立 Wi-Fi 熱點，六個自架服務全部就緒。

### 六大內建服務

| 用途 | 服務 | 取代什麼 |
|------|------|----------|
| 📸 照片 | Immich | Google Photos |
| 🎬 影片 | Jellyfin | Netflix / Plex |
| 🎵 音樂 | Navidrome | Spotify |
| 🎧 有聲書 | Audiobookshelf | Audible |
| 📚 書籍漫畫 | Kavita | Kindle / 漫畫 App |
| 📰 新聞 | FreshRSS | Feedly |

### 硬體需求

- **RPi 5**（8 GB 版本測試通過）
- **NVMe SSD**（搭配 Pi 5 NVMe 外殼或 HAT，測試用 Samsung 990 PRO 1 TB）
- 系統佔 35.4 GB，剩餘空間全部給你的檔案
- 一台 Linux 電腦用來燒錄和讀取初始密碼

### 預估成本

| 零件 | 價格（美元） |
|------|-------------|
| RPi 5 8GB | ~$80 |
| NVMe SSD 1TB | ~$80 |
| NVMe HAT/外殼 | ~$15 |
| 電源供應器 | ~$12 |
| **合計** | **~$187** |

一次性支出，之後零月費。

## 技術亮點

### NixOS 宣告式架構

整個 OS 就是一個 NixOS flake，所有設定都用程式碼定義。這意味著：

- **完全可重現**：同一份 flake 建出的映像檔完全一致
- **原子更新**：更新要嘛全成功，要嘛全回滾，不會卡在半路
- **透明可審計**：整個 OS 的每一行設定都在 GitHub 上

### 離線即用

Keephaven 開機後會自動建立自己的 Wi-Fi 網路（`Keephaven-XXXXX`）。**不需要路由器，不需要網際網路**——你的手機連上這個 Wi-Fi 就能存取所有服務。

所有 Docker 映像檔都以 digest 鎖定並預先烘焙進系統映像，執行時完全不需要從 Docker Hub 拉取任何東西。

### 簽章更新機制

系統每天檢查一次更新，但：

1. 只會在儀表板上**顯示**有更新可用
2. 你手動點「Install now」才會安裝
3. 更新檔必須通過 `keys/keephaven-update.pub` 的簽章驗證
4. 不信任官方更新伺服器？換成你自己的 key 和 URL

### 災難復原分區

系統自帶一個獨立的 recovery 分區。如果主系統分區損壞，下次開機自動從備份復原。家用 NAS 最怕的就是資料遺失，這個設計很實在。

### Box-to-Box 備份

透過 Tailscale 配對兩台 Keephaven，一台每晚自動備份到另一台。異地備援的 3-2-1 備份策略，用兩台 RPi 就能實現。

### 隱私設計

系統唯一的對外連線是每天一次的更新檢查，只帶版本號在 User-Agent 裡。沒有帳號、沒有序號、沒有遙測。遠端存取（Tailscale）預設關閉，要用你自己開。

## 心得與延伸

### 為什麼這個專案特別？

Self-hosted 社群不缺教學，但大多數方案都是「裝好 OS → 裝 Docker → 拉 image → 改 config → 設定反向代理 → 設定 SSL → ...」，每一步都可能踩坑。Keephaven 把這整條路徑壓縮成**一次燒錄**，而且是以 NixOS 的方式做到可重現、可審計。

作者 [@Dafarusd](https://github.com/dafarusd) 一個人完成這整個 OS，從磁碟分割、Wi-Fi AP、recovery 分區、簽章更新到六個服務的整合，工程量驚人。

### 可以改進的方向

- **4 GB Pi 5 支援**：目前只測試 8 GB 版本，六個服務同時跑可能吃緊
- **SD 卡支援**：目前只支援 NVMe，對入門者門檻較高
- **初始密碼設定**：需要把 SSD 拔出來用 Linux 電腦讀取，稍嫌麻煩（HDMI 輸出密碼會更友善）
- **Vaultwarden**：程式碼裡已經有但還沒啟用，密碼管理器加進來就更完整了

### 延伸玩法

- 加裝外接硬碟擴充儲存空間，當家庭媒體伺服器
- 搭配 Tailscale 從外面存取家裡的照片和影片
- 用 FreshRSS 搭配 AI 摘要，打造個人新聞策展系統
- 兩台 Keephaven 分別放家裡和辦公室，互相備份

## 參考資料

- 🔗 原文：[Reddit — Open sourced my Raspberry Pi 5 home cloud today](https://www.reddit.com/r/raspberry_pi/comments/1wqbpnq/open_sourced_my_raspberry_pi_5_home_cloud_today/)
- 💻 GitHub：[dafarusd/keephaven](https://github.com/dafarusd/keephaven)
- 📥 映像檔下載：[keephaven-2026.08.19.img.zst](https://github.com/dafarusd/keephaven)（6.3 GB 壓縮 / 35.4 GB 解壓）
- 📄 授權：AGPL-3.0
