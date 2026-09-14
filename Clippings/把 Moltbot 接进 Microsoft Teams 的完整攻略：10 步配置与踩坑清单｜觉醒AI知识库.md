---
title: "把 Moltbot 接进 Microsoft Teams 的完整攻略：10 步配置与踩坑清单｜觉醒AI知识库"
source: "https://www.jxxy.net/ai/articles/_bradgroux-moltbot-microsoft-teams-setup-guide/"
author:
published: 2026-01-28
created: 2026-09-14
description: "Moltbot 接 Teams 全流程：Azure Bot 单租户、Cloudflare Tunnel 暴露 webhook、manifest.json RSC 权限、逐团队安装。10 步配置 + 6 条踩坑（botId 不匹配=静默失败、RSC 需完全重启 Teams）。"
tags:
  - "clippings"
---
自从我开始在 X 上分享 @moltbot 之旅，一个问题反复出现：「你是怎么让它跑在 Microsoft Teams 里的？」

问得好。多数人用 @telegram 或 @discord 跑 Moltbot，那些基本即插即用。@MicrosoftTeams？Teams 完全是另一种动物（Moltbot 文档）。@Microsoft 的 bot 框架一层又一层，文档假设你在构建企业 SaaS 产品，而不是接一个个人 AI 助手。

但事情是这样的：如果 Teams 是你真正工作的地方，你的 AI 助手就该住在那里。我不想仅为跟机器人说句话就切换到另一个应用。我要它就在我的团队频道里：读消息、回答问题、帮我经营生意。

所以这就是我们具体做了什么。深夜构建、大量咖啡（好吧，是 @DrPepper Zero）、加上一些试错。

## 开始前你需要什么

- 一台你控制的机器上 **安装好 Moltbot** （我在 Mac mini 上跑）
- 一个 Azure 账户（免费层就够）
- 一种把本地端口暴露到互联网的方法（我用 @Cloudflare Tunnel，下文详解）
- **约 1–2 小时** 做初始配置

## 第 1 步：在 Moltbot 安装 Teams 插件

Teams 支持不随核心安装捆绑，是独立插件。这让不需要它的人的基础安装更轻。

`clawdbot plugins install @clawdbot/msteams`

一条命令。轻松开场。

## 第 2 步：创建 Azure Bot

这里开始变「微软」。前往 Azure Portal 创建 Azure Bot 资源。

关键设置：

- **定价层：** Free（开发和个人用途）
- **应用类型：** Single Tenant
- **创建方式：** Create new Microsoft App ID

创建好后，拿到三个凭据：

- **App ID** —— 来自 Bot Configuration 页面
- **Client Secret** —— 在 App Registration 的 Certificates & Secrets 下创建
- **Tenant ID** —— 来自 App Registration Overview

记在安全的地方。三个都要用。

## 第 3 步：配置 Moltbot

把 Teams 凭据加进 Moltbot 配置：

```json
{
  "channels": {
    "msteams": {
      "enabled": true,
      "appId": "<YOUR_APP_ID>",
      "appPassword": "<YOUR_CLIENT_SECRET>",
      "tenantId": "<YOUR_TENANT_ID>",
      "webhook": {
        "port": 3978,
        "path": "/api/messages"
      }
    }
  }
}
```

**专业提示：** 也可以用环境变量（MSTEAMS\_APP\_ID、MSTEAMS\_APP\_PASSWORD、MSTEAMS\_TENANT\_ID）代替把凭据写进配置文件（我把全部凭据存在 @1Password，它也有 Moltbot 的插件/技能）。

## 第 4 步：暴露你的 webhook

这是最容易绊倒人的一步。Teams 通过 HTTPS webhook 给你的机器人发消息，它需要从互联网访问到你的机器。

我选 **Cloudflare Tunnel** ：免费、快、可作为持久服务运行（重启后仍在）。ngrok 或 Tailscale Funnel 也行。

```bash
# Install cloudflared
brew install cloudflared

# Create the tunnel
cloudflared tunnel create your-bot-name

# Configure it to route your domain to localhost:3978
# Then run it as a service so it stays up
```

结果：一个公共 URL 如 `https://yourbot.yourdomain.com` ，直通你本地的 Moltbot 实例。

