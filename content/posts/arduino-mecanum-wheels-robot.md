---
title: "麥克納姆輪全向機器人 — Arduino 實現 360° 自由移動"
date: 2026-06-13
draft: false
tags: ["Arduino", "機器人", "麥克納姆輪", "3D列印", "藍牙遙控"]
categories: ["專案推薦"]
description: "用 Arduino 搭配麥克納姆輪打造全向移動機器人，前後左右斜向旋轉全都行，還能用手機藍牙遙控。"
---

## 前言

一般的輪子只能前進後退，轉彎得靠差速。但如果你的機器人可以像螃蟹一樣橫著走呢？

**麥克納姆輪（Mecanum Wheel）** 是一種在輪面上裝了 45° 斜角滾子的特殊輪子，四個輪子透過不同的轉速組合，可以讓機器人往任何方向移動——前後、左右、斜向，甚至原地旋轉。這個來自 HowToMechatronics 的經典教學，帶你從零開始打造一台全向移動機器人。

## 專案概述

### 核心規格

| 項目 | 規格 |
|------|------|
| 控制器 | Arduino Mega 2560（或 Uno + 馬達擴展板） |
| 馬達 | 4 × NEMA 17 步進馬達（進階版）或 4 × TT 減速馬達（入門版） |
| 驅動 | 4 × DRV8825 / A4988 步進驅動（或 L293D 擴展板） |
| 輪子 | 4 × 麥克納姆輪（可購買或 3D 列印） |
| 通訊 | NRF24L01 無線模組 / HC-05 藍牙模組 |
| 電源 | 3S LiPo 電池（11.1-12.6V） |
| 遙控 | 自製遙控器 / 手機 App（藍牙） |
| 預估成本 | NT$1,500-3,000（視輪子來源） |

### 運動學原理

麥克納姆輪的魔法在於那些 45° 斜角滾子。當輪子轉動時，滾子會產生一個斜向的分力：

```
前進：四輪同向正轉
後退：四輪同向反轉
右移：左輪正轉 + 右輪反轉（對角配對）
左移：左輪反轉 + 右輪正轉（對角配對）
右前斜移：僅左前 + 右後兩輪轉動
原地右旋：左側正轉 + 右側反轉
```

關鍵是四個輪子必須按照特定排列安裝——**每個輪子頂部滾子的軸線必須指向機器人中心**，形成 X 型排列。裝錯方向，機器人就會做出各種詭異的動作。

### 材料清單（入門版）

| 材料 | 數量 | 參考價格 |
|------|------|---------|
| Arduino Uno R3 | 1 | NT$150 |
| L293D 馬達擴展板 | 1 | NT$80 |
| TT 減速馬達 | 4 | NT$120 |
| 麥克納姆輪組（48mm）| 1 組 | NT$400-800 |
| HC-05 藍牙模組 | 1 | NT$100 |
| 18650 電池座 + 電池 | 1 | NT$150 |
| 4WD 底盤 | 1 | NT$150 |
| 杜邦線、螺絲等 | 若干 | NT$50 |
| **合計** | | **NT$1,200-1,600** |

## 技術亮點

### 3D 列印自製麥克納姆輪

現成的麥克納姆輪價格不便宜（一組四個約 NT$800-2,000），但 HowToMechatronics 的作者 Dejan 提供了完整的 3D 列印設計檔：

- 每個輪子由**內外兩片 + 10 個滾子 + 軸承座**組成
- 滾子軸心用 3mm 鋼絲裁切（每根約 40mm）
- M4 螺絲固定內外片，M3 墊圈讓滾子順暢轉動
- 軸承座專為 NEMA 17 步進馬達軸設計

自己印一組四個輪子的材料成本大約只要 NT$100-150（PLA 耗材 + 鋼絲 + 螺絲），省下不少預算。

### 步進馬達 vs 直流馬達

這個專案有兩種路線：

**進階版（步進馬達）：**
- NEMA 17 步進馬達提供精確的速度和位置控制
- 可以實現「自動巡航」功能——錄製路徑後自動重播
- 需要 DRV8825 驅動板，接線較複雜
- 成本較高但控制精度好

**入門版（直流馬達）：**
- TT 減速馬達 + L293D 擴展板，接線簡單
- 用 `AFMotor` 函式庫控制四個馬達
- 藍牙接收指令，根據指令組合控制四輪轉向
- 成本低、上手快，適合第一次做機器人的人

### 手機遙控 App

透過 HC-05 藍牙模組，可以用手機 App 遙控機器人。控制邏輯很直覺：

```cpp
void moveSidewaysRight() {
  motor1.run(BACKWARD);  // 右前輪反轉
  motor.run(FORWARD);    // 左前輪正轉
  motor3.run(BACKWARD);  // 右後輪反轉
  motor2.run(FORWARD);   // 左後輪正轉
}
```

App 發送數字指令（0-12），Arduino 接收後對應到不同的移動模式。還可以透過滑桿調整 `wheelSpeed` 控制移動速度。

## 心得與延伸

麥克納姆輪機器人是學習機器人運動學的絕佳教材。它不只是「讓輪子轉」這麼簡單，而是要理解力的分解與合成——每個輪子產生的斜向力如何疊加成你想要的移動方向。

### 延伸方向

1. **PID 速度控制**：加裝編碼器，用 PID 演算法讓四輪轉速更精確一致
2. **自動避障**：加超音波或紅外線感測器，實現自主導航
3. **ROS 整合**：升級到 Raspberry Pi，跑 ROS 做路徑規劃
4. **視覺循跡**：加攝影機做影像辨識，自動跟隨物體或循線
5. **機械手臂**：在車體上加裝機械手臂，變成移動操作平台

這個專案的學習曲線適中——入門版用 Arduino Uno + L293D 擴展板，一個週末就能完成基本功能；進階版用步進馬達 + 自製 PCB，可以玩上好幾個月。不管哪個版本，看到機器人第一次成功橫移的那一刻，都會忍不住喊「好酷！」。

## 參考資料

- [HowToMechatronics - Arduino Mecanum Wheels Robot（完整教學 + 3D 列印檔）](https://howtomechatronics.com/projects/arduino-mecanum-wheels-robot/)
- [Arduino Project Hub - Building an Arduino Mecanum Wheels Robot](https://projecthub.arduino.cc/lee_curiosity/building-an-arduino-mecanum-wheels-robot-26edbb)
- [YouTube - Arduino Mecanum Wheel Robot DIY Guide](https://www.youtube.com/watch?v=VhBQXkIwL9A)
- [Luis Llamas - Robot with Mecanum Wheel controlled by Arduino](https://www.luisllamas.es/en/mecanum-wheel-robot-controlled-by-arduino/)
