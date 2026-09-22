---
title: "MarkItDown 教學｜安裝、PDF 轉 Markdown 與 OCR 排錯"
source: "https://klab.tw/2026/04/markitdown-tutorial/"
author:
  - "[[Kyle]]"
published: 2026-04-12
created: 2026-09-22
description: "MarkItDown 是微軟開源的文件轉換工具，能將 PDF、Word、Excel、PowerPoint 與網頁等內容整理成 Markdown，供搜尋、文件分析與 RAG 知識庫使用。本文從 Python 環境與套件安裝開始，提供 CLI、Python API 和批次轉檔範例，並區分一般文字文件、掃描圖片與複雜版面的處理方式。遇到空白輸出、缺少依賴或 OCR 未啟動時，可依排錯表檢查；需要 LLM 或 Azure 的功能則另外說明設定、資料傳送與服務費用的界線。"
tags:
  - "clippings"
---
MarkItDown 是微軟開源的文件轉換工具，能把 PDF、Office 文件與網頁等內容整理成 Markdown，方便搜尋、批次處理或建立 RAG 知識庫。選擇轉換方式時，先確認文件是否有可擷取的文字：一般文字文件可由內建轉換器處理，掃描圖片與複雜版面則可能需要 OCR 或雲端解析服務。

## MarkItDown 是什麼

**MarkItDown** 是微軟在 2024 年底開源的 Python 工具，專門把各種檔案格式轉換成 Markdown。它跟過去常用的 `textract` 類似，但最大的差異在於 MarkItDown 會盡量保留文件的結構——標題層級、表格、列表、超連結都會對應到 Markdown 語法，而不是把所有內容打成一整段純文字。

Markdown 用少量標記表達標題、清單、表格與連結，適合保存可供後續程式處理的文件結構。不過轉成 Markdown 不代表能完整保留原始版面，也不保證模型的分析結果一定改善；重要表格、數字與閱讀順序仍需要檢查。

