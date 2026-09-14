---
title: "Day 24 : 公司內部開發AI機器人Microsoft Teams方案評估篇 - iT 邦幫忙::一起幫忙解決難題，拯救 IT 人的一天"
source: "https://ithelp.ithome.com.tw/articles/10331810"
author:
  - "[[iThome]]"
published:
created: 2026-09-14
description: "今天我們主要探討內部通訊軟體搭配API服務的解決方案評估過程，從通用的傳統聊天機器人到企業內部的聊天機器人的問題與解決策略。 需求概述 通用的傳統聊天機器人 在公司使用通訊軟體可以說是各式各樣，舉凡S..."
tags:
  - "clippings"
---
[2023 iThome 鐵人賽](https://ithelp.ithome.com.tw/2023ironman)DAY 24

0

[IT管理](https://ithelp.ithome.com.tw/2023ironman/it-management)

### 萬丈高樓平地起：解決方案架構師的探索之旅系列 第 24 篇

## Day 24: 公司內部開發AI機器人Microsoft Teams方案評估篇

- 分享至

今天我們主要探討內部通訊軟體搭配API服務的解決方案評估過程，從通用的傳統聊天機器人到企業內部的聊天機器人的問題與解決策略。

## 需求概述

## 通用的傳統聊天機器人

在公司使用通訊軟體可以說是各式各樣，舉凡Slack、Line、Teams等等的工具，然後也可以看到的是這一些工具也有衍伸聊天機器人應用在很多的場景，例如有些聊天機器人可以幫助員工查詢公司的資訊、安排行程、回答常見問題等等。

然而隨著聊天機器人的功能與應用越來越多元，也帶來了一些挑戰，像是如何讓聊天機器人更加智慧化、有趣、創新和個性化，舉例來說大部分的機器人需要使用關鍵字的方式去觸發特定的主題，所以當問題偏離了規劃的主題看到的結果就會影響使用的意願。

## 應用在公司內部的傳統機器人

接著再把使用群組拉回到公司內部，需要考量的點比起剛剛提到的問題會多了一些因素，以下是一些可能的挑戰：

- 安全性問題：保護企業敏感資料始終是首要之務。傳統的聊天機器人可能沒有足夠的安全機制來確保資訊的安全。
- 隱私問題：員工可能會擔心他們的聊天紀錄是否被公司所存取或監視。
- 技術限制：一些傳統聊天機器人可能不支援API整合或與其他系統的交互，這可能會限制它們在企業環境中的應用。
- 知識更新：隨著業務不斷變化，保持聊天機器人的知識庫更新可能會成為一大挑戰。
- 工具兼容性：聊天機器人需要與公司現有的工具和系統兼容，否則可能會造成效率低下。
- 用戶體驗：如果聊天機器人無法提供及時和準確的回應，它可能會影響員工的工作效率和滿意度。
- 教育和培訓：員工可能需要時間學習如何與聊天機器人互動，這可能會導致初期的生產力下降。

雖然聊天機器人在許多場景中都證明了其價值，但在企業內部使用時公司仍需謹慎評估其潛在的問題和挑戰，需要謹慎的選擇合適的技術和工具並確保持續的培訓和支援。

![https://ithelp.ithome.com.tw/upload/images/20231005/20141298GhYDzb4OPl.png](https://ithelp.ithome.com.tw/upload/images/20231005/20141298GhYDzb4OPl.png)

## 解決策略

這一些的問題我們可以聚焦兩個地方，分別是數據安全和隱私以及整合困難的問題，首先在安全和隱私在前幾天提到評估導入工具的時候有特別強調是很重要的事情，因為每個公司內部基本上都會有資安的團隊，所以新的系統要正式推出前也會需要有資安的項目需要透過才能正式上線。

> 關於數據面的隱私從使用的API服務而言就可以考慮Azure OpenAI

再來是整合困難的部分，這個部分如果有時程上的壓力並且內部開發的人手不多的時候，可以從現有的通訊軟體工具或者是Low Code工具(會這麼說是因為從無到有開發的系統需要花較大的功夫，包含在開發前的整個架構設計以及人員的配置)。

## 什麼是Microsoft Teams

Microsoft Teams是微軟開發的專有商務通訊平台，是Microsoft 365產品家族的一部分，它主要與類似服務Slack競爭提供了聊天和視訊會議、文件儲存和應用程式整合的功能。Microsoft Teams取代了微軟旗下的其他商業消息和協作平台，包括Skype for Business。

Microsoft Teams是基於雲的團隊協作軟體，是Microsoft 365和Office 365套件的一部分。其核心功能包括商務消息、呼叫、視訊會議和文件共享。所有規模的企業都可以使用Teams。(也有個人版可以使用)

在架構和安全方面Teams是建立在Microsoft 365群組、Microsoft Graph上，並具有與Microsoft 365其餘部分相同的企業級安全性、合規性和可管理性，另外Teams利用儲存於Azure Active Directory（Azure AD）中的身份，當創建一個團隊時，將創建一個新的Microsoft 365群組。(可以說Teams跟微軟的其他服務也是緊密整合)

> 明天會以Microsoft Teams的服務(剛好公司內部是使用這個通訊軟體工具)搭配OpenAI的技術(OpenAI這個部分可以對應Azure OpenAI)分享。

- [留言](#reply)
- [追蹤](https://ithelp.ithome.com.tw/users/login)
- [檢舉](https://ithelp.ithome.com.tw/users/login)[上一篇](https://ithelp.ithome.com.tw/articles/10331808)

[

Day 23: 公司內部開發AI機器人可行性方案 (Low Code篇)

](https://ithelp.ithome.com.tw/articles/10331808)[下一篇](https://ithelp.ithome.com.tw/articles/10331812)

[

Day 25: 公司內部開發AI機器人Microsoft Teams多種開發方案篇

](https://ithelp.ithome.com.tw/articles/10331812)

系列文

[萬丈高樓平地起：解決方案架構師的探索之旅](https://ithelp.ithome.com.tw/users/20141298/ironman/6654) 共 30 篇

目錄

1. 26
	[Day 26: 解決方案架構師導入與開發AI機器人方案的抉擇(Decision Making)](https://ithelp.ithome.com.tw/articles/10331814)
2. 27
	[Day 27: 從架構圖來看解決方案整體的輪廓](https://ithelp.ithome.com.tw/articles/10331816)
3. 28
	[Day 28: 微軟生態系的AI服務延伸思考潛在的解決方案](https://ithelp.ithome.com.tw/articles/10331823)
4. 29
	[Day 29: 番外篇: AI服務日常應用(ChatGPT、Bing Chat、Bard)](https://ithelp.ithome.com.tw/articles/10331832)
5. 30
	[Day 30: 解決方案架構師的探索之旅總結與終點前的回顧](https://ithelp.ithome.com.tw/articles/10331835)
[完整目錄](https://ithelp.ithome.com.tw/users/20141298/ironman/6654)

[![.](https://itadstatic.ithome.com.tw/B9/1789119962_6aa3cdda3af6d.png)](https://itadapi.ithome.com.tw/media/click?q=B9%7Eithome_forum%7E2609B9001)

![圖片](https://event.ithome.com.tw/itplus/upload-img/2023/8/14/ff4ada01-1df1-40bc-906f-88450c045ae6_small.jpg)

[NC3 Observatory Platform](https://itplus.ithome.com.tw/vod-page/156?utm_source=iThelp&utm_medium=articlebottom&utm_campaign=iThelparticlebottom)

臺灣資安大會 |

20 分

![圖片](https://event.ithome.com.tw/itplus/upload-img/2023/8/4/0ded2355-1d95-4f96-a361-9320428dee83_small.png)

[【Esther Derby】Leaders at All Levels（Agile summit '23）｜TITANSOFT 鈦坦科技](https://itplus.ithome.com.tw/vod-page/460?utm_source=iThelp&utm_medium=articlebottom&utm_campaign=iThelparticlebottom)

鈦坦人開講 |

46 分

![圖片](https://event.ithome.com.tw/itplus/upload-img/2023/11/21/cd3f2ea4-3f97-4607-b074-b1c15ffec3f7_small.png)

[《運用Semantic Kernel SDK 駕馭生成式AI應用的提示工程(Prompt Engineering)》](https://itplus.ithome.com.tw/vod-page/745?utm_source=iThelp&utm_medium=articlebottom&utm_campaign=iThelparticlebottom)

MWC |

41 分

![圖片](https://event.ithome.com.tw/itplus/upload-img/2025/8/6/c5b4a298-f4a5-43c8-b5e2-fa65f4eea85e_small.jpg)

[Azure DevOps Troubleshooting and best practices](https://itplus.ithome.com.tw/webinar-page/265?utm_source=iThelp&utm_medium=articlebottom&utm_campaign=iThelparticlebottom)

iThome鐵人賽 |

37 分