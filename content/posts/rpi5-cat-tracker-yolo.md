---
title: "你的貓去哪了？— RPi 5 × YOLOv11 AI 貓咪追蹤攝影機"
date: 2026-06-22
draft: false
tags: ["Raspberry Pi 5", "YOLOv11", "電腦視覺", "Kalman Filter", "寵物科技", "Python", "Pan-Tilt"]
categories: ["專案推薦"]
description: "用 Raspberry Pi 5 打造 AI 貓咪追蹤攝影機，YOLOv11 偵測 + Kalman Filter 追蹤 + HSV 色彩辨識，雲台自動跟著貓跑。"
---

## 前言

養貓的人都知道，貓的行蹤永遠是個謎。牠可能在你轉頭的瞬間從沙發跳到書櫃頂端，也可能整個下午蜷在你找不到的角落睡覺。如果你曾經想過「要是有台攝影機能自動跟著我的貓就好了」——恭喜，有人真的做出來了，而且只需要一塊 Raspberry Pi 5。

這個專案在 Reddit 上引起不少討論：一台裝在雲台上的攝影機，搭配 YOLOv11 物件偵測模型，不只能即時找到畫面中的貓，還能透過 Kalman Filter 追蹤貓的移動軌跡，甚至用 HSV 色彩直方圖分辨「哪隻是橘貓、哪隻是虎斑」。最讓人心動的是，訓練方式超級簡單——只要拍你家貓咪一小段影片就行。

## 專案概述

### 硬體架構

| 元件 | 說明 | 參考價格 |
|------|------|---------|
| Raspberry Pi 5 (4GB/8GB) | 主控板，負責推論與伺服控制 | ~$60–80 |
| Camera Module v3 / USB 攝影機 | 影像擷取 | ~$25–35 |
| PCA9685 伺服驅動板 | I²C 控制多路 PWM | ~$3–5 |
| SG90 伺服馬達 ×2 | Pan（水平）+ Tilt（垂直）| ~$2–4 |
| Pan-Tilt 雲台支架 | 機構固定 | ~$5–10 |

**總成本估算：約 $95–135 美元**（不含 Pi 5 的話約 $35–55）

### 軟體堆疊

```
攝影機 → YOLOv11 偵測 → Kalman Filter 追蹤 → HSV 辨識 → PID/P 控制 → 伺服雲台
```

整個 pipeline 跑在 Raspberry Pi 5 上，不需要雲端，不需要 GPU 加速卡。Pi 5 的四核 Cortex-A76 搭配 Ultralytics 的 YOLOv11 nano/small 模型，足以達到可用的即時推論幀率。

## 技術亮點

### 1. YOLOv11 邊緣推論

YOLOv11 是 Ultralytics 在 2024 年底推出的最新版本，相比 YOLOv8 在同等模型大小下提升了精度，同時維持了輕量化的推論速度。在 Pi 5 上使用 `yolo11n`（nano）模型，搭配 NCNN 或 ONNX Runtime 後端，可以達到約 **5–15 FPS** 的偵測速率——對於追蹤一隻慵懶的家貓來說綽綽有餘。

```python
from ultralytics import YOLO

model = YOLO("yolo11n.pt")
results = model.predict(source=0, stream=True, classes=[15, 16])
# COCO class 15 = cat, 16 = dog
```

### 2. Kalman Filter 多目標追蹤

單靠 YOLO 逐幀偵測有個問題：當貓暫時被遮擋或跑出畫面邊緣時，偵測框會消失。Kalman Filter 的加入解決了這個痛點——它根據貓的歷史運動軌跡預測下一幀的位置，即使偵測暫時中斷也能維持追蹤。

核心概念：
- **預測（Predict）**：根據上一幀的位置和速度，估計這一幀貓會在哪裡
- **更新（Update）**：當 YOLO 偵測到貓時，用實際位置修正預測值
- **關聯（Association）**：多隻貓時，用匈牙利演算法將偵測框與追蹤器配對

