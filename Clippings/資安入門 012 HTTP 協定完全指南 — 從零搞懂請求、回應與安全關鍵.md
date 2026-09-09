---
title: "[資安入門] 012 HTTP 協定完全指南 — 從零搞懂請求、回應與安全關鍵"
source: "https://feifei.tw/http-protocol-complete-guide-for-cybersecurity-beginners/"
author:
  - "[[飛飛]]"
published: 2026-02-28
created: 2026-09-01
description: "專為資安初學者撰寫的 HTTP 完全指南。從協定基礎、請求與回應封包結構、HTTP Header、Method、Status Code，到 Cookie 安全、CORS、HTTPS、HTTP Request Smuggling 等資安必備知識，一篇搞懂 Web 安全的核心基礎。"
tags:
  - "clippings"
---
> **適合對象** ：資安初學者、網頁開發新手、準備 CTF / 滲透測試的自學者  
> **閱讀時間** ：約 20 分鐘

---

## 一、為什麼資安人一定要懂 HTTP？

在資訊安全的世界裡，絕大多數的攻擊都發生在「網路」上，而網路上最普遍的溝通語言就是 HTTP。無論是 XSS、CSRF、SSRF、SQL Injection 還是 API 濫用，幾乎都與 HTTP 脫不了關係。如果你不了解 HTTP，就像一個外科醫生不懂解剖學 — 連問題出在哪裡都找不到。

本文會帶你從最基礎的「協定是什麼」開始，一步步認識 HTTP 的結構、封包組成、常見方法與狀態碼，最後聚焦在資安實務中你最需要掌握的 HTTP 重點。

---

## 二、協定（Protocol）是什麼？

### 2.1 用生活比喻理解協定

「協定」就是雙方事先約定好的溝通規則。

想像你走進一間餐廳點餐：你會翻開菜單、告訴服務生你要什麼菜、服務生把訂單送到廚房、廚房做好菜再由服務生端回來給你。這整個流程之所以能順利運作，是因為你和餐廳之間有一套「大家都知道」的互動規則。

在電腦網路中， **協定就是兩台電腦之間溝通時必須遵守的規則** ，它定義了：

- 訊息的格式長什麼樣（信封怎麼寫）
- 訊息傳送的順序（誰先說話、誰先回應）
- 遇到錯誤時怎麼處理（聽不懂要怎麼反應）

### 2.2 常見的網路協定

網路世界有許多不同層級的協定，以下列出幾個你最常聽到的：

| 協定 | 全名 | 用途 |
| --- | --- | --- |
| **HTTP** | HyperText Transfer Protocol | 瀏覽網頁、API 通訊 |
| **HTTPS** | HTTP Secure（HTTP + TLS） | 加密版的 HTTP |
| **TCP** | Transmission Control Protocol | 可靠的資料傳輸（HTTP 的底層） |
| **UDP** | User Datagram Protocol | 快速但不保證送達的傳輸 |
| **DNS** | Domain Name System | 把網域名稱轉成 IP 位址 |
| **FTP** | File Transfer Protocol | 檔案傳輸 |
| **SSH** | Secure Shell | 遠端安全登入 |

在這些協定中，HTTP 是資安人員日常接觸最頻繁的一個，因為它是整個 Web 世界的基礎。

---

## 三、HTTP 是什麼？

### 3.1 定義

**HTTP（HyperText Transfer Protocol，超文本傳輸協定）** 是一種應用層協定，用來在客戶端（Client）與伺服器（Server）之間傳輸超文本（如 HTML）、圖片、影片、JSON 資料等內容。

簡單來說： **你的瀏覽器和網站伺服器之間講的話，就是 HTTP。**

### 3.2 HTTP 的基本運作模型

HTTP 採用 **請求—回應模型（Request-Response Model）** ：

```
┌──────────┐        HTTP Request         ┌──────────┐
│          │  ───────────────────────►   │          │
│  Client  │                             │  Server  │
│ (瀏覽器)  │  ◄───────────────────────   │ (網站)    │
│          │        HTTP Response        │          │
└──────────┘                             └──────────┘
```
1. **客戶端發送請求（Request）** ：「我想看首頁」
2. **伺服器處理並回應（Response）** ：「好的，這是首頁的 HTML」

### 3.3 HTTP 的重要特性

**無狀態（Stateless）** ：HTTP 本身不記得你是誰。每一次請求都是獨立的，伺服器不會自動記住你上一次做了什麼。這就是為什麼需要 Cookie、Session 等機制來維持登入狀態 — 這也是很多資安漏洞的根源。

