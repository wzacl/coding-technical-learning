URL（Uniform Resource Locator，統一資源定位符）就是：

> 網路上某個資源的完整地址。

例如：

```text
https://www.example.com/manual/index.html?id=123#chapter2
```

可以拆成：

| 部分   | 範例                   | 功能           |
| ---- | -------------------- | ------------ |
| 通訊協定 | `https://`           | 規定瀏覽器如何連線    |
| 網域名稱 | `www.example.com`    | 指定要連接的網站或伺服器 |
| 路徑   | `/manual/index.html` | 指定伺服器中的頁面或資源 |
| 查詢參數 | `?id=123`            | 傳遞額外條件或資料    |
| 錨點   | `#chapter2`          | 跳到頁面中的指定位置   |

URL 不只用於網頁，也可以指向：

```text
https://example.com/manual.pdf       # PDF 文件
https://example.com/images/logo.png  # 圖片
https://api.example.com/users/123    # API 資源
ftp://example.com/files/data.zip     # FTP 檔案
```

可以把它想成寄信地址：

- `https`：運送方式
    
- `example.com`：建築物地址
    
- `/manual/index.html`：建築物裡的房間
    
- `?id=123`：要尋找的特定資料
    

需要注意的是，網址列中看到 `https://` 只代表連線經過加密，不代表該網站一定安全或可信。