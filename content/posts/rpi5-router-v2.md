---
title: "自己的路由器自己造 — RPi 5 雙 2.5G 網卡全能家用網關"
date: 2026-08-10
draft: false
tags: ["Raspberry Pi 5", "路由器", "RaspAP", "Home Assistant", "NVMe", "網路", "自動化"]
categories: ["專案推薦"]
description: "用 RPi 5 + U2500 雙 2.5G 網卡 HAT + NVMe SSD 打造一台功能完整的家用路由器，跑 RaspAP 管 WiFi、Home Assistant 管智慧家庭，一台搞定。"
---

## 前言

你家的路由器是 ISP 送的那台嗎？功能陽春、介面醜、想裝個 Ad Blocker 還得另外買設備？

一位 Reddit 使用者決定用 Raspberry Pi 5 打造第二代自製路由器，配備 **雙 2.5G 乙太網路**、**1TB NVMe SSD**、**RaspAP** 無線管理、外加 **Home Assistant** 智慧家庭中樞 — 全部塞進一個自製壓克力外殼裡。這不是玩具路由器，這是一台認真的家用網路閘道器。

## 專案概述

### 硬體清單

| 零件 | 規格 | 用途 |
|------|------|------|
| Raspberry Pi 5 | 8GB RAM | 主控 |
| 52Pi U2500 HAT | 雙 USB3.0 轉 2.5G 乙太網路 + M.2 NVMe | 網路 + 儲存擴充 |
| NVMe SSD | 1TB M.2 2230/2242 | 系統碟 + 儲存 |
| WiFi Dongle | USB 外接 | 無線 AP |
| 5V 2pin USB 風扇 | — | 散熱 |
| 自製壓克力外殼 | — | 整線 + 保護 |

### 軟體堆疊

- **Pi OS Lite (Debian 13 Trixie)** — 無桌面環境，資源全給網路服務
- **RaspAP** — Web UI 管理無線 AP，支援 20+ 語言、VPN、Ad Blocking
- **Home Assistant** — 智慧家庭自動化平台

### 架構概念

```
[WAN] ──→ [U2500 ETH0] ──→ RPi 5 ──→ [U2500 ETH1] ──→ [LAN]
                              │
                              ├── RaspAP (WiFi AP)
                              ├── Home Assistant
                              └── NVMe SSD (系統 + 資料)
```

Pi 5 透過 U2500 HAT 取得兩個 2.5G 網路介面（WAN + LAN），搭配板載 WiFi 或 USB WiFi Dongle 提供無線 AP，形成完整的三介面路由器。

## 技術亮點

### U2500 HAT：一片板子解決兩個問題

Pi 5 原生只有一個 Gigabit 乙太網路口，做路由器至少需要 WAN + LAN 兩個口。52Pi 的 U2500 HAT 巧妙地利用 **PCIe x1 接 NVMe SSD** + **兩個 USB 3.0 口轉 2.5G 乙太網路**，一片板子同時解決儲存和網路的痛點。

設定也不複雜，只需在 `/boot/firmware/config.txt` 加兩行：

```
dtparam=pciex1
dtparam=pciex1_gen=3
```

重開機後 NVMe 和雙網卡就能用了。

### 效能實測：Pi 5 路由器的天花板在哪？

根據社群實測數據：

| 配置 | WAN 吞吐量 | 搭配 IDS | 功耗 |
|------|-----------|---------|------|
| Pi 5 (8GB) | 500-700 Mbps | 200-400 Mbps | 6-8W |
| Pi 5 + USB 網卡 | ~500 Mbps | ~200 Mbps | 7-9W |
| Pi 4 (4GB) | 200-300 Mbps | 50-100 Mbps | 4-6W |

**500 Mbps 以下的寬頻**，Pi 5 路由器完全勝任。如果你家是 1Gbps 光纖且需要跑入侵偵測（IDS），那可能會碰到瓶頸 — 但老實說，大多數台灣家庭 300M 方案用 Pi 5 綽綽有餘。

### NVMe 取代 SD 卡：24/7 運行的關鍵

路由器是 24/7 不關機的設備，SD 卡在持續讀寫下平均 18-24 個月就會出問題。改用 NVMe SSD 不只速度快（PCIe Gen3），更重要的是**壽命和可靠度**大幅提升。1TB 的容量還能順便當 NAS 用。