**明文傳輸（Plaintext）** ：原始的 HTTP（不是 HTTPS）傳輸的資料是不加密的，任何人只要能攔截你的網路流量，就能看到你傳了什麼。這就是為什麼 HTTPS 如此重要。

**基於 TCP** ：HTTP 通常運行在 TCP 之上，預設使用 Port 80（HTTP）或 Port 443（HTTPS）。

### 3.4 HTTP 版本演進

| 版本 | 年份 | 重點特色 |
| --- | --- | --- |
| HTTP/0.9 | 1991 | 只能 GET，只能傳 HTML |
| HTTP/1.0 | 1996 | 加入 Header、Status Code、POST |
| HTTP/1.1 | 1997 | 持久連線、Host Header、Chunked Transfer |
| HTTP/2 | 2015 | 二進位格式、多工傳輸、Header 壓縮 |
| HTTP/3 | 2022 | 基於 QUIC（UDP），更快更安全 |

目前實務中最常見的仍是 HTTP/1.1 和 HTTP/2。

---

## 四、HTTP 封包結構：請求與回應

HTTP 的通訊由兩種封包組成： **請求封包（Request）** 和 **回應封包（Response）** 。理解它們的結構是資安分析的基本功。

### 4.1 請求封包（HTTP Request）

當你在瀏覽器輸入 `https://example.com/login` 並按下 Enter，瀏覽器會送出類似這樣的請求：

```
POST /login HTTP/1.1
                          ← 請求行（Request Line）
 example.com                             ← 以下都是 Header
 Mozilla/5.0 (Windows NT 10.0)
 application/x-www-form-urlencoded
 35
 session_id=abc123
 text/html
                                               ← 空行（分隔 Header 和 Body）
username=admin&password=P%40ssw0rd             ← Body（請求主體）
```

一個請求封包由以下三個部分組成：

**① 請求行（Request Line）**

```
[方法] [路徑] [HTTP版本]
POST   /login  HTTP/1.1
```
- **方法（Method）** ：告訴伺服器你想做什麼（GET、POST、PUT、DELETE 等）
- **路徑（Path）** ：你要存取的資源位置
- **HTTP 版本** ：使用的 HTTP 版本

**② 請求標頭（Request Headers）**

標頭是一組 `Key: Value` 的資訊，用來提供這次請求的額外資訊，例如你是誰、你接受什麼格式、你帶了什麼 Cookie 等。

**③ 請求主體（Request Body）**

Body 是請求的「內容」，通常在 POST、PUT 等方法中出現，用來傳送表單資料、JSON、檔案等。GET 請求一般沒有 Body。

### 4.2 回應封包（HTTP Response）

伺服器收到請求後，會回傳這樣的回應：

```
HTTP/1.1 200 OK                               ← 狀態行（Status Line）
 text/html; charset=utf-8         ← 以下都是 Header
 session_id=xyz789; HttpOnly; Secure
 1234
 DENY
 max-age=31536000
                                               ← 空行
<!DOCTYPE html>                                ← Body（回應主體）
<html>
  <head><title>Welcome</title></head>
  <body>登入成功！</body>
</html>
```

**① 狀態行（Status Line）**

```
[HTTP版本] [狀態碼] [狀態描述]
HTTP/1.1   200      OK
```

**② 回應標頭（Response Headers）**

伺服器回傳的額外資訊，包括內容類型、安全相關設定、Cookie 等。

**③ 回應主體（Response Body）**

伺服器實際回傳的內容，可能是 HTML、JSON、圖片、檔案等。

---

## 五、HTTP Header 深入解析

Header 是 HTTP 封包中最「資訊量密集」的部分，也是資安分析時你最常檢視的地方。

### 5.1 常見請求標頭（Request Headers）

| Header | 說明 | 資安相關性 |
| --- | --- | --- |
| `Host` | 指定目標網站的主機名稱 | 可被用於 Host Header Injection |
| `User-Agent` | 標示客戶端的瀏覽器/裝置資訊 | 可被偽造；WAF 可能依此判斷 |
| `Cookie` | 攜帶 Session、認證等資訊 | **Session Hijacking 的核心目標** |
| `Referer` | 告訴伺服器你從哪裡來 | 可被偽造；有些 CSRF 防禦依賴它 |
| `Authorization` | 攜帶認證資訊（如 Bearer Token） | Token 洩漏 = 帳號被接管 |
| `Content-Type` | 請求 Body 的資料格式 | 錯誤設定可能繞過伺服器驗證 |
| `Origin` | 發起跨域請求的來源 | CORS 檢查的關鍵欄位 |
| `X-Forwarded-For` | 紀錄原始客戶端 IP（經過代理時） | 可被偽造以繞過 IP 限制 |
| `Accept` | 客戶端能接受的回應格式 | 少數情況下影響伺服器行為 |