這套方法其實就是經典的 SORT（Simple Online and Realtime Tracking）演算法的核心，被廣泛用於自駕車和監控系統。

### 3. HSV 色彩直方圖辨識個體

多貓家庭的終極問題：「這隻是小花還是小虎？」

這個專案用了一個聰明但簡單的方法——HSV 色彩直方圖。每隻貓在初次偵測時，系統會擷取其 bounding box 內的影像，轉換到 HSV 色彩空間後計算直方圖作為「貓咪指紋」。之後每次偵測到貓，就比對直方圖相似度來判斷是哪隻。

```python
import cv2

def get_cat_signature(frame, bbox):
    x1, y1, x2, y2 = bbox
    roi = frame[y1:y2, x1:x2]
    hsv = cv2.cvtColor(roi, cv2.COLOR_BGR2HSV)
    hist = cv2.calcHist([hsv], [0, 1], None, [50, 60], [0, 180, 0, 256])
    cv2.normalize(hist, hist)
    return hist
```

HSV 空間的好處是對光線變化比 RGB 更穩定——你家客廳白天和晚上的光線差很多，但貓的毛色在 Hue 通道上的分布相對一致。

### 4. 雲台伺服控制

偵測到貓的位置後，系統計算貓在畫面中心的偏移量，透過比例控制（P Control）驅動雲台的 Pan/Tilt 伺服馬達，讓攝影機「追著貓看」。PCA9685 驅動板透過 I²C 接收 PWM 指令，控制兩顆 SG90 馬達分別負責水平和垂直旋轉。

當多隻貓同時出現時，系統會優先追蹤「主目標」（例如最大的偵測框或指定的貓），避免雲台在兩隻貓之間瘋狂擺動。

## 心得與延伸

### 為什麼這個專案值得做？

這個專案的美妙之處在於它是**三種經典 CV 技術的完美組合**：

1. **深度學習偵測**（YOLO）→ 找到「哪裡有貓」
2. **狀態估計追蹤**（Kalman Filter）→ 預測「貓要去哪」
3. **特徵匹配辨識**（HSV 直方圖）→ 判斷「這是哪隻貓」

對學習電腦視覺的人來說，這三個技術各自都是重要的基礎知識，而這個專案把它們串在一起解決了一個有趣又實用的問題。

### 可能的延伸方向

- **加入 AI HAT+**：Raspberry Pi 的 AI 加速模組（Hailo-8L）可以將推論速度提升到 30+ FPS，讓追蹤更流暢
- **Slack / LINE 通知**：偵測到貓在特定區域（例如廚房流理台上）時自動發送警報
- **行為分析**：記錄貓的移動熱力圖，分析牠最常待的位置和活動時段
- **多攝影機聯動**：多個 Pi 各守一個房間，建立全屋貓咪追蹤網路
- **DeepSORT 升級**：將 HSV 直方圖替換為 Re-ID 深度特徵，提升多貓辨識的準確度

### 訓練門檻極低

最讓人驚豔的設計是訓練流程：「拍你的貓一小段影片就行。」不需要標註資料集、不需要雲端 GPU 訓練——系統只是擷取每隻貓的外觀特徵建立 profile，之後靠比對來辨識。這讓非 ML 背景的人也能輕鬆上手。

## 參考資料

- [原文討論 — Reddit r/raspberry_pi](https://www.reddit.com/r/raspberry_pi/comments/1uaaqao/cat_tracker_on_raspberry_pi_5/)
- [Ultralytics YOLOv11 官方文件](https://docs.ultralytics.com/)
- [類似專案：Pan-Tilt Pet Tracker（YOLOv8 版）](https://github.com/Murasan201/12-001-pan-tilt-pet-tracker)
- [SORT 追蹤演算法論文](https://arxiv.org/abs/1602.00763)
- [OpenCV HSV 色彩直方圖教學](https://docs.opencv.org/4.x/d8/dc8/tutorial_histogram_comparison.html)
