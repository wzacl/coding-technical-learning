# 台灣投信 ETF 折溢價、成分股與權重資料來源

查核日期：**2026-09-23（台灣時間）**。

範圍以[證交所 ETF e添富商品頁](https://www.twse.com.tw/zh/ETFortune/products)當日發行機構篩選器列出的 **23 家**為主，並交叉檢查[櫃買中心 ETF 篩選器](https://info.tpex.org.tw/ETF/zh/filter.html)。包含股票、債券、期貨與主動式 ETF；不同類型不一定有「成分股」。本表是當日查核範圍，不代表所有投信公司都有發行 ETF。

## 1. 查核結論

- **有可直接讀取的官網資料端點。** 本次實測國泰、復華、永豐、野村共 4 家，無須帳號或 API Key 即可取得非空的即時預估淨值 JSON，並包含市價與折溢價欄位。它們是官網使用的端點；本次未確認這些端點有對外提供的開發者契約、版本承諾或服務保證。詳見第 3 節。
- **持股與權重也有可取得的資料。** 復華的 `assets` 端點實測包含證券代碼、名稱、股數、市值與權重；永豐、群益、野村、大華銀等官方網頁也查到持股／權重表。是否揭露全數、日期與資產類別仍須逐檔確認。
- **證交所有公開 JSON 介接格式。** 這是要求投信以固定網址提供資料的格式說明；文件本身未列出各家可呼叫網址，也不包含完整持股清單。不能因此推定所有投信端點皆可供外部匿名使用。
- **不能把「未找到 API」寫成「沒有 API」。** 其餘機構以下列官方網頁作為查詢入口；表中明確區分已成功呼叫、需 token、以及尚未驗證 JSON 的情況。

### API 狀態如何閱讀

| 標記                | 本文的判定標準                                          |
| ----------------- | ------------------------------------------------ |
| **B：官網 JSON 已測通** | 依官網前端線索找到端點，使用文中方法取得有效、非空資料；未確認它是對外承諾維護的開發者 API。 |
| **T：有 token 限制**  | 已辨識官網端點，但未帶有效網頁 token 的測試失敗，不能當作可直接匿名使用的 API。    |
| **C：官方網頁**        | 官方查詢入口已找到；本次未驗證可匿名取得所需資料的 JSON API。不是斷言不存在 API。  |

另有具公開文件的交易所 OpenAPI，與跨投信介接規格，列於第 4 節，避免與投信官網內部端點混為一談。

## 2. 各家投信官方入口

「持股／PCF」欄提供官方查詢位置，**PCF（申購買回清單）不一律等於基金完整投資組合**。連到單一 ETF 的網址是可核對的範例，其他基金須在官網切換。

| 投信     | 折溢價／即時預估淨值官方入口                                                             | 持股、權重／PCF 官方入口                                                                                                                                                                                                                                                                                             | API 查核結果與資料限制                                                                                      |
| ------ | -------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| 元大     | [即時預估淨值](https://www.yuantaetfs.com/tradeInfo/INav/Asia_ETF)               | [0050 申購買回清單](https://www.yuantaetfs.com/tradeInfo/pcf/0050)                                                                                                                                                                                                                                               | **C**。預估淨值頁需切換市場／類型；0050 樣本可讀到申贖籃子與股數，未據此確認完整持股權重 API。                                             |
| 國泰     | [即時預估淨值](https://www.cathaysite.com.tw/ETF/estimate)                       | [申購買回清單](https://www.cathaysite.com.tw/ETF/purchase)                                                                                                                                                                                                                                                       | **B**。預估淨值 JSON、申贖資料、籃子證券 JSON 及 Excel 回應已測通；籃子的 `basketShares` 不是基金持股權重。                          |
| 富邦     | [即時預估淨值](https://websys.fsit.com.tw/FubonETF/Trade/Estimate.aspx)          | [申購買回清單](https://websys.fsit.com.tw/FubonETF/Trade/Pcf.aspx)                                                                                                                                                                                                                                               | **C**。另有[歷史折溢價](https://websys.fsit.com.tw/FubonETF/Trade/PremiumDiscount.aspx)；PCF 欄位與下載內容須依基金確認。 |
| 群益     | [即時預估淨值](https://www.capitalfund.com.tw/etf/transaction/networth)          | [申購買回清單](https://www.capitalfund.com.tw/etf/transaction/buyback)、[00997A 範例](https://www.capitalfund.com.tw/etf/product/detail/502/buyback)                                                                                                                                                                | **C**。00997A 範例網頁實際包含股票代碼、股數與持股權重欄位。                                                               |
| 復華     | [即時估計淨值](https://www.fhtrust.com.tw/ETF/etf_data_value)                    | [00929 基金專頁／投資組合](https://www.fhtrust.com.tw/ETF/etf_detail/ETF21)                                                                                                                                                                                                                                         | **B**。即時預估淨值與實際投資組合 JSON 都已測通；投資組合包含權重。                                                            |
| 中國信託   | [即時預估淨值](https://www.ctbcinvestments.com.tw/Etf/INav)                      | [申購買回清單](https://www.ctbcinvestments.com.tw/Etf/Buyback)                                                                                                                                                                                                                                                   | **T**。官網 API 未帶有效 token 時回傳業務錯誤；不能只看 HTTP 200 就判定成功。                                               |
| 凱基     | [即時預估淨值](https://www.kgifund.com.tw/Fund/RealtimeNav)                      | [申購買回清單](https://www.kgifund.com.tw/Fund/RedemptionList)                                                                                                                                                                                                                                                   | **C**。官方入口已確認；未驗證完整持股權重 JSON。                                                                      |
| 永豐     | [ETF 專區／即時預估淨值](https://sitc.sinopac.com/SinopacEtfs/)                     | [006204 申購買回／基金資產](https://sitc.sinopac.com/SinopacEtfs/Etfs/SinglePcf/006204)                                                                                                                                                                                                                             | **B**。預估淨值 JSON 已測通；範例 HTML 包含股票、股數與「佔基金淨資產之權重(%)」，持股 JSON 未驗證。                                    |
| 台新     | [即時預估淨值](https://www.tsit.com.tw/ETF/Home/Estimate)                        | [申購買回清單](https://www.tsit.com.tw/ETF/Home/Pcf)                                                                                                                                                                                                                                                             | **C**。可能先顯示風險說明；另有[折溢價查詢](https://www.tsit.com.tw/ETF/Home/DPSearch)。                              |
| 統一     | [即時預估淨值](https://www.ezmoney.com.tw/ETF/Transaction/Estimate)              | [申購買回清單](https://www.ezmoney.com.tw/ETF/Transaction/PCF)                                                                                                                                                                                                                                                   | **C**。即時入口本次轉往風險揭露頁；資料表需依頁面流程與基金選擇載入，未完整核對。                                                        |
| 兆豐     | [即時預估淨值](https://www.megafunds.com.tw/MEGA/etf/trade_estimate.aspx)        | [申購買回清單](https://www.megafunds.com.tw/MEGA/etf/trade_pcf.aspx)                                                                                                                                                                                                                                             | **C**。預估淨值頁另提供當日預估淨值下載功能；尚未驗證固定 JSON 介面。                                                           |
| 第一金    | [ETF 系列入口](https://www.fsitc.com.tw/ETFList.aspx)                          | [同一 ETF 系列入口](https://www.fsitc.com.tw/ETFList.aspx)                                                                                                                                                                                                                                                       | **C，查核有限**。本次讀到 ETF 風險同意頁；同意後的即時表格、逐檔持股／權重與下載內容尚未完整核對。                                             |
| 野村     | [即時預估淨值](https://www.nomurafunds.com.tw/ETFWEB/inav)                       | [申購買回清單](https://www.nomurafunds.com.tw/ETFWEB/pcf)                                                                                                                                                                                                                                                        | **B**。預估淨值 POST JSON 已測通；00980A 範例網頁另有股票／期貨與權重，並有「查看更多」及持股比重入口，不能只取預設前幾筆。                          |
| 玉山     | [即時預估淨值](https://www.esunam.com/ETF/trad-info)                             | [持股比重](https://www.esunam.com/ETF/stock-percent)、[申購買回清單](https://www.esunam.com/ETF/etf-pcf)                                                                                                                                                                                                              | **C**。預估淨值表與官網持股比重功能路徑已確認；持股動態資料與 JSON 尚未完整核對。                                                     |
| 大華銀    | [即時預估淨值](https://www.uobam.com.tw/fund/etf/estimate)                       | [申購買回清單／基金資產](https://www.uobam.com.tw/fund/etf/pcf)                                                                                                                                                                                                                                                       | **C**。本次 009829 樣本包含股票代號、股數、金額及佔基金淨資產權重，並分開標示 PCF 日期與基金資產日期。                                       |
| 富蘭克林華美 | [即時預估淨值](https://www.ftft.com.tw/etf/estimate/)                            | [交易資訊](https://www.ftft.com.tw/etf/transcation/) → 申購買回清單                                                                                                                                                                                                                                                  | **C**。預估淨值表已讀取；交易資訊需切換對應頁籤，未驗證持股 JSON。網址中的 `transcation` 為官網原有拼字。                                  |
| 街口     | [即時預估淨值](https://jkoam.com/etf/predict)                                    | [00693U PCF](https://jkoam.com/etf/etf-pcf/34)、[基金組合入口](https://jkoam.com/etf/etf-constituent/34)                                                                                                                                                                                                          | **C**。預估淨值表已確認；後兩者為官網動態頁面，未完整驗證表格內容。商品 ETF 應看期貨部位與曝險，不宜一概稱為股票持股。                                   |
| 聯邦     | [即時預估淨值入口](https://www.usitc.com.tw/CustCenter/InstantNav_Preview)         | [申購買回清單／基金成分股](https://www.usitc.com.tw/CustCenter/BuyBackList)                                                                                                                                                                                                                                            | **C**。即時入口有風險說明；申贖頁可見基金成分股、股數與權重欄位，未驗證 JSON。                                                       |
| 華南永昌   | [即時預估淨值](https://www.hnfunds.com.tw/etf/nav-estimation)                    | [申購買回清單](https://www.hnfunds.com.tw/etf/trading-info/redemption-list)                                                                                                                                                                                                                                      | **C**。預估淨值表已讀取；申贖頁為動態功能入口，未完整驗證持股內容。                                                               |
| 安聯     | [即時預估淨值](https://etf.allianzgi.com.tw/instant-estimated)                   | [申購買回清單](https://etf.allianzgi.com.tw/list-trade)                                                                                                                                                                                                                                                          | **C**。預估淨值表已讀取；申贖頁為動態功能入口，未完整驗證持股內容。                                                               |
| 貝萊德投信  | [即時預估淨值](https://www.blackrock.com/tw/ishares/inav)                        | [官方 ETF 產品清單](https://www.blackrock.com/tw/products/products-list#/?productView=ishares&dataView=perfNav) → 選個別基金                                                                                                                                                                                          | **C**。預估淨值頁嵌入外部報價服務；未將該 iframe 視為官方公開 API。各台灣掛牌產品的完整持有明細／權重下載仍須逐檔確認。                               |
| 摩根     | [即時預估淨值](https://am.jpmorgan.com/tw/zh/asset-management/twetf/funds/inav/) | [台灣鑫收主動式 ETF 投資組合](https://am.jpmorgan.com/tw/zh/asset-management/twetf/products/jpmorgan-taiwan-taiwan-equity-high-income-active-etf-tw00000401a1#/portfolio)、[PCF](https://am.jpmorgan.com/tw/zh/asset-management/twetf/products/jpmorgan-taiwan-taiwan-equity-high-income-active-etf-tw00000401a1#/pcf) | **C**。官方預估淨值與產品頁功能連結已確認；未驗證持股 JSON。                                                                |
| 聯博     | [即時預估淨值](https://www.abfunds.com.tw/zh-tw/etfs/inav.html)                  | [00980D 申購買回清單](https://www.abfunds.com.tw/zh-tw/etfs/pcf.TW00000980D8.html)                                                                                                                                                                                                                               | **C**。官網入口已確認，資料須由動態內容載入；未完整驗證持股表與 JSON。                                                           |

命名依本次官方入口與交易所清單整理。[台新 ETF 專區](https://www.tsit.com.tw/ETF)目前可查到「台新投等債15+（原名：新光投等債15+）」00775B；使用舊機構名稱搜尋時，應再核對基金代碼與目前官方入口。

## 3. 已實測的投信資料端點

以下都是 **2026-09-23 單次查核成功的官網端點**，不代表持續可用、可任意高頻抓取、可重製散布或所有基金都具備相同欄位。網址含固定日期者是本次測試樣本，正式使用時要依官網資料日期選擇。

### 3.1 國泰：即時預估淨值、申贖籃子與 Excel

來源：[即時預估淨值頁](https://www.cathaysite.com.tw/ETF/estimate)、[申購買回清單頁](https://www.cathaysite.com.tw/ETF/purchase)及其前端呼叫。

| 方法／用途        | 已測通的完整網址                                                                                                                                | 回傳內容                                                                                                              |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| GET，即時預估淨值   | [GetRealTimeEstimateNavList](https://cwapi.cathaysite.com.tw/api/ETF/GetRealTimeEstimateNavList)                                        | JSON `result[]`：`stockCode`、`fundCode`、`estimateNav`、`lastPrice`、`diff`、`diffRate`、`closingNav`、`closingNavDate`。 |
| GET，申贖基本資料   | [GetBuySale，00878 範例](https://cwapi.cathaysite.com.tw/api/BuySale/GetBuySale?FundCode=CN)                                               | JSON `result`：基金名稱、ETF 代碼、日期、淨資產、單位數與申購基數等。                                                                       |
| GET，申贖籃子證券   | [GetStocksList，00878 範例](https://cwapi.cathaysite.com.tw/api/BuySale/GetStocksList?FundCode=CN&SearchDate=2026%2F09%2F23)               | JSON `result[]`：`prod`、`prodName`、`basketShares` 等；本次回傳 30 筆。                                                     |
| GET，申贖 Excel | [DownloadBuySaleExcel，00878 範例](https://cwapi.cathaysite.com.tw/api/BuySale/DownloadBuySaleExcel?fundCode=CN&SearchDate=2026%2F09%2F23) | 實測 HTTP 200、XLSX MIME 類型且有檔案內容；未解析工作表。                                                                            |

- `CN` 是 00878 的投信內部基金代碼，不能直接把 `00878` 填入 `FundCode`；可先由預估淨值回應取得 `stockCode` 與 `fundCode` 對照。
- `GetStocksList` 本次需提供 `SearchDate` 才有資料；只傳 `FundCode` 的測試回傳空陣列。
- `basketShares` 是申贖籃子的股數，回應沒有完整基金持股權重，不能直接當成 `weight_pct`。
- `diffRate` 是百分比數值；例如 `-0.64` 表示折價 `0.64%`，不是 `-64%`。`closingNavDate` 是正式淨值日期，不是盤中報價更新時間。

### 3.2 復華：即時預估淨值與實際投資組合

來源：[即時估計淨值頁](https://www.fhtrust.com.tw/ETF/etf_data_value)、[00929 基金專頁](https://www.fhtrust.com.tw/ETF/etf_detail/ETF21)及其前端呼叫。

| 方法／用途 | 已測通的完整網址 | 回傳內容 |
| --- | --- | --- |
| GET，即時預估淨值 | [ETFRTnav](https://www.fhtrust.com.tw/api/ETFRTnav) | JSON `result[]`：`etfCode`、`fundID`、`pnav`、`mVal`、`dp`、`dpPct`、`modifiedTime`、`prePnav`、`preDate`。 |
| GET，基金投資組合 | [assets，00929／2026-09-22 範例](https://www.fhtrust.com.tw/api/assets?fundID=ETF21&qDate=2026%2F09%2F22) | JSON `result[0]` 有基金資訊與資料日期 `dDate`；`result[0].detail[]` 有資產類別、證券代碼、名稱、股數、市值與權重。 |

投資組合明細欄位：

| 欄位 | 意義與格式 |
| --- | --- |
| `ftype` | 資產類別，例如股票；不可假設所有列都是股票。 |
| `stockid`、`stockname` | 證券代碼、證券名稱。 |
| `qshare` | 持有數量，可能是含千分位的字串。 |
| `qshareCur` | 幣別。 |
| `mvalue`、`price` | 市值、價格；轉數值前須處理字串格式。 |
| `prate_addaccint` | 權重，例如 `3.952%`，不是小數比例 `0.03952`。 |

`ETF21` 對應 00929；查詢參數為 `fundID` 與 `qDate`。本次選擇 2026-09-22 的投資組合，有非空股票明細。另測 `/api/ETFPcf` 可取得申贖資料，但該次樣本的證券陣列為空，不能拿它替代上述 `assets` 投資組合端點。

### 3.3 永豐：即時預估淨值 JSON；持股可由官方 HTML 讀取

來源：[永豐 ETF 專區](https://sitc.sinopac.com/SinopacEtfs/)及其前端呼叫。

- **本次實測 GET**：[ListEtfEstNavs](https://sitc.sinopac.com/SinopacEtfs/EtfApi/ListEtfEstNavs)。官網前端使用 POST；本文成功測試的是 GET，未把兩種方法混稱為都已測試。
- 回應本文是 JSON 陣列，雖然伺服器的 `Content-Type` 為 `text/html`；解析時需注意。
- 已核對欄位：`fund_id`、`fund_name`、`rtnav`、`price`、`price_diff`、`price_diff_ratio`、`data_time`、`pre_nav`、`pre_nav_date`。
- [006204 PCF／基金資產頁](https://sitc.sinopac.com/SinopacEtfs/Etfs/SinglePcf/006204)另有股票代碼、名稱、股數及佔基金淨資產權重，也有其他資產類別。本次確認 HTML 內容，未驗證對應持股 JSON 端點。

### 3.4 野村：即時預估淨值 POST JSON

來源：[野村即時預估淨值](https://www.nomurafunds.com.tw/ETFWEB/inav)及其前端呼叫。

- 方法：**POST**。
- 端點：[GetFundIntradayNAV](https://www.nomurafunds.com.tw/API/ETFAPI/api/Fund/GetFundIntradayNAV)。直接用瀏覽器 GET 開啟不能替代以下 POST 測試。
- 標頭：`Content-Type: application/json`。
- 本次成功使用的 body：

```json
{"Type":5,"Keyword":"","FundNo":"","FundType":0}
```

有效資料位於 `Entries[]`：`CStockNo`、`CFundShortName`、`CEstimateNav`、`CLatestMarketPrice`、`CDiff`、`CDiffPct`、`CDataDt`、`CLastDayNAVDate`。本次回傳 11 筆，包含股票、債券與主動式 ETF；不據此保證涵蓋所有產品。

`CDiffPct` 為百分比數值。回應中即使 `TotalItems` 為 0，`Entries` 仍可能有資料，本次即為此情況；應檢查實際陣列。持股部分則由[官方 PCF 頁](https://www.nomurafunds.com.tw/ETFWEB/pcf)確認有股票／期貨及權重，尚未驗證持股 API。

### 3.5 中國信託：辨識到端點，但未通過匿名呼叫測試

來源：[中國信託即時預估淨值](https://www.ctbcinvestments.com.tw/Etf/INav)及其前端呼叫。

端點為 **POST** [ETFINav](https://www.ctbcinvestments.com.tw/API/etf/ETFINav)。未帶有效 token、以 `{}` 測試時，HTTP 狀態雖為 200，回應卻是：

```json
{"ResultCode":1,"ResultMsg":"Token 無效或過期，請重新開啟網頁","Data":null}
```

因此列為 **T**，不列入已可直接匿名使用的四家。本文未驗證 token 的完整取得與更新流程；需要查詢時先使用官方網頁。

### 3.6 PowerShell 取用範例

以下使用與本次查核相同的方法及參數。日期固定於測試樣本，使用其他日期前須確認該基金有對應資料。

```powershell
# 國泰：即時預估淨值與折溢價
$cathayNav = Invoke-RestMethod -Uri 'https://cwapi.cathaysite.com.tw/api/ETF/GetRealTimeEstimateNavList'
$cathayNav.result | Select-Object stockCode, fundCode, estimateNav, lastPrice, diffRate

# 復華：00929 實際投資組合，只列股票類別
$fhPortfolio = Invoke-RestMethod -Uri 'https://www.fhtrust.com.tw/api/assets?fundID=ETF21&qDate=2026%2F09%2F22'
$fhPortfolio.result[0].detail |
    Where-Object { $_.ftype -eq '股票' } |
    Select-Object stockid, stockname, qshare, mvalue, prate_addaccint

# 永豐：回應雖標為 text/html，內容是 JSON；明確轉換
$sinopacResponse = Invoke-WebRequest -Uri 'https://sitc.sinopac.com/SinopacEtfs/EtfApi/ListEtfEstNavs'
$sinopacNav = $sinopacResponse.Content | ConvertFrom-Json
$sinopacNav | Select-Object fund_id, fund_name, rtnav, price, price_diff_ratio, data_time

# 野村：POST JSON
$nomuraNav = Invoke-RestMethod -Method Post `
    -Uri 'https://www.nomurafunds.com.tw/API/ETFAPI/api/Fund/GetFundIntradayNAV' `
    -ContentType 'application/json' `
    -Body '{"Type":5,"Keyword":"","FundNo":"","FundType":0}'
$nomuraNav.Entries |
    Select-Object CStockNo, CEstimateNav, CLatestMarketPrice, CDiffPct, CDataDt
```

## 4. 交易所的公開規格與整合入口

### 跨投信 JSON 格式：有規格，不等於有統一查詢 API

證交所公開的 [ETF 申贖資訊及即時淨值揭露專區介接格式說明 v1.5](https://dsp.twse.com.tw/public/static/downloads/tradingDepartment/ETF%20%E7%94%B3%E8%B4%96%E8%B3%87%E8%A8%8A%E5%8F%8A%E5%8D%B3%E6%99%82%E6%B7%A8%E5%80%BC%E6%8F%AD%E9%9C%B2%E5%B0%88%E5%8D%80%E4%BB%8B%E6%8E%A5%E6%A0%BC%E5%BC%8F%E8%AA%AA%E6%98%8E_20250109142554.pdf)要求各投信提供固定 HTTP／HTTPS 網址，以 UTF-8 JSON 傳送資料。

其中 `msgArray` 是資料陣列；`a`／`b` 對應 ETF 代碼／名稱，`e`／`f`／`g` 對應成交價／預估淨值／折溢價幅度，`i`／`j` 為日期／時間，另有單位數、前日淨值與參考網頁等欄位。**規格沒有完整持股與逐檔權重，也沒有提供各投信端點目錄。** 第 3 節各官網回傳格式不一定採用這組欄位名稱。

### 已有正式公開文件的 OpenAPI

| 來源 | 可取得內容 | 本次查核界線 |
| --- | --- | --- |
| [證交所 OpenAPI 文件](https://openapi.twse.com.tw/)／[Swagger JSON](https://openapi.twse.com.tw/v1/swagger.json) | [GET `/v1/opendata/t187ap47_L` 基金基本資料彙總表](https://openapi.twse.com.tw/v1/opendata/t187ap47_L)：基金代號、名稱、類型、追蹤指數、成立／上市日期等。 | 本次實測取得 JSON；適合建立基金主檔。欄位「股票及債券投資比例說明」不是每一成分證券的實際持股權重。 |
| [櫃買中心 OpenAPI 文件](https://www.tpex.org.tw/openapi/) | 可由文件檢查公開資料集。 | 本次檢查公開 Swagger，未找到可一次涵蓋所有 ETF 即時預估淨值、完整持股與權重的介面。 |
| [ETF e添富](https://www.twse.com.tw/zh/ETFortune/products) | 上市 ETF 商品與發行機構查詢。 | 作為機構／產品清單來源；不視為全市場持股 API。 |
| [櫃買中心 ETF 篩選器](https://info.tpex.org.tw/ETF/zh/filter.html) | 上櫃 ETF 與發行機構查詢。 | 與上市清單交叉核對，避免只查上市 ETF。 |

目前未確認有一個免費、具完整公開文件的官方介面，同時提供全體上市櫃 ETF 的盤中折溢價、基金完整持股與權重。這是本次查核結論，不是對所有官方或商業資料服務不存在此功能的斷言。

## 5. 取用資料時必須分清楚的口徑

| 資料 | 用途 | 不可直接推定的事情 |
| --- | --- | --- |
| 盤中預估淨值 iNAV、盤中市價 | 觀察即時預估折溢價；常見計算為 `(市價 ÷ iNAV - 1) × 100%`。 | iNAV 不等於當日結算後的正式 NAV；海外市場可能尚未開盤，報價也可能延遲。 |
| 正式 NAV、收盤價 | 比較同一日期口徑的收盤折溢價。 | 不應把今日市價與未註明日期的前日 NAV 混算後稱為即時折溢價。 |
| PCF／申購買回籃子 | 查閱一個申贖基數交付的證券、股數或現金。 | 不保證是基金所有持倉；籃子股數也不是基金淨資產占比。國泰本次樣本即須如此區分。 |
| 基金投資組合／持股權重 | 查看特定資料日的實際投資部位。 | 現金、債券、期貨或其他項目可能分表；只加股票權重不一定等於 100%。 |
| 指數成分股／指數權重 | 查看指數編製結果。 | 不等於基金當日持倉；主動式 ETF 也不應硬套追蹤指數成分股。 |

若後續建立程式，至少一併保存 ETF 代碼、投信內部基金代碼、資料種類、資料日期／時間、幣別、權重單位及來源網址。基金代碼與證券代碼應保存為字串，避免遺失 `0050` 的前導零或 `00980A` 的字母。

網站 API 請先確認回應中的成功欄位與有效資料，再解析數字；空陣列、業務錯誤、只有網頁外框的 HTTP 200 都不等於取得資料。更新頻率應依資料本身的時間與網站規範決定，避免以重複輪詢製造沒有新增資訊的請求。網站可瀏覽或單次可匿名存取，也不代表已授權大量下載或再散布。

## 6. 查核方法與尚未驗證項目

1. 以證交所／櫃買中心清單找機構，再搜尋、開啟各投信官方頁面與功能連結。
2. 對可辨識的前端資料請求，使用一般 HTTP GET／POST 做單次測試，檢查內容格式、業務回應、資料陣列及日期；第 3 節成功端點均取得實際資料。
3. 第 2 節 52 個不重複官方網頁連結均完成 GET 檢查並回應 HTTP 200；其中包含重新導向後的風險說明頁。只取得動態網頁外框、風險同意畫面或功能路徑者，已在表內註明，未將「HTTP 200」當成完整內容已驗證。PowerShell 範例另通過語法解析檢查。
4. **未驗證**每一家、每一檔 ETF 的全數歷史資料、全部資產明細、盤中長時間更新情況、流量限制、跨瀏覽器 CORS、使用授權或長期穩定性。國泰 Excel 僅核對檔案回應，未檢查工作表。
5. 網頁與未承諾對外維護的端點可能改版；正式接入前，應以代表性基金重新核對欄位、日期與資料完整性。

## 7. 原有補充連結

- [金融情報站：國泰投信 ETF 一覽](https://fininfostation.com/issuer/cathayfund)：保留原筆記的第三方參考來源；非投信官方網站，不作為本表 API 可用性或官方數據的認定依據。