## 第 5 步：设置 Messaging Endpoint

回到 Azure Portal → 你的 Bot 资源 → Configuration：

把 **Messaging Endpoint** 设为： `https://yourbot.yourdomain.com/api/messages`

这告诉 Microsoft 把 Teams 进站消息发到哪里。

## 第 6 步：启用 Teams 频道

仍在 Azure Portal → 你的 Bot → Channels → 点 **Microsoft Teams** → Configure → 接受服务条款 → Save。

这把你的机器人注册为 Teams 兼容 bot。

## 第 7 步：构建 Teams 应用包

这是最繁琐的部分。需要创建 manifest.json 告诉 Teams 你的机器人信息：

```json
{
  "manifestVersion": "1.23",
  "version": "1.0.0",
  "id": "<YOUR_APP_ID>",
  "name": { "short": "Your Bot Name" },
  "bots": [{
    "botId": "<YOUR_APP_ID>",
    "scopes": ["personal", "team", "groupChat"],
    "supportsFiles": true
  }],
  "authorization": {
    "permissions": {
      "resourceSpecific": [
        { "name": "ChannelMessage.Read.Group", "type": "Application" },
        { "name": "ChannelMessage.Send.Group", "type": "Application" },
        { "name": "ChatMessage.Read.Chat", "type": "Application" }
      ]
    }
  }
}
```

**RSC（Resource-Specific Consent）权限** 至关重要——它让你的机器人无需每次被 @ 都能读写频道消息。

还需要两个图标文件：

- outline.png（32×32）
- color.png（192×192）

三个文件一起打成 zip——这就是你的 Teams 应用包。

## 第 8 步：上传到 Teams

在 Teams → Apps → Manage your apps → **Upload a custom app** → 选你的 ZIP。

如果 sideload 失败（一些组织限制），改从 Teams Admin Center 上传。

**重要：** 上传后，把你希望它工作的每个团队都安装一遍。RSC 权限按安装逐一生效。

## 第 9 步：设置访问控制

Moltbot 可以锁定谁能跟它说话：

```json
{
  "msteams": {
    "dmPolicy": "allowlist",
    "allowFrom": ["user@yourorg.com"],
    "groupPolicy": "allowlist"
  }
}
```

我的 DM 和群组都保持 allowlist。只有批准的用户能得到响应。

## 第 10 步：重启并测试

`clawdbot gateway restart`

在 Teams 里发条消息。一切接线正确的话，你的机器人会响应。

## 用时间换来的经验

几件花了我们时间的事，让它们不再花你的：

- manifest 里 **botId 必须与 webApplicationInfo.id 完全一致** 。不匹配 = 静默失败。
- **RSC 权限在重装应用并完全退出/重启 Teams 前不生效。** 不是关闭——是完全退出。Teams 缓存非常激进。
- **Cloudflare Tunnel 优于 ngrok 的持久性。** ngrok URL 重启就变（除非付费）。Cloudflare Tunnel 给你稳定域名。
- **用 Single Tenant。** 多租户 bot 已弃用。别较劲。
- **先用 Azure Web Chat 测试** （Azure Portal 里），再去排查 Teams。它独立于 Teams 的怪癖确认 webhook 正常。
- **manifest 上传报错很晦涩。** 上传时「Something went wrong」就改走 Teams Admin Center，并看浏览器 DevTools 里的真实报错。

![图片](https://www.jxxy.net/ai/media/articles/BradGroux-2016315658720432480/media_b54_p1.jpg)

## 最终结果

我的 AI 助手（Veritas）现在住在我的 Teams 工作区里。DM、群聊、频道——全覆盖。它读频道讨论、回答问题、协助研究、管理任务，并与工作流里的其他一切集成。

配置比 Discord 或 Telegram 复杂吗？绝对。但如果 Teams 是你的日常主力，值得。让 AI 助手待在真正干活的地方，改变游戏规则。

@BradGroux（@DigitalMeld）在 Mac mini 上跑 Moltbot（原 Clawdbot），配 Claude Opus 4.5、Cloudflare Tunnel 和太多 Dr Pepper Zero。另外，本文大部分由 Clawdbot 写成……就这么回事。