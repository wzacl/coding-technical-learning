---
title: "2025熱門Python GUI開發工具推薦：10款現代化框架特色比較"
source: "https://www.kantti.net/tw/article/1060/popular-python-gui-frameworks-2025-comparison"
author:
published: 2025-10-23
created: 2026-08-31
description: "我最近在想一件事，為什麼... 為什麼都 2024 年了，很多用 Python 寫的 GUI 工具，看起來還是那麼像 2005 年的東西？你知道的，就是那種....."
tags:
  - "clippings"
---
我最近在想一件事，為什麼... 為什麼都 2024 年了，很多用 Python 寫的 GUI 工具，看起來還是那麼像 2005 年的東西？

你知道的，就是那種... Windows XP 風格的視窗，灰色的按鈕，邊框生硬。每次看到用 Tkinter 或某些老舊函式庫做出來的介面，我腦中都會浮現大學時代的選課系統。真的不是討厭 GUI 編程，只是那種過時的感覺，讓人有點提不起勁。

所以我就花了點時間，真的去挖了一下，想找出一些「現代」一點的 Python GUI 函式庫。不是那種只能做個「Hello World」就沒下文的玩具，而是真的能用在專案上，而且看起來不會讓你覺得羞恥的工具。

這篇就是我的整理筆記。不管你是想做一個精緻的桌面應用、一個給自己用的數據儀表板，還是一個讓你方便拖拉檔案、執行腳本的小工具，下面這些東西，應該... 應該能幫你省下不少時間，也讓你的心血結晶看起來更順眼一點。

## 所以，到底有哪些選擇？

老實說，選擇還真不少。但我不想只列個清單出來，那沒什麼意思。我自己是習慣把他們分成幾類，這樣比較好思考要用哪個。

大概可以分成：重量級的全能型選手、用 Web 技術來做的、還有一些...嗯...蠻有特色的黑馬。

