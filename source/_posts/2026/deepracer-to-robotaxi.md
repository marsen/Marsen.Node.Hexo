---
title: "[產業報告] 從 DeepRacer 到 Robotaxi：Tesla vs Waymo 的無人駕駛現況"
date: 2026/04/22 16:13:21
tags:
  - 產業報告
  - AI
  - 自動駕駛
---

## 前情提要

2019 年，我在 [AWS DeepRacer 工作坊](https://blog.marsen.me/2019/05/18/2019/aws_deepraceer/)裡用 Python 寫了幾行 reward function，讓一台 1:18 的小賽車學會跑彎道。

多年後，無人駕駛已經在真實道路上運作，這篇記錄一下產業現況。

## 兩條路線

### Tesla：純視覺派

馬斯克從第一性原理出發：人用兩隻眼睛就能開車，攝影機應該也夠。

2022 年起，Tesla 把 radar 全拔掉，FSD 只靠攝影機。

為什麼可行？道路本來就是為人類視覺設計的，攝影機加上夠大的神經網路應該能學會同樣的事。

FSD v12 起改用 end-to-end 架構——8 顆攝影機的原始影像直接輸入，輸出方向盤與油門指令，中間沒有人工規則，全靠模型自己學。

每台 Tesla 行駛時都在 shadow mode 默默預測「如果 FSD 接管會怎麼做」，遇到預測與駕駛行為不符的片段就自動上傳訓練。

### Waymo：多感測器派

Waymo 走另一條路：LiDAR + radar + 精密地圖，多重機制確保安全。

2025 年，Waymo 達成 1 億英里全自動駕駛里程，每週 25 萬次付費乘車，是目前**唯一真正商業化的無人駕駛服務**。

## 現況數據

資料來源：[Obi report](https://rideobi.com/tesla/)（分析 2025/11 - 2026/01 共 94,348 筆乘車紀錄）、Morgan Stanley 估算。

從 2025 年底的數據看：

| | Tesla Robotaxi | Waymo |
|　--　|　--　|　--　|
| 每公里車費 | $1.99 | $5.72 |
| 車輛硬體成本 | ~$30,000（Cybercab） | ~$200,000 |
| 等車時間 | 15 分鐘 | 5.7 分鐘 |
| 無人駕駛狀態 | 仍有安全駕駛員 | 已完全無人 |
| 鳳凰城獲利 | 否 | 是 |

## 商業規模

資料來源：[Contrary Research](https://research.contrary.com/report/tesla-waymo-and-the-great-sensor-debate)、[notateslaapp.com](https://www.notateslaapp.com/news/3108/teslas-fsd-shadow-mode-what-it-is-and-how-it-improves-fsd)。

這個差距的根源在**商業模式決定的資料規模**。

Waymo 賣服務，不賣車。他們的車是改裝的 Jaguar I-Pace，加上一堆感測器，一台 $200,000。

要擴大訓練資料，就得多派車上路，成本線性增長。

截至 2025 年初，Waymo 史上累計行駛里程約 **7,100 萬英里**。

Tesla 賣車給消費者。全球幾百萬台 Tesla 都在 shadow mode 默默記錄「如果 FSD 接管會怎麼做」。

每賣出一台車，消費者不只付了錢，還順便幫 Tesla 訓練模型。

到 2025 年底，Tesla FSD 累計行駛里程已超過 **85 億英里**，整體車隊的 shadow mode 資料更遠不止於此。

兩者相差超過 **100 倍**。這個差距，不是砸錢買 LiDAR 能追上的。

如果視覺派能解決惡劣天氣的弱點（Tesla FSD 在大雨、濃霧下表現仍不穩定），Tesla 幾乎必勝——因為硬體成本差了 7 倍，資料規模差了超過 100 倍。

Waymo 的策略是「先把安全做到無懈可擊，再壓低成本」。

Tesla 的策略是「先把規模做到無法追趕，再解決邊界問題」。

2026 年，這場賽局還沒有結果。

## 小結

2019 年，我坐在工作坊裡，看著一台小車學會跑彎道，覺得強化學習很神奇，但離現實很遠。

2026 年，Robotaxi 已經在路上跑了。

技術本身不是護城河，商業模式才是。

Waymo 用感測器堆出安全，Tesla 用車隊規模堆出資料——這不只是兩家公司的競爭，是兩種對「如何贏得終局」的根本性分歧。

## 參考

- [[實作筆記] AWS DeepRacer](https://blog.marsen.me/2019/05/18/2019/aws_deepraceer/)
- [Tesla FSD vision-only vs LiDAR - Electrek](https://electrek.co/2025/03/23/everyones-missing-the-point-of-the-tesla-vision-vs-lidar-wile-e-coyote-video/)
- [Waymo vs Tesla robotaxi cost comparison - Obi](https://rideobi.com/tesla/)
- [2026 Is the Year of Autonomous Driving](https://www.humai.blog/2026-is-the-year-of-autonomous-driving-waymo-in-10-cities-tesla-expanding-where-you-can-already-get-a-robotaxi/)

(fin)