### 5.2 常見回應標頭（Response Headers）

| Header | 說明 | 資安相關性 |
| --- | --- | --- |
| `Set-Cookie` | 伺服器要求瀏覽器儲存 Cookie | 缺少安全旗標 = 嚴重風險 |
| `Content-Type` | 回應內容的格式 | 設定錯誤可能導致 XSS |
| `Content-Security-Policy` | 控制頁面可載入的資源來源 | **防禦 XSS 的重要機制** |
| `X-Frame-Options` | 控制頁面是否可被嵌入 iframe | 防禦 Clickjacking |
| `Strict-Transport-Security` | 強制瀏覽器使用 HTTPS | 防止 SSL Stripping 攻擊 |
| `X-Content-Type-Options` | 防止瀏覽器猜測 MIME 類型 | 設為 `nosniff` 可防禦部分攻擊 |
| `Access-Control-Allow-Origin` | CORS 設定，允許哪些來源跨域存取 | 設定過寬 = 資料洩漏風險 |
| `Server` | 揭露伺服器軟體資訊 | 資訊洩漏，幫助攻擊者選擇利用工具 |
| `X-Powered-By` | 揭露後端框架資訊 | 同上，建議移除 |
| `Cache-Control` | 控制快取行為 | 敏感頁面應設 `no-store` |

---

## 六、HTTP Method（請求方法）

HTTP Method 決定了你想對伺服器上的資源做什麼操作。

### 6.1 常用方法一覽

| Method | 用途 | 有 Body | 冪等性 |
| --- | --- | --- | --- |
| **GET** | 取得資源 | 通常無 | 是 |
| **POST** | 提交資料、建立資源 | 是 | 否 |
| **PUT** | 完整更新資源 | 是 | 是 |
| **PATCH** | 部分更新資源 | 是 | 否 |
| **DELETE** | 刪除資源 | 可有可無 | 是 |
| **HEAD** | 同 GET，但只回傳 Header | 無 | 是 |
| **OPTIONS** | 查詢伺服器支援的方法（CORS 預檢） | 無 | 是 |
| **TRACE** | 回顯伺服器收到的請求（偵錯用） | 無 | 是 |
| **CONNECT** | 建立隧道（通常用於 HTTPS 代理） | 無 | 否 |

### 6.2 資安關注重點

**GET vs POST 的安全差異** ：GET 的參數會出現在 URL 中，這代表它會被記錄在瀏覽器歷史紀錄、伺服器日誌、Referer Header 中。敏感資料（密碼、Token）絕對不該用 GET 傳遞。

**Method 覆寫攻擊** ：有些框架支援 `X-HTTP-Method-Override` Header 或 `_method` 參數，讓你用 POST 偽裝成 PUT/DELETE。攻擊者可能利用這點繞過權限檢查。

**TRACE 方法的風險** ：TRACE 方法會把伺服器收到的請求原封不動回傳，這可能被用於 **Cross-Site Tracing（XST）** 攻擊，竊取 `HttpOnly` Cookie。正式環境應停用 TRACE。

**OPTIONS 與 CORS** ：瀏覽器在跨域請求前會先送 OPTIONS 預檢請求。如果 CORS 設定不當（例如 `Access-Control-Allow-Origin: *` ），攻擊者就能從惡意網站讀取你的 API 資料。

---

## 七、HTTP Status Code（狀態碼）

狀態碼是伺服器對你的請求的「回答」，用三位數字表示。

### 7.1 五大分類

| 分類 | 範圍 | 意義 |
| --- | --- | --- |
| **1xx** | 100–199 | 資訊回應（繼續處理中） |
| **2xx** | 200–299 | 成功（請求被正確處理） |
| **3xx** | 300–399 | 重新導向（資源在別的地方） |
| **4xx** | 400–499 | 客戶端錯誤（你的請求有問題） |
| **5xx** | 500–599 | 伺服器錯誤（伺服器出了問題） |

### 7.2 資安常見狀態碼詳解