專案原始碼、安裝方式與版本變更可從 [Microsoft MarkItDown 官方 GitHub](https://github.com/microsoft/markitdown) 查詢。本文範例分成命令列與 Python API，兩種方式都能搭配專案需要的格式依賴。

## 支援的檔案格式

MarkItDown 支援的格式相當豐富，以下是主要的類別：

| 類別 | 支援格式 |
| --- | --- |
| Office 文件 | Word (.docx)、Excel (.xlsx,.xls)、PowerPoint (.pptx) |
| PDF | .pdf（支援 Azure Document Intelligence 加強解析） |
| 圖片 | EXIF 元資料擷取、搭配 LLM 做圖片描述、OCR 辨識 |
| 音訊 | EXIF 元資料擷取、語音轉文字 |
| 網頁與標記語言 | HTML、CSV、JSON、XML |
| 壓縮檔 | ZIP（自動遞迴處理內部檔案） |
| 其他 | YouTube 影片字幕、EPub 電子書、Outlook 郵件 |

ZIP 檔案會自動展開並逐一轉換裡面的檔案，適合批次處理文件。

### 文件類型與轉換方式

同樣是 PDF，內容可能是文字、掃描影像或兩者混合。先依輸入內容選擇方式，比只看副檔名更容易找到缺漏原因。

| 輸入內容 | 起始方式 | 需要注意 |
| --- | --- | --- |
| 文字型 PDF、Office 文件 | 先用 [CLI 基本轉換](#markitdown-cli) | 核對表格、標題與閱讀順序 |
| 掃描頁面、文件內的圖片文字 | 使用 [OCR plugin](#markitdown-ocr) 與支援視覺輸入的 LLM | 需要啟用 plugin 並設定 client；只安裝套件不會自動完成 OCR |
| 多欄、複雜表格或特殊版面 | 評估 [Azure Document Intelligence](#markitdown-azure) | 需雲端資源與認證，先確認資料傳送範圍與服務計費 |

MarkItDown 本身是開源軟體；額外呼叫 LLM 或 Azure 時，依所用服務的方案計費。處理文字文件不必為了完成基本轉檔而先申請雲端 OCR。

## 安裝 MarkItDown

官方要求 **Python 3.10 以上** ，並建議使用虛擬環境。原文在 2026 年 4 月的安裝觀察中，Python 3.14 曾因依賴相容性而解析到舊版 0.0.2；這是當時環境的結果，不能直接當作現在所有 3.14 安裝都會失敗的結論。若需要沿用本文範例，可以先用 Python 3.12 建立獨立環境，並確認實際安裝版本：

```bash
# 在已啟用的虛擬環境中安裝所有格式支援
python -m pip install 'markitdown[all]'

# 或只安裝需要的格式支援，減少相依套件
python -m pip install 'markitdown[pdf,docx,pptx]'
```

可以選擇安裝的模組包含： `[all]` 、 `[pdf]` 、 `[docx]` 、 `[pptx]` 、 `[xlsx]` 、 `[xls]` 、 `[outlook]` 、 `[az-doc-intel]` 、 `[audio-transcription]` 、 `[youtube-transcription]` 。如果只是處理 PDF 和 Office 文件，不需要裝全部，可以減少安裝時間與磁碟空間。

尚未建立虛擬環境時，可先執行以下步驟；其中 `python` 應指向已安裝且符合需求的 Python 版本：

```bash
# 建立虛擬環境
python -m venv .venv
source .venv/bin/activate  # macOS / Linux
# .venv\Scripts\activate   # Windows

# 在虛擬環境中安裝
pip install 'markitdown[all]'
```

### 安裝版本與轉換結果排錯

先確認目前 shell 使用哪個 Python 環境，再檢查 MarkItDown 與其依賴。以下指令只列出版本與相依性問題，不會升級套件。

```bash
python --version
python -m pip show markitdown
python -m pip check
```

| 現象 | 檢查方向 |
| --- | --- |
| 找不到 markitdown 指令 | 啟用安裝時使用的虛擬環境，確認 pip 與 Python 屬於同一環境 |
| 提示缺少轉換依賴 | 依格式安裝對應 extras，例如 `markitdown[pdf]` |
| PDF 輸出空白或圖片文字缺漏 | 確認是否為掃描內容，再檢查 OCR plugin、 `enable_plugins=True` 、 `llm_client` 與模型設定 |
| 只安裝到舊版 | 保存 Python 版本與 pip 訊息，在獨立環境確認依賴限制，不直接覆蓋原工作環境 |

## CLI 命令列使用

安裝完成後，MarkItDown 會提供 `markitdown` 命令，可以直接在終端機使用。

### 基本轉換

```bash
# 轉換 PDF，結果輸出到終端機
markitdown report.pdf

# 轉換並儲存到檔案（使用 -o 參數）
markitdown report.pdf -o report.md

# 也可以用重導向儲存
markitdown presentation.pptx > presentation.md
```

### 透過 pipe 輸入

MarkItDown 也支援標準輸入，可以跟其他指令串接：

```bash
# 透過 pipe 輸入檔案（二進位格式需要用 -x 提示副檔名）
cat document.pdf | markitdown -x .pdf

# 搭配 curl 直接轉換網路上的檔案
curl -s https://example.com/report.xlsx | markitdown -x .xlsx > report.md
```

### 批次轉換

如果有一整個資料夾的文件需要轉換，可以用簡單的 shell 腳本：

```bash
# 把資料夾內所有 PDF 轉成 Markdown
for f in documents/*.pdf; do
  markitdown "$f" -o "${f%.pdf}.md"
done

# 轉換多種格式
for f in documents/*.{pdf,docx,pptx,xlsx}; do
  [ -f "$f" ] && markitdown "$f" -o "${f%.*}.md"
done
```

## Python API 使用

除了 CLI，MarkItDown 也提供 Python API，方便整合到自己的程式或 pipeline 中。

### 基本用法

```python
from markitdown import MarkItDown

# 建立轉換器實例
md = MarkItDown()

# 轉換檔案
result = md.convert("report.pdf")

# 取得 Markdown 文字
print(result.text_content)
```

`result.text_content` 會回傳轉換後的 Markdown 字串，可以直接拿來用。

### 批次處理多個檔案

```python
from markitdown import MarkItDown
from pathlib import Path

md = MarkItDown()

# 把資料夾內所有 PDF 轉成 Markdown
for pdf_file in Path("documents").glob("*.pdf"):
    result = md.convert(str(pdf_file))
    output_path = pdf_file.with_suffix(".md")
    output_path.write_text(result.text_content, encoding="utf-8")
    print(f"已轉換: {pdf_file.name} -> {output_path.name}")
```

### 搭配 Azure Document Intelligence

對於結構複雜的 PDF（例如多欄排版、含大量表格），可以搭配 Azure Document Intelligence 來提升解析品質：

```python
from markitdown import MarkItDown

# 使用 Azure Document Intelligence 加強 PDF 解析
md = MarkItDown(docintel_endpoint="https://your-resource.cognitiveservices.azure.com/")
result = md.convert("complex-report.pdf")
print(result.text_content)
```

需要先在 Azure 建立 Document Intelligence 資源，並設定好認證。這個功能對於掃描文件或排版特殊的 PDF 特別有用。

## 實測轉換效果

以下保留原文測試的幾種格式範例，示範轉換後的結構與限制。

### Excel → Markdown 表格

準備一個有標題列的 Excel 檔案，MarkItDown 會自動把工作表名稱轉成標題，資料轉成 Markdown 表格：

```
## 銷售數據
| 產品 | 數量 | 單價 | 總額 |
| --- | --- | --- | --- |
| 筆記型電腦 | 15 | 32000 | 480000 |
| 滑鼠 | 120 | 350 | 42000 |
| 鍵盤 | 80 | 1200 | 96000 |
| 螢幕 | 25 | 8500 | 212500 |
```

工作表名稱「銷售數據」直接變成了 `##` 標題，第一列自動被識別為表頭。如果 Excel 有多個工作表，每個都會轉成獨立的區塊。

### HTML → Markdown

HTML 的轉換效果最好，標題、表格、段落、連結都能精準對應到 Markdown：

```
# 2026 年 AI 程式助手比較

以下是目前主流的 AI 程式開發助手的比較：

| 工具 | 開發者 | 特色 |
| --- | --- | --- |
| Claude Code | Anthropic | CLI 為主，深度理解程式碼 |
| GitHub Copilot | GitHub/Microsoft | IDE 整合最廣泛 |
| Cursor | Cursor Inc. | 專屬 IDE，AI-first 設計 |

## 結論

每個工具都有自己的優勢，選擇適合自己工作流程的最重要。
```

### Word (.docx) → Markdown

Word 文件的標題層級、列表、表格都能正確轉換。實測一份包含標題、項目符號列表和表格的 Word 文件：

```
MarkItDown 測試報告

# 第一章：功能介紹

MarkItDown 支援多種檔案格式轉換，包含：

* PDF 文件
* Word 文件
* Excel 試算表

# 第二章：使用方式

可以透過 CLI 或 Python API 來使用。

|  |  |
| --- | --- |
| 方式 | 適用情境 |
| CLI | 快速轉換單個檔案 |
| Python API | 整合到程式中批次處理 |
```

可以注意到表格的表頭辨識不如 Excel 精準——Word 的表格沒有明確的「表頭」概念，所以第一列會被當成一般資料列，表頭顯示為空。整體來說結構保留得很完整。

### PowerPoint (.pptx) → Markdown

每張投影片會標註頁碼，標題變成 `#` 一級標題：

```
<!-- Slide number: 1 -->
# MarkItDown 簡介
微軟開源的文件轉換工具

<!-- Slide number: 2 -->
# 支援格式
PDF
Word
Excel
PowerPoint
HTML
圖片
音訊
```

投影片的文字內容都有保留，但排版和視覺效果當然就沒了——畢竟 Markdown 本來就不是拿來做簡報的。這個輸出格式很適合讓 LLM 理解簡報在講什麼。

### ZIP 自動展開

把多個檔案打包成 ZIP 丟給 MarkItDown，它會自動展開並逐一轉換，每個檔案用 `## File:` 標記：

```
Content from the zip file \`test-bundle.zip\`:

## File: sales.xlsx

## 銷售數據
| 產品 | 數量 | 單價 | 總額 |
| --- | --- | --- | --- |
| 筆記型電腦 | 15 | 32000 | 480000 |
...

## File: ai-tools.html

# 2026 年 AI 程式助手比較
...

## File: data.csv

| 日期 | 溫度(°C) | 濕度(%) | 天氣 |
| --- | --- | --- | --- |
...
```

這個功能非常實用。把一整包文件丟進去，一次就能全部轉完。

### 幾個要注意的地方

實測下來有幾個值得注意的行為：

- **JSON 和 XML** 基本上是原樣輸出，不會轉成 Markdown 結構。這合理，因為這些格式本身就是結構化的文字，LLM 可以直接讀。
- **圖片不搭配 LLM 時** ，如果圖片沒有 EXIF 元資料，轉換結果會是空的。要讓圖片有輸出，需要傳入 `llm_client` 讓模型描述圖片內容。
- **YouTube 字幕擷取** 不太穩定，可能會因為 YouTube 的反爬機制而失敗，回傳的是網頁 HTML 而不是字幕內容。
- **pipe 輸入二進位格式** （如 PDF、Excel）時，建議加上 `-x .xlsx` 參數提示副檔名，否則 MarkItDown 可能無法正確辨識格式。

## 進階功能：LLM 圖片描述與 OCR

MarkItDown 可以搭配 LLM 來處理圖片，讓轉換結果包含圖片的文字描述，這對建立完整的知識庫非常有幫助。

### 用 LLM 描述圖片內容

```python
from markitdown import MarkItDown
from openai import OpenAI

# 建立 OpenAI client
client = OpenAI()

# 傳入 LLM client，MarkItDown 會自動用它來描述圖片
md = MarkItDown(
    llm_client=client,
    llm_model="gpt-4o"
)

# 轉換圖片檔，會得到 LLM 產生的圖片描述
result = md.convert("architecture-diagram.png")
print(result.text_content)
```

這裡傳入的 `llm_client` 使用的是 OpenAI SDK 的介面，任何相容 OpenAI API 的服務都可以用，包括 Azure OpenAI。也可以透過 `llm_prompt` 參數自訂描述圖片時的提示詞。

### OCR 辨識文件中的圖片文字

安裝 `markitdown-ocr` plugin 後，可以對 PDF、Word、PowerPoint、Excel 文件中嵌入的圖片進行 OCR，擷取圖片裡的文字：

在同一個 Python 環境安裝 OCR plugin 與 OpenAI 相容 client，再確認 plugin 能被找到。若主程式使用 pipx，plugin 也必須裝入同一個 pipx 環境，不能只裝到另一套系統 Python。

```bash
# 在已安裝 MarkItDown 的 Python 環境執行
python -m pip install markitdown-ocr openai
markitdown --list-plugins

# 若主程式使用 pipx 安裝，改用這個安裝方式
# pipx inject markitdown markitdown-ocr openai
```

清單應出現 OCR plugin。若已安裝卻沒有辨識文字，核對 client、模型是否支援圖片，以及 API Key、額度和錯誤警告；plugin 載入成功不代表 OCR 已執行。

官方文件指出，未提供 `llm_client` 時，plugin 雖然可以載入，OCR 仍會略過並使用一般轉換器。模型必須能接收圖片；呼叫外部 LLM 時，圖片內容會送到該服務，費用與資料處理由該服務的條件決定。

```python
from markitdown import MarkItDown
from openai import OpenAI

# 啟用 plugin 並搭配 LLM vision 能力做 OCR
md = MarkItDown(
    enable_plugins=True,
    llm_client=OpenAI(),
    llm_model="gpt-4o",
)

# 轉換含有圖片的 PDF，圖片中的文字也會被辨識出來
result = md.convert("scanned-document.pdf")
print(result.text_content)
```

這個功能對於處理掃描文件、含有截圖的簡報特別實用。

## Docker 使用

如果不想在本機安裝 Python 環境，也可以透過 Docker 來使用：

```bash
# 建立 Docker image
docker build -t markitdown:latest .
# 轉換檔案（透過標準輸入輸出）
docker run --rm -i markitdown:latest < report.pdf > report.md
```

Docker 方式適合在 CI/CD pipeline 中使用，或是團隊成員不想各自設定 Python 環境的情況。

## 實際應用場景

MarkItDown 在 AI 開發流程中有幾個常見的應用場景：

### RAG 知識庫建立

建立 RAG 系統時，最費工的步驟之一就是把企業內部的文件轉成可被向量化的文字。MarkItDown 可以一次處理整個文件資料夾，轉出來的 Markdown 保留了原始結構，切 chunk 的時候可以根據標題層級來分段，效果比純文字好很多。

```python
from markitdown import MarkItDown
from pathlib import Path

md = MarkItDown()

# 把公司內部文件全部轉成 Markdown
knowledge_base = []
for doc in Path("company-docs").rglob("*.*"):
    if doc.suffix in [".pdf", ".docx", ".pptx", ".xlsx"]:
        result = md.convert(str(doc))
        knowledge_base.append({
            "source": doc.name,
            "content": result.text_content
        })

# 接下來就能把 knowledge_base 丟給 embedding model 做向量化
```

### 文件分析與摘要

想用 LLM 分析一份報告或合約？先用 MarkItDown 轉成 Markdown，再丟給模型處理：

```python
from markitdown import MarkItDown
from openai import OpenAI

md = MarkItDown()
client = OpenAI()

# 轉換文件
result = md.convert("quarterly-report.pdf")

# 用 LLM 做摘要
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "請用繁體中文摘要以下文件的重點。"},
        {"role": "user", "content": result.text_content}
    ]
)
print(response.choices[0].message.content)
```

### MCP Server 整合

MarkItDown 也有社群開發的 [MCP Server](https://github.com/mcp/microsoft/markitdown) ，可以讓 Claude Desktop、Cursor 等支援 MCP 的 AI 工具直接呼叫 MarkItDown 來轉換文件，不需要自己寫程式碼。

## Plugin 擴充系統

MarkItDown 提供了 plugin 系統，讓開發者可以擴充支援的格式或加入自訂的轉換邏輯。預設 plugin 是關閉的，需要手動啟用：

```python
from markitdown import MarkItDown

# 啟用所有已安裝的 plugin
md = MarkItDown(enable_plugins=True)
result = md.convert("special-format.xyz")
```

CLI 也可以管理 plugin：

```bash
# 列出已安裝的 plugin
markitdown --list-plugins

# 啟用 plugin 進行轉換
markitdown --use-plugins document.pdf
```

目前官方提供的 plugin 包括 `markitdown-ocr` （前面提到的 OCR 功能），如果想開發自己的 plugin，可以參考 GitHub 上的 [markitdown-sample-plugin](https://github.com/microsoft/markitdown/tree/main/packages/markitdown-sample-plugin) 範例。

## 快速使用的工具

除了直接執行 CLI，也可以透過線上介面或 Agent skill 使用轉換流程，選擇時要確認檔案交給哪個服務處理。

### 線上轉換工具

本站有一個線上工具網站 [https://ai.klab.tw](https://ai.klab.tw/) ，上面大部分功能都是直接靠瀏覽器的 JavaScript 就可以執行，幾乎不用上傳資料到伺服器就可以完成非常多事情。

- [Word 轉 Markdown / HTML 轉換工具](https://ai.klab.tw/word-to-markdown)
- [Excel 轉 Markdown / HTML 表格轉換工具](https://ai.klab.tw/excel-to-markdown/)

### Word to Markdown SKILL

若 Agent 沒有直接讀取 Word 的工具，可以把文字轉成 Markdown，並將內嵌圖片抽成獨立檔案，讓後續流程分別處理文字與圖片。

以下 Skill 整理含圖片 Word 文件的轉換步驟與已知限制，可依 Agent 支援的 skill 位置保存。實際支援路徑仍需依所用 client 版本確認。

| Agent | 位置 |
| --- | --- |
| Claude Code | .claude/skills/docx-to-markdown/SKILL.md |
| Codex | .agents/skills/docx-to-markdown/SKILL.md |
| Copilot | .github/skills/docx-to-markdown/SKILL.md |
| Cursor | .cursor/skills/docx-to-markdown/SKILL.md |
| Antigravity | .agent/skills/docx-to-markdown/SKILL.md |

```markdown
---
name: docx-to-markdown
description: >-
  將 Word (.docx) 檔案轉成 Markdown，同時抽出內嵌圖片，
  在 md 中以 ![](imageN.png) 引用。適用於含圖片的 docx 轉檔。
---

# Word to Markdown (with images)

使用 Microsoft 的 \`markitdown\` 做主要轉換，
  再從 docx（本質是 zip）抽出 \`word/media/\` 下的圖片檔，
  並依 \`document.xml\` 中的實際出現順序替換 md 內的 base64 placeholder。

## 前置確認

1. 詢問 **docx 檔案位置**（若使用者未明確指定）。
2. 輸出目錄預設與 docx 同目錄；圖片與 md 同資料夾。

## 環境需求

\`markitdown\` 官方要求 Python 3.10 以上。以下範例使用 Python 3.13；
先確認該執行檔存在，並記錄安裝後的版本與依賴檢查結果。
2026 年 4 月曾在 Python 3.14 遇到依賴解析至舊版的情形，
這是歷史環境觀察，不能當成所有新版環境的固定限制。

安裝（若尚未安裝或裝到舊版）：

\`\`\`bash
pipx install --python /opt/homebrew/bin/python3.13 'markitdown[all]'
# 已安裝時改用升級指令，不需先移除
# pipx upgrade markitdown
\`\`\`

驗證：記錄 \`markitdown --version\`，並依目前套件支援核對轉換結果。

## 轉換步驟

以 \`$DOCX\` 代表 docx 路徑，\`$DIR\` 為其所在目錄。

### 1. 執行 markitdown

\`\`\`bash
cd "$DIR"
markitdown "$DOCX" -o "${DOCX%.docx}.md"
\`\`\`

此時 md 內的圖片會是 \`![](data:image/png;base64...)\` placeholder（非真實 base64，markitdown 會截斷）。

### 2. 抽出內嵌圖片

docx 是 zip，圖片在 \`word/media/\`：

\`\`\`bash
unzip -o -j "$DOCX" "word/media/*" -d .
\`\`\`

### 3. 解析文件圖片順序並替換

同一張圖可能被引用多次，
  所以圖片檔數（\`word/media/\` 下）通常 ≤ md 中的 placeholder 數量。
  必須依 \`word/document.xml\` 中 \`embed=\` / \`r:embed=\` 的出現順序對應 \`word/_rels/document.xml.rels\` 的 rId → 檔名。

使用下列 Python 腳本（依當前目錄執行；md 檔名需先變數代入）：

\`\`\`python
#!/usr/bin/env python3
import re, pathlib, sys, zipfile

docx_path = pathlib.Path(sys.argv[1])
md_path = docx_path.with_suffix(".md")

with zipfile.ZipFile(docx_path) as z:
    rels = z.read("word/_rels/document.xml.rels").decode()
    doc = z.read("word/document.xml").decode()

rid_to_file = {
    m.group(1): m.group(2)
    for m in re.finditer(r'Id="([^"]+)"[^>]*?Target="media/([^"]+)"', rels)
}

order = []
for m in re.finditer(r'(?:embed|r:embed|link|r:link)="([^"]+)"', doc):
    if m.group(1) in rid_to_file:
        order.append(rid_to_file[m.group(1)])

text = md_path.read_text()
pat = re.compile(r"!\[\]\(data:image/[a-zA-Z]+;base64[^)]*\)")
it = iter(order)

def repl(_m):
    try:
        return f"![]({next(it)})"
    except StopIteration:
        return "![](MISSING)"

new_text, n = pat.subn(repl, text)
md_path.write_text(new_text)
print(f"replaced {n} placeholders with {len(order)} image refs")
\`\`\`

呼叫：\`python3 replace.py "$DOCX"\`

### 4. 驗證

\`\`\`bash
grep -c "MISSING" "${DOCX%.docx}.md"   # 應為 0
grep -c "base64" "${DOCX%.docx}.md"    # 應為 0
\`\`\`

## 注意事項

- **placeholder 數 ≠ 圖檔數**：docx 可重複引用同一張圖，
  document.xml 的 \`embed=\` 出現次數才是 md placeholder 數；
  腳本以此次數迭代 \`order\` 做替換，
  重複的圖會正確指向同一檔名。

- **rels 解析坑**：Relationship 的 \`Type\` 屬性含 \`/\`，正則不能用 \`[^/]*?\`，要用 \`[^>]*?\`。
- **臨時檔清理**：如果從 zip 額外解出 xml 做除錯，完成後記得刪除；圖片與 md 保留。
- **非 png 圖片**：\`word/media/\` 也可能出現 \`.jpeg\`、\`.gif\`、\`.emf\`、\`.wmf\`。
  腳本使用原始檔名所以相容。
  若是 \`.emf\`/\`.wmf\`（Word 的向量格式），
  大多數 Markdown renderer 無法顯示，
  需另外轉 png。
```

使用時先將一份 Word 檔案放在專案的子資料夾，然後使用 `/docx-to-markdown` 指令啟動 Skill，跟 Agent 說要轉換的檔案在哪裡，Agent（例如 Claude Code）就會依已授權的範圍確認 Python 環境，然後將轉換後的 MD 檔案與圖檔案放在相同的資料夾內。

## 小結

MarkItDown 解決了 AI 開發流程中一個很基本但很重要的問題：把非結構化的文件變成 LLM 能理解的格式。安裝簡單、API 直覺、支援的格式涵蓋大部分常見的辦公文件，加上 OCR 和 LLM 圖片描述等進階功能，不管是快速做個文件轉換，還是建立完整的 RAG pipeline，MarkItDown 都是值得放進工具箱的選項。

GitHub 倉庫： [microsoft/markitdown](https://github.com/microsoft/markitdown)