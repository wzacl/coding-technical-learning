---
title: "[ Day 29 ] 上篇的http補充、418茶壺事件 - iT 邦幫忙::一起幫忙解決難題，拯救 IT 人的一天"
source: "https://ithelp.ithome.com.tw/articles/10390925"
author:
  - "[[iThome]]"
published:
created: 2026-09-01
description: "HTTP Request（請求）→ 後端 GET vs POST 項目 GET POST 資料位置 放在網址（Query String） 放在 Request Body 安全性 參數容易被看到（帳密會..."
tags:
  - "clippings"
---
[2025 iThome 鐵人賽](https://ithelp.ithome.com.tw/2025ironman)DAY 29

0

[Security](https://ithelp.ithome.com.tw/2025ironman/security)

### 從0基礎開始起飛，一起一步步踏入資安系列 第 29 篇

## \[ Day 29 \] 上篇的http補充、418茶壺事件

- 分享至

## HTTP Request（請求）→ 後端

### GET vs POST

| 項目 | GET | POST |
| --- | --- | --- |
| **資料位置** | 放在網址（Query String） | 放在 Request Body |
| **安全性** | 參數容易被看到（帳密會「裸奔」） | 資料不在網址中 |
| **用途** | 讀取資料（搜尋、查詢） | 傳送資料（登入、上傳、表單） |
| **快取** | 可被快取 | 不容易快取 |

為什麼登入表單幾乎都用 POST？

- 因為如果用 GET，帳號密碼會直接顯示在網址列，風險很大。

### 常見 Header 解說（Request Headers）

HTTP Header 是請求中最容易被忽略但其實超重要的部分。它們負責「夾帶瀏覽器與使用者資訊」讓伺服器知道怎麼處理這個請求

| Header 名稱 | 說明 |
| --- | --- |
| Host | 告訴伺服器「你想去哪個網站」 → 一個伺服器可能有多個網站，靠這個分流 |
| User-Agent | 告訴伺服器「你是用什麼設備 / 瀏覽器來的」 |
| Referer | 告訴伺服器「你是從哪一個頁面連過來的」 |
| Cookie | 附帶之前伺服器設定的 Cookie（例如登入身分） |
| Content-Type | 告訴伺服器 Body 的資料格式 |
| Content-Length | 告訴伺服器 Body 的長度（位元組） |

### User-Agent 為什麼會有「Mozilla」？

`User-Agent` 欄位裡常看到：

```swift
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 ...
```

這跟 Mozilla 軟體社群有關。早期網頁標準支援差，很多網站只對「Mozilla 系瀏覽器」開放進階功能，後來各家瀏覽器（例如 Chrome、Safari）為了相容，也都在 UA 裡放上「Mozilla/5.0」，成為歷史遺留。

- 是只有 Mozilla 能跑網站，而是這一行字成了「相容旗號」。

### Request Body

Request Body 就是 POST、PUT、PATCH 請求中「要送給伺服器的資料」，例如登入資訊、表單內容、JSON。  
常見 Header 搭配：

- `Content-Type: application/json`
- `Content-Length: 123`

範例:

```json
{
  "username": "ken",
  "password": "1234"
}
```

## HTTP Response（回應）→ 前端

瀏覽器發出 Request 後，伺服器會回一個 Response，包含狀態碼、標頭、資料。  
範例：

```yaml
HTTP/2 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 1024

<html>...</html>
```

### 起始行（Status Line）

```markdown
HTTP/2 200 OK
```
- `HTTP/2` → 協定版本
- `200` → 狀態碼
- `OK` → 狀態描述

---

### 狀態碼分類與常見代表

| 類別 | 範圍 | 意義 |
| --- | --- | --- |
| 1xx | 100–199 | 資訊（少見） |
| 2xx | 200–299 | 請求成功 |
| 3xx | 300–399 | 重新導向 |
| 4xx | 400–499 | 使用者/請求有錯 |
| 5xx | 500–599 | 伺服器出錯 |

### 常見實例

| 狀態碼 | 名稱 | 說明 |
| --- | --- | --- |
| 200 | OK | 一切正常 |
| 301 | Moved Permanently | 永久搬家（轉址） |
| 302 | Found | 臨時轉址 |
| 304 | Not Modified | 沒變，瀏覽器用快取 |
| 400 | Bad Request | 請求格式錯 |
| 401 | Unauthorized | 未登入 |
| 403 | Forbidden | 沒權限 |
| 404 | Not Found | 找不到 |
| 405 | Method Not Allowed | 方法不被允許 |
| 500 | Internal Server Error | 伺服器爆炸 |
| 502 | Bad Gateway | 中繼站壞了 |
| 503 | Service Unavailable | 忙碌或維修 |
| 504 | Gateway Timeout | 超時 |
| 418 | I’m a teapot | 茶壺搶救大作戰（愚人節彩蛋） |

- 4xx → 通常是你有問題
- 5xx → 通常是伺服器出包了

---

## 418 茶壺事件

HTTP 418 "I'm a teapot" 是 1998 年 IETF 愚人節提案 RFC 2324 的一部分。  
它的意思是：「如果你對茶壺發送泡咖啡的請求，它會拒絕並回傳 418：我只是茶壺，泡咖啡不關我事」  
如今它變成工程師圈的幽默彩蛋  

- [留言](#reply)
- [追蹤](https://ithelp.ithome.com.tw/users/login)
- [檢舉](https://ithelp.ithome.com.tw/users/login)[上一篇](https://ithelp.ithome.com.tw/articles/10389983)

[

\[ Day 28 \] 淺談一下http和http請求 ( http request )

](https://ithelp.ithome.com.tw/articles/10389983)[下一篇](https://ithelp.ithome.com.tw/articles/10391473)

[

\[ Day 30 \] Forensics簡介以及有關Forensics的CTF類型、工具整理

](https://ithelp.ithome.com.tw/articles/10391473)

系列文

[從0基礎開始起飛，一起一步步踏入資安](https://ithelp.ithome.com.tw/users/20177897/ironman/8508) 共 30 篇

目錄

1. 26
	[\[ Day 26 \] 簡單介紹cookie給你聽](https://ithelp.ithome.com.tw/articles/10388606)
2. 27
	[\[ Day27 \] 更多關於cookie的介紹!](https://ithelp.ithome.com.tw/articles/10389221)
3. 28
	[\[ Day 28 \] 淺談一下http和http請求 ( http request )](https://ithelp.ithome.com.tw/articles/10389983)
4. 29
	[\[ Day 29 \] 上篇的http補充、418茶壺事件](https://ithelp.ithome.com.tw/articles/10390925)
5. 30
	[\[ Day 30 \] Forensics簡介以及有關Forensics的CTF類型、工具整理](https://ithelp.ithome.com.tw/articles/10391473)
[完整目錄](https://ithelp.ithome.com.tw/users/20177897/ironman/8508)

[![.](https://itadstatic.ithome.com.tw/B9/1787886940_6a90fd5c11c52.gif)](https://itadapi.ithome.com.tw/media/click?q=B9%7Eithome_forum%7E2608B9005)

![圖片](https://event.ithome.com.tw/itplus/upload-img/2023/7/26/1d2006cd-ce2a-40db-875c-d25f4a837487_small.jpg)

[使用Azure Board實現Scrum](https://itplus.ithome.com.tw/vod-page/435?utm_source=iThelp&utm_medium=articlebottom&utm_campaign=iThelparticlebottom)

Agile Summit 敏捷高峰會 |

41 分

![圖片](https://event.ithome.com.tw/itplus/upload-img/2023/8/4/0ded2355-1d95-4f96-a361-9320428dee83_small.png)

[【Esther Derby】Leaders at All Levels（Agile summit '23）｜TITANSOFT 鈦坦科技](https://itplus.ithome.com.tw/vod-page/460?utm_source=iThelp&utm_medium=articlebottom&utm_campaign=iThelparticlebottom)

鈦坦人開講 |

46 分

![圖片](https://event.ithome.com.tw/itplus/upload-img/2023/8/16/194c4c74-15d3-463b-900c-a8af8eed02f2_small.png)

[數位轉型起手式，從技術質變到人才量變](https://itplus.ithome.com.tw/vod-page/570?utm_source=iThelp&utm_medium=articlebottom&utm_campaign=iThelparticlebottom)

Cloud Summit 臺灣雲端大會 |

33 分

![圖片](https://event.ithome.com.tw/itplus/upload-img/2025/11/19/e9adca61-8a78-406b-bdb6-6afd9a004285_small.png)

[照表操課，或實驗闖關？——導入 LeSS 的兩條路](https://itplus.ithome.com.tw/vod-page/1347?utm_source=iThelp&utm_medium=articlebottom&utm_campaign=iThelparticlebottom)

Hello World Dev Conference |

50 分