| 狀態碼 | 名稱 | 資安意義 |
| --- | --- | --- |
| **200** | OK | 請求成功。注意：即使回傳 200，Body 裡可能有錯誤訊息 |
| **301** | Moved Permanently | 永久重新導向。 **Open Redirect** 漏洞可能利用此機制 |
| **302** | Found | 暫時重新導向。同樣可能被利用於 Open Redirect |
| **304** | Not Modified | 使用快取。確認敏感資料不被快取 |
| **400** | Bad Request | 請求格式錯誤。可能洩漏伺服器解析邏輯 |
| **401** | Unauthorized | 未認證。可用於探測哪些端點需要登入 |
| **403** | Forbidden | 無權限。代表資源存在但你沒權限（資訊洩漏） |
| **404** | Not Found | 資源不存在。可用於目錄枚舉 |
| **405** | Method Not Allowed | 不允許的方法。可探測伺服器接受的 Method |
| **429** | Too Many Requests | 請求頻率過高。暴力破解防護的指標 |
| **500** | Internal Server Error | 伺服器內部錯誤。 **可能洩漏堆疊追蹤（Stack Trace）** |
| **502** | Bad Gateway | 上游伺服器錯誤。可能暗示架構資訊 |
| **503** | Service Unavailable | 服務暫時不可用。DDoS 攻擊的目標狀態 |

### 7.3 狀態碼在滲透測試中的應用

在做滲透測試或 Bug Bounty 時，狀態碼是你判斷伺服器行為的重要線索。舉例來說，403 和 404 的差異可以幫助你判斷目錄是否存在。如果 `/admin` 回傳 403 而 `/asdfgh` 回傳 404，你就知道 `/admin` 這個路徑確實存在但被限制存取。

---

## 八、資安必備的 HTTP 重點知識

### 8.1 Cookie 安全旗標

Cookie 是 HTTP 中最重要的狀態管理機制，也是攻擊者最想竊取的資訊之一。

```
session_id=abc123; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=3600
```

| 旗標 | 作用 | 沒設的風險 |
| --- | --- | --- |
| `HttpOnly` | JavaScript 無法讀取該 Cookie | XSS 可直接竊取 Session |
| `Secure` | 只在 HTTPS 下傳送 | 中間人攻擊可攔截 Cookie |
| `SameSite=Strict` | 跨站請求不帶 Cookie | CSRF 攻擊風險 |
| `SameSite=Lax` | 僅允許 top-level 導航帶 Cookie | 比 Strict 寬鬆，折衷方案 |
| `SameSite=None` | 跨站請求也帶 Cookie（需搭配 Secure） | 最寬鬆，CSRF 風險最高 |

### 8.2 HTTPS 與 TLS

HTTP 是明文傳輸，HTTPS 則是在 HTTP 外面包了一層 TLS（Transport Layer Security）加密。

**HTTPS 保護什麼** ：傳輸中的資料不被竊聽或篡改（機密性與完整性），以及驗證你連到的伺服器是不是真的（身份驗證）。

**HTTPS 不保護什麼** ：URL 路徑在建立連線後是加密的，但 DNS 查詢和 SNI（Server Name Indication）仍可能洩漏你在存取哪個網站。

**常見攻擊** ：SSL Stripping（降級為 HTTP）、過期或自簽憑證、弱加密套件等。

### 8.3 CORS（跨來源資源共享）

CORS 是瀏覽器的安全機制，控制「A 網站的 JavaScript 能不能讀取 B 網站的回應」。

**危險的設定** ：

```
*
true
```

如果同時允許任意來源和攜帶憑證，攻擊者可以從惡意網站直接讀取受害者在目標網站的資料。

### 8.4 安全相關 Header 檢查清單

部署 Web 應用程式時，以下 Header 是你應該確認有正確設定的：

```
# 1. 強制 HTTPS
 max-age=31536000; includeSubDomains; preload

# 2. 防禦 XSS — 限制可載入的資源來源
 default-src 'self'; script-src 'self'; style-src 'self'

# 3. 防禦 Clickjacking — 禁止被 iframe 嵌入
 DENY

# 4. 防止 MIME Sniffing
 nosniff

# 5. 控制 Referer 洩漏
 strict-origin-when-cross-origin

# 6. 限制瀏覽器 API 權限
 camera=(), microphone=(), geolocation=()

# 7. 移除不必要的資訊洩漏 Header
# 移除 Server、X-Powered-By、X-AspNet-Version 等
```

### 8.5 HTTP 請求走私（HTTP Request Smuggling）