### RaspAP：最簡單的無線 AP 管理

RaspAP 提供了一個漂亮的 Web 介面來管理 WiFi AP，安裝只需一行指令：

```bash
curl -sL https://install.raspap.com | bash
```

功能包括：
- 多 SSID 管理
- WPA2/WPA3 加密
- DHCP 設定
- VPN 整合（OpenVPN、WireGuard）
- Ad Blocking（內建 DNS 過濾）
- 即時流量監控

### Home Assistant：路由器 + 智慧家庭一體機

把 Home Assistant 跑在路由器上是個聰明的選擇 — 反正路由器 24/7 開著，Pi 5 的 8GB RAM 足夠同時處理路由和自動化任務。你的 Zigbee 燈泡、溫濕度感測器、門鎖全都可以在同一台機器上管理。

## 成本估算

| 項目 | 價格（約） |
|------|----------|
| Raspberry Pi 5 8GB | NT$2,800 |
| 52Pi U2500 HAT | NT$1,800 |
| 1TB NVMe SSD (2242) | NT$2,000 |
| USB WiFi Dongle | NT$300 |
| 散熱風扇 | NT$150 |
| 壓克力外殼（自製/3D列印） | NT$200 |
| **合計** | **約 NT$7,250** |

對比一台中階家用路由器（NT$3,000-5,000），Pi 5 方案貴了一些，但你得到的是：
- 完整的 Linux 系統，想裝什麼就裝什麼
- Home Assistant 智慧家庭中樞
- 1TB NVMe 儲存空間
- Ad Blocking、VPN、DNS 過濾
- 完全的控制權和可擴充性

## 心得與延伸

### 誰適合這個專案？

這不是給「只想上網」的人。如果你符合以下任一條件，這個專案值得一試：

1. **家裡有智慧家庭設備** — 省掉一台 HA 主機的錢
2. **受夠了 ISP 路由器** — 想要 Ad Blocking、VPN、自訂 DNS
3. **想學網路** — 從 iptables、DHCP、NAT 到 VLAN，全都可以實際操作
4. **家用寬頻 ≤ 500Mbps** — Pi 5 的效能甜蜜點

### 可以再加什麼？

- **Pi-hole** — DNS 層級廣告過濾，全家設備都受益
- **Unbound** — 本地遞迴 DNS，不再把查詢紀錄送給第三方
- **WireGuard VPN** — 出門在外安全連回家
- **Grafana + Prometheus** — 網路流量視覺化監控
- **Tailscale** — 零設定的 Mesh VPN

### 與其他方案的比較

| 方案 | 價格 | 吞吐量 | 擴充性 | 學習價值 |
|------|------|--------|--------|---------|
| ISP 路由器 | 免費 | 看型號 | ❌ | ❌ |
| 中階家用路由器 | NT$3-5K | 1Gbps+ | ⚠️ | ❌ |
| Pi 5 路由器 | ~NT$7.2K | ~500Mbps | ✅ | ✅ |
| x86 Mini PC (OpenWrt) | NT$3-5K | 2.5Gbps+ | ✅ | ✅ |

如果純粹追求效能，x86 Mini PC 跑 OpenWrt 是更好的選擇。但如果你想要**路由器 + 智慧家庭 + NAS 三合一**，Pi 5 方案的整合度無人能敵。

### 注意事項

- **散熱很重要** — 路由器 24/7 運行，Pi 5 又是出了名的熱，主動散熱必備
- **USB WiFi Dongle 的選擇** — 不是每個都能當 AP 用，確認晶片支援 AP 模式
- **備份** — NVMe 比 SD 卡可靠，但還是建議定期備份設定檔

## 參考資料

- [原文 Reddit 貼文](https://www.reddit.com/r/raspberry_pi/comments/1vj6r8k/pi_5_router_v2_up_and_running/)
- [52Pi U2500 HAT Wiki](https://wiki.52pi.com/index.php?title=EP-0235)
- [RaspAP 官網](https://raspap.com/)
- [LaswitchTech Pi 5 Router 完整教學](https://laswitchtech.com/en/projects/router-pi5/index)
- [Jeff Geerling pi-router 專案](https://github.com/geerlingguy/pi-router)