![傳統 GUI 與現代 GUI 的視覺感受對比](https://www.kantti.net/cdn-cgi/image/format=auto,width=800,quality=80/tw/uploads/article/1060/690b31f6e2764.jpg)

傳統 GUI 與現代 GUI 的視覺感受對比

## 重量級選手：功能完整，但需要點耐心

這類通常就是指那些功能最完整、最穩定、但也最複雜的。如果你要做的東西是個商業軟體等級的應用，那大概就是從這裡面選。

### PySide6 (Qt for Python)

**一句話結論：** 想讓你的 App 在 Windows、macOS 上看起來就像原生軟體，用它就對了。

PySide6 其實就是 Qt 這個 C++ 跨平台框架的官方 Python 版本。Qt 的歷史很久，功能非常非常強大。大名鼎鼎的 Autodesk Maya、VLC 播放器，底層都有 Qt 的影子。所以，它的穩定性和效能是不用懷疑的。

不過呢，它的學習曲線也是最陡峭的。你需要理解它的一套設計邏輯，像是「信號與槽」(signals & slots)、各種佈局管理器...等等。跟寫網頁前端的思維模式完全不同。我自己是覺得，一旦你跨過了那個坎，就會覺得它無比強大。

說到這個，Qt 的文件量雖然超級龐大，但好處是中英文資料都很多。特別是 C++ 的社群，有很多資源可以參考，很多時候稍微改一下就能在 Python 裡用。這點跟一些新興框架很不一樣，像後面會提到的 Flet，目前主要還是得看英文的官方文件或 Discord 社群，中文的討論就少很多。

```python
from PySide6.QtWidgets import QApplication, QPushButton

# 這段程式碼很短，但背後是一個完整的事件循環
app = QApplication([])
button = QPushButton("點我一下")
button.show()
app.exec()
```

## 現代魔法：用新思維做 GUI

接下來這幾個，是我自己覺得最有趣、也最能體現「現代感」的。它們通常用了一些比較新的技術或思維，開發起來會感覺...嗯，愉快很多。

### Flet

**一句話結論：** 用寫 Python 的方式，直接寫出 [Flutter](https://flutterxperts.com/flutter-vs-python-gui-development/) App，還能跑在網頁上。

這個真的很酷。Flet 讓你用 Python 來控制 Flutter 引擎。Flutter 是 Google 開發的 UI 工具包，以漂亮的介面和高效能著稱。本來要寫 Dart 語言才能用，但 Flet 把它搬到了 Python 世界。

最棒的是，你完全不用碰前端的 HTML/CSS/JS，也不用學 Dart。你就只是寫 Python，定義你的按鈕、輸入框、排版，它就會即時呈現在一個視窗裡，而且還支援熱重載（Hot Reload），改完程式碼存檔，介面就自動更新，開發體驗超好。

```
import flet as ft

def main(page: ft.Page):
    page.title = "Flet 範例"

    def btn_click(e):
        if not txt_name.value:
            txt_name.error_text = "名字不能是空的！"
            page.update()
        else:
            name = txt_name.value
            page.clean()
            page.add(ft.Text(f"哈囉, {name}!"))
    
    txt_name = ft.TextField(label="你的名字")

    page.add(
        txt_name,
        ft.ElevatedButton("送出", on_click=btn_click)
    )

ft.app(target=main)
```

你看，它的寫法非常直觀，有點像在組合積木。做出來的介面也很現代，內建就有 Material Design 風格。

### Dear PyGui

**一句話結論：** 需要高效能、硬體加速的介面，特別是做工程或遊戲工具。

Dear PyGui 是少數幾個直接利用 GPU 進行渲染的函式庫。所以它的反應速度非常快，拖動視窗、繪製圖表都很流暢，完全不卡頓。它的寫法也很特別，是一種「立即模式」(Immediate Mode) 的 API，這在遊戲開發領域比較常見。

簡單說，你不用去管理什麼物件狀態，你只要在每一幀（frame）告訴它要畫什麼就行了。這讓它在做一些需要即時更新數據的儀表板、或是科學計算視覺化的工具時，特別好用。它內建的圖表、表格、日誌功能都非常強大。

```
from dearpygui.core import *
from dearpygui.simple import *

def log_message(sender, data):
    log_info("按鈕被點擊了！")

with window("主視窗"):
    add_button("紀錄訊息", callback=log_message)
    add_input_text("輸入框")
    # 這個 Logger 視窗超實用
    add_logger()

start_dearpygui()
```
![在編輯器中編寫 Textual 應用程式的樣子](https://www.kantti.net/cdn-cgi/image/format=auto,width=800,quality=80/tw/uploads/article/1060/690b31fc2a447.jpg)

在編輯器中編寫 Textual 應用程式的樣子

## 另闢蹊徑：非傳統 GUI 方案

有時候我們不一定需要一個獨立的桌面視窗。有些工具，用其他形式呈現反而更方便。

### Textual

**一句話結論：** 讓你的終端機應用程式，變得跟 VS Code 一樣酷炫。

Textual 這個專案，我真的超愛。它讓你可以在終端機（Terminal）裡面，建立擁有佈局、色彩、滑鼠互動，甚至是動畫的 TUI (Text-based User Interface)。

如果你寫過一些命令列工具，就知道傳統的 \`print\` 和 \`input\` 能做到的很有限。但 Textual 讓你像在寫網頁一樣，用類似 CSS 的方式來排版你的終端機畫面。你可以有按鈕、輸入框、進度條...。對於那些需要在遠端伺服器上運行的管理工具、監控面板來說，這簡直是神器，因為你不需要圖形介面環境就能跑。

### Eel

**一句話結論：** 給會寫網頁前端的人，用 Python 當作桌面應用的「大腦」。

如果你本身就熟悉 HTML/CSS/JavaScript，那 Eel 可能會是你的最愛。它的概念很簡單：你用你最熟悉的網頁技術來打造使用者介面，然後用 Eel 把這個前端跟你的 Python 後端邏輯「黏」起來。

它會在本地開一個小小的網頁伺服器，然後用一個極輕量的瀏覽器視窗來顯示你的 \`index.html\`。你可以從 JavaScript 呼叫 Python 函式，也能從 Python 呼叫 JavaScript 函式。跟 Electron 比起來，它打包後的體積小非常非常多，因為它用的是系統內建的瀏覽器引擎，而不是自己包一個完整的 Chromium。

## 所以...我到底該選哪個？

嗯...這問題沒有標準答案。每次選技術都是一種取捨。所以我弄了個簡單的比較表，把我自己的感覺放進去，可能會比較好懂。

| 框架 (Framework) | 感覺如何 (Look & Feel) | 好不好上手 (Learning Curve) | 最適合做啥 (Best For...) |
| --- | --- | --- | --- |
| **PySide6 (Qt)** | 非常原生，跟作業系統融為一體。但要好看，得自己花時間用 QSS (類似 CSS) 刻。 | 高。觀念很多，要學的東西不少，得有心理準備把它當一門課來學。 | 功能超複雜、需要長期維護的專業桌面軟體。比如影片剪輯、3D 建模輔助工具之類的。 |
| **Flet** | 很現代，有 Google 的 Material Design 風格。開箱即用，不用調就蠻好看的。 | 低。寫起來就像在組合樂高，很直覺。就算沒 GUI 經驗也能快速上手。 | 想快速弄出一個漂亮的小工具、內部儀表板，或是你的個人專案。跨平台發佈也簡單。 |
| **Dear PyGui** | 科技感、工程感很重。預設主題有點像遊戲引擎的編輯器。不是走傳統軟體路線。 | 中等。API 很獨特，跟傳統 GUI 不一樣。但理解了它的邏輯後，寫起來會很快。 | 需要即時更新大量數據的儀表板、科學模擬、 [資料視覺化工具](https://www.kantti.net/tw/article/1054/probability-distribution-data-analysis-normal-distribution-application-forecast-anomaly-detection) 。任何需要 GPU 加速的場景。 |
| **CustomTkinter** | 比原本的 Tkinter 好看一百倍。有深色模式，元件也比較現代。但骨子裡還是 Tkinter。 | 非常低。如果你會 Tkinter，幾乎是無痛轉移，API 很像。 | 學校作業、教學範例，或是你被迫要用 Tkinter 的老舊專案，想幫它拉皮一下。 |
| **Textual** | 在終端機裡算是頂級酷炫了。有版面、有顏色、有互動，像是跑在文字模式裡的 App。 | 中等。需要學它那套基於 CSS Flexbox 的佈局系統。但如果你寫過網頁，會覺得很熟悉。 | 任何跑在 Linux 伺服器上的工具。像是系統監控、日誌分析、遠端管理腳本的介面。 |
| **Eel** | 完全取決於你的前端功力。你可以用 React、Vue，弄得多漂亮都可以。 | 低 (如果你懂前端)。對 Python 開發者來說，就是學幾個簡單的溝通函式而已。 | 前端開發者想用 Python 處理後端邏輯，做成一個輕量級桌面應用。 |

## 還有一些值得一提的

當然，選擇不止上面這些。像是 Kivy，它在觸控介面和行動裝置上還是很強大；還有 Toga，是 [BeeWare](https://thecyberiatech.com/blog/mobile-app/kivy-vs-beeware/) 專案的一部分，目標是真正寫一次程式碼就能在所有平台（包括手機）跑，雖然還在發展中，但概念很棒。

PyWebIO 和 Remi 也很類似，它們讓你純用 Python 寫程式，然後在瀏覽器裡生成一個 UI，你連 HTML 都不用寫。這對於快速製作內部使用的小工具非常方便。

![用現代框架打造的數據儀表板範例](https://www.kantti.net/cdn-cgi/image/format=auto,width=800,quality=80/tw/uploads/article/1060/690b31ffa416c.jpg)

用現代框架打造的數據儀表板範例

## 結語：別再讓「醜」成為藉口了

我自己覺得，很多 Python 開發者之所以不做 GUI，不是因為它難，而是因為預設的工具做出來...真的太醜了，缺乏成就感。

但現在的情況真的不一樣了。上面這些工具，或多或少都解決了這個「美觀」的問題。有些讓你用更現代的語法，有些直接幫你套上好看的樣式，有些則是讓你發揮你原本就會的網頁技能。

所以，如果你手邊有一個一直想做成圖形介面的小腳本或小工具，但始終因為覺得麻煩或難看而沒動手，或許現在是個好時機，可以再試一次了。Python 的 GUI 生態，真的比你想像的進步更多。

---

**換你說說看：**

看完這些，你有用過哪一個嗎？或者，有沒有哪個框架讓你覺得「哦？這個好像可以來試試看」？在下面留言分享一下你的想法吧！