HTTP Request Smuggling 是一種進階攻擊，利用前端代理伺服器（如 CDN、Load Balancer）與後端伺服器對 HTTP 封包解析方式的差異來「走私」惡意請求。

攻擊的核心在於 `Content-Length` 和 `Transfer-Encoding: chunked` 這兩個 Header 的衝突。當前端和後端對「這個請求在哪裡結束」有不同的判斷時，攻擊者就能注入一個隱藏的請求，影響其他使用者。

### 8.6 HTTP 參數污染（HPP）

當同一個參數被傳送多次時，不同的伺服器框架處理方式不同：

```
GET /search?q=apple&q=banana
```
- PHP 取最後一個（ `banana` ）
- ASP.NET 合併（ `apple,banana` ）
- Python Flask 取第一個（ `apple` ）

攻擊者可利用這種差異繞過 WAF 或參數驗證。

### 8.7 常見 HTTP 攻擊手法速覽

| 攻擊 | 利用的 HTTP 元素 | 簡述 |
| --- | --- | --- |
| **XSS** | Response Body、Content-Type | 注入惡意腳本到回應中 |
| **CSRF** | Cookie、Referer、Method | 偽造受害者的請求 |
| **SSRF** | Request URL、Host Header | 誘使伺服器發出惡意請求 |
| **SQL Injection** | Request Parameters、Body | 在參數中注入 SQL 語法 |
| **Open Redirect** | Location Header（3xx） | 利用重新導向將使用者導到惡意網站 |
| **Clickjacking** | X-Frame-Options | 在透明 iframe 中誘騙使用者點擊 |
| **Session Hijacking** | Cookie、Set-Cookie | 竊取或偽造 Session Cookie |
| **Host Header Injection** | Host Header | 利用 Host 標頭進行密碼重設投毒等攻擊 |
| **CRLF Injection** | Header（注入換行符） | 在 Header 中注入 `\r\n` 來偽造 Header 或 Response Splitting |
| **HTTP Request Smuggling** | Content-Length、Transfer-Encoding | 利用前後端解析差異走私請求 |

### 8.8 實用工具推薦

學習和分析 HTTP 時，以下工具是你的好幫手：

| 工具 | 用途 |
| --- | --- |
| **Burp Suite** | 攔截和修改 HTTP 請求的瑞士刀（滲透測試必備） |
| **OWASP ZAP** | 開源的 Web 應用程式安全掃描器 |
| **curl** | 命令列 HTTP 工具，快速測試請求 |
| **Wireshark** | 網路封包擷取與分析（可看到原始 HTTP 封包） |
| **Postman** | API 測試與開發工具 |
| **浏覽器 DevTools** | F12 開發者工具的 Network 分頁 |
| **httpie** | 比 curl 更人性化的命令列 HTTP 工具 |

---

## 九、動手練習：用 curl 觀察 HTTP

理論講完了，來動手看看真實的 HTTP 封包：

```bash
# 查看完整的請求和回應 Header
curl -v https://example.com

# 只看回應 Header
curl -I https://example.com

# 送出 POST 請求（模擬登入）
curl -X POST https://example.com/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"test123"}'

# 攜帶 Cookie
curl -b "session_id=abc123" https://example.com/dashboard

# 檢查安全 Header
curl -s -I https://example.com | grep -iE "strict-transport|content-security|x-frame|x-content-type"
```

---

## 十、總結

HTTP 是 Web 世界的基礎語言，也是資安攻防的核心戰場。作為資安新手，你需要記住以下幾件事：

1. **HTTP 是無狀態且預設明文的** — 這是許多安全問題的根源
2. **請求和回應封包的結構** （Request Line / Status Line、Headers、Body）是你分析任何 Web 漏洞的基礎
3. **Header 是情報的寶庫** — 無論是攻擊還是防禦，Header 都是你最需要關注的地方
4. **Cookie 安全旗標一定要設** — `HttpOnly` 、 `Secure` 、 `SameSite` 缺一不可
5. **狀態碼會透露資訊** — 403 vs 404 的差異可以洩漏目錄結構
6. **安全 Header 是防線** — CSP、HSTS、X-Frame-Options 等不是選配，是必備
7. **動手實作是最好的學習** — 打開 Burp Suite 或 curl，實際觀察封包

掌握了 HTTP，你就擁有了踏入資安世界最重要的基礎知識。接下來，建議你進一步學習 OWASP Top 10、練習 CTF 題目、並嘗試使用 Burp Suite 攔截和分析真實的 HTTP 流量。