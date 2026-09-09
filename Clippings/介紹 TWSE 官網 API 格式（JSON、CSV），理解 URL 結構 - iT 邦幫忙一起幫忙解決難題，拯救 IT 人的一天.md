---
title: "介紹 TWSE 官網 API 格式（JSON、CSV），理解 URL 結構 - iT 邦幫忙::一起幫忙解決難題，拯救 IT 人的一天"
source: "https://ithelp.ithome.com.tw/articles/10381138"
author:
  - "[[iThome]]"
published:
created: 2026-09-08
description: "昨天完成了環境設定，那今天我們要來了解台灣證交所 (TWSE) 提供的公開 API。TWSE 官網雖然沒有寫「API 文件」，但實際上很多股市資料都能用 URL 直接取得，格式包含 JSON 與 CS..."
tags:
  - "clippings"
---
[2025 iThome 鐵人賽](https://ithelp.ithome.com.tw/2025ironman)DAY 2

0

[AI & Data](https://ithelp.ithome.com.tw/2025ironman/ai-and-data)

### 從網路爬蟲到資料洞察的應用系列 第 2 篇

## 介紹 TWSE 官網 API 格式（JSON、CSV），理解 URL 結構

- 分享至

昨天完成了環境設定，那今天我們要來了解台灣證交所 (TWSE) 提供的公開 API。TWSE 官網雖然沒有寫「API 文件」，但實際上很多股市資料都能用 URL 直接取得，格式包含 JSON 與 CSV。

### 為什麼要先了解 API 格式？

在寫爬蟲或資料分析之前，先搞清楚 API 回傳的格式非常重要。

- JSON：結構清楚，容易用 Python 的 requests + json() 解析。
- CSV：表格型資料，適合直接用 pandas 讀取並分析。

### TWSE API 基本範例

舉例來說，如果要抓取 每日收盤行情（stock\_day），URL 會長這樣：  
[https://www.twse.com.tw/exchangeReport/STOCK\_DAY?response=json&date=20240901&stockNo=2330](https://www.twse.com.tw/exchangeReport/STOCK_DAY?response=json&date=20240901&stockNo=2330)

拆解一下：

- exchangeReport/STOCK\_DAY → API 功能（這裡是「個股月成交資訊」）。
- response=json → 回傳格式（可改成 csv）。
- date=20240901 → 查詢日期（YYYYMMDD，這裡代表 2024 年 9 月份）。
- stockNo=2330 → 股票代號（2330 = 台積電）。

如果想要 CSV 格式，只要把 response=json 改成 response=csv 即可：  
[https://www.twse.com.tw/exchangeReport/STOCK\_DAY?response=csv&date=20240901&stockNo=2330](https://www.twse.com.tw/exchangeReport/STOCK_DAY?response=csv&date=20240901&stockNo=2330)

### JSON 與 CSV 的差異

- JSON 範例（縮短後）
```json
{
  "stat": "OK",
  "date": "20240901",
  "title": "臺灣積體電路製造股份有限公司 各日成交資訊",
  "fields": ["日期", "成交股數", "成交金額", "開盤價", "最高價", "最低價", "收盤價", "漲跌價差", "成交筆數"],
  "data": [
    ["113/09/02","35,000,000","21,000,000,000","570.0","575.0","568.0","572.0","+2.0","15,000"],
    ["113/09/03","38,000,000","22,000,000,000","575.0","582.0","572.0","580.0","+8.0","18,000"]
  ]
}
```
- CSV 範例（前幾行）
```bash
"日期","成交股數","成交金額","開盤價","最高價","最低價","收盤價","漲跌價差","成交筆數"
"113/09/02","35,000,000","21,000,000,000","570.0","575.0","568.0","572.0","+2.0","15,000"
"113/09/03","38,000,000","22,000,000,000","575.0","582.0","572.0","580.0","+8.0","18,000"
```

### Python 讀取範例

- 讀取 JSON
```python
import requests

url = "https://www.twse.com.tw/exchangeReport/STOCK_DAY?response=json&date=20240901&stockNo=2330"
res = requests.get(url)
data = res.json()

print(data["title"])  # 臺灣積體電路製造股份有限公司 各日成交資訊
print(data["fields"]) # 欄位名稱
print(data["data"][0]) # 第一筆資料
```
- 讀取 CSV
```python
import pandas as pd

url = "https://www.twse.com.tw/exchangeReport/STOCK_DAY?response=csv&date=20240901&stockNo=2330"
df = pd.read_csv(url)

print(df.head())
```

那今天就先這樣。  
![/images/emoticon/emoticon29.gif](https://ithelp.ithome.com.tw/images/emoticon/emoticon29.gif)

- [留言](#reply)
- [追蹤](https://ithelp.ithome.com.tw/users/login)
- [檢舉](https://ithelp.ithome.com.tw/users/login)[上一篇](https://ithelp.ithome.com.tw/articles/10380332)

[

前言與設定環境

](https://ithelp.ithome.com.tw/articles/10380332)[下一篇](https://ithelp.ithome.com.tw/articles/10382169)

[

利用 twstock 套件快速取得股市資料

](https://ithelp.ithome.com.tw/articles/10382169)

系列文

[從網路爬蟲到資料洞察的應用](https://ithelp.ithome.com.tw/users/20169253/ironman/8872) 共 17 篇

目錄

1. 13
	[一次抓取多個月份的資料（迴圈組 URL）](https://ithelp.ithome.com.tw/articles/10388653)
2. 14
	[將多月資料合併成一個 DataFrame](https://ithelp.ithome.com.tw/articles/10389892)
3. 15
	[計算月平均價、月成交量](https://ithelp.ithome.com.tw/articles/10390343)
4. 16
	[畫月平均收盤價折線圖](https://ithelp.ithome.com.tw/articles/10391401)
5. 17
	[畫月成交量長條圖](https://ithelp.ithome.com.tw/articles/10391931)
[完整目錄](https://ithelp.ithome.com.tw/users/20169253/ironman/8872)

[![.](https://itadstatic.ithome.com.tw/B9/1782203851_6a3a45cbd87c1.gif)](https://itadapi.ithome.com.tw/media/click?q=B9%7Eithome_forum%7E2606B9005)

![圖片](https://event.ithome.com.tw/itplus/upload-img/2023/8/16/6066d504-6705-471c-9df1-8a46c78b005e_small.png)

[優化 Kafka 的最後一哩路 - 從客戶端到伺服器端的負載平衡解決方案](https://itplus.ithome.com.tw/vod-page/572?utm_source=iThelp&utm_medium=articlebottom&utm_campaign=iThelparticlebottom)

Cloud Summit 臺灣雲端大會 |

27 分

![圖片](https://event.ithome.com.tw/itplus/upload-img/2024/8/5/3363868e-76dc-4dd0-a37c-0d8b6b91df5b_small.png)

[居安思危不如行災難備援：AWS 災備實踐勝於空談](https://itplus.ithome.com.tw/vod-page/889?utm_source=iThelp&utm_medium=articlebottom&utm_campaign=iThelparticlebottom)

Cloud Summit 臺灣雲端大會 |

27 分

![圖片](https://event.ithome.com.tw/itplus/upload-img/2025/1/21/a721f11d-942c-4326-9844-979f5fb19f65_small.png)

[衝出新手村，開發與維運的體驗進化之旅](https://itplus.ithome.com.tw/vod-page/1048?utm_source=iThelp&utm_medium=articlebottom&utm_campaign=iThelparticlebottom)

DevOpsDays |

22 分

![圖片](https://event.ithome.com.tw/itplus/upload-img/2025/11/19/75d76c2c-596a-4674-a952-2e94362c4986_small.png)

[Building an On-Prem AI Agent for Windows Server 2025: Hands-on System Maintenance](https://itplus.ithome.com.tw/vod-page/1357?utm_source=iThelp&utm_medium=articlebottom&utm_campaign=iThelparticlebottom)
