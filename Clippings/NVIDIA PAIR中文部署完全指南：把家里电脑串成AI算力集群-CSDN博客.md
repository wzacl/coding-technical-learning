---
title: "NVIDIA PAIR中文部署完全指南：把家里电脑串成AI算力集群-CSDN博客"
source: "https://blog.csdn.net/deepin20100/article/details/164395034"
author:
  - "[[赛博仓鼠]]"
published: 2026-09-05
created: 2026-09-07
description: "文章浏览阅读548次，点赞5次，收藏4次。全网首份NVIDIA PAIR中文部署教程：把家里闲置电脑串成AI算力集群，Windows/macOS/Linux三平台手把手安装，支持18节点协同，数据不出内网。"
tags:
  - "clippings"
---
## NVIDIA PAIR中文部署完全指南：把家里电脑串成AI算力集群

2026年9月3日，英伟达开源了Personal AI Router（PAIR）——一款能在局域网内自动调度AI推理请求的"个人AI路由器"。它不是模型，不生成内容，但能让你的RTX 4090台式机、吃灰的MacBook Pro、室友的M3 Ultra 笔记本 协同工作：单机18分钟的AI任务，多设备协同压到8分48秒。

本文是 **全网首份中文部署教程** ，从环境准备到设备配对到效果验证，手把手讲清楚。

### 一、PAIR是什么——以及它解决了什么问题

PAIR的定位很简单： **局域网内的AI请求调度员** 。

你家可能有一台主力PC（RTX 4090）、一台旧笔记本（GTX 1660 Ti）、一台MacBook Pro（M3 Pro）。它们各自装了Ollama或LM Studio，但彼此不认识——你想跑 Qwen 35B，只能盯着主力机的显存条发愁； meanwhile那台旧笔记本的6GB显存在吃灰。

PAIR干的事：自动发现局域网内所有已装PAIR的设备，实时监控每台设备的GPU显存、CPU负载、内存余量、模型加载状态，然后把你的AI请求分发给最闲的那台设备。

**它不替代任何现有工具** ：Ollama照常加载 GGUF ，LM Studio继续拖拽模型文件，PAIR只做一件事——把请求路由到对的机器上。

**核心优势** ：

- 数据完全不出内网，连路由器都不用联网
- 最多支持18个节点同时在线
- mTLS加密通信，6位验证码配对
- 三种路由策略：最快空闲优先、显存匹配优先、低功耗优先

### 二、硬件与系统要求

#### 支持的设备

| 平台      | 最低要求          | 推荐配置                  | 特殊说明                      |
| ------- | ------------- | --------------------- | ------------------------- |
| Windows | RTX 20系及以上    | RTX 40/50系，驱动≥v555.42 | 需NVIDIA GPU               |
| macOS   | Apple M4及以上   | M4 Pro/Max/Ultra      | M1/M2需Rosetta2兼容层         |
| Linux   | Ubuntu 24.04+ | 服务器版或工作站版             | 需NVIDIA Container Toolkit |

#### 通用要求

- **内存** ：至少8GB（每台设备）
- **磁盘** ：建议20GB+（用于PAIR本身+模型缓存）
- **网络** ：所有设备必须在同一局域网（WiFi或有线均可）
- **推理引擎** ：Ollama或LM Studio（至少安装其一）

#### 不支持的情况

- AMD显卡（当前版本仅支持NVIDIA GPU的Windows/Linux设备）
- macOS Intel芯片（仅Apple Silicon M4及以上）
- 无GPU的设备（可作为纯CPU节点加入，但不建议）

### 三、Windows端安装部署

#### 步骤1：确认环境

右键"此电脑"→属性，确认：

- Windows 10/11 64位
- NVIDIA驱动版本≥v555.42（右键桌面→NVIDIA控制面板→系统信息→驱动程序版本）
- 已安装Ollama或LM Studio

驱动版本不够？去 [nvidia.cn/drivers](https://www.nvidia.cn/drivers/) 下载最新Game Ready驱动。

#### 步骤2：下载PAIR

打开PowerShell或CMD：

```powershell
# 创建安装目录
mkdir C:\AI-Tools\PAIR
cd C:\AI-Tools\PAIR

# 克隆仓库
git clone https://github.com/NVIDIA/PAIR.git
cd PAIR
powershell1234567
```

没有git？先去 [git-scm.com](https://git-scm.com/) 下载安装。

#### 步骤3：安装依赖

PAIR需要Python 3.10+和pip：

```powershell
# 检查Python版本
python --version
# 如果低于3.10，去python.org下载安装

# 创建虚拟环境（推荐）
python -m venv venv
venv\Scripts\activate

# 安装依赖
pip install -r requirements.txt
powershell12345678910
```

Windows用户特别注意：如果安装过程中出现 `Microsoft Visual C++` 相关报错，先安装 [Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/) 。

#### 步骤4：启动PAIR

```powershell
# 进入PAIR目录（已在虚拟环境中）
cd C:\AI-Tools\PAIR\PAIR

# 启动PAIR服务
python -m pair.server
powershell12345
```

首次启动会生成配对验证码，记下这个6位数字。同时终端会显示本地IP地址（如 `192.168.1.105` ）。

#### 步骤5：加入防火墙规则

Windows Defender可能会拦截PAIR的mDNS广播。如果其他设备发现不了这台机器：

```powershell
# 以管理员身份运行PowerShell
New-NetFirewallRule -DisplayName "PAIR-mDNS" -Direction Inbound -Protocol UDP -LocalPort 5353 -Action Allow
New-NetFirewallRule -DisplayName "PAIR-TCP" -Direction Inbound -Protocol TCP -LocalPort 8080-8090 -Action Allow
powershell123
```

### 四、macOS端安装部署

#### 步骤1：确认芯片

点击左上角苹果图标→关于本机：

- 如果是M4/M4 Pro/M4 Max/M4 Ultra/M3 Ultra——原生支持
- 如果是M1/M2/M3——需要安装Rosetta 2
```bash
# M1/M2/M3用户执行
softwareupdate --install-rosetta --agree-to-license
bash12
```

#### 步骤2：安装Homebrew（如果还没有）

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
bash1
```

#### 步骤3：安装依赖

```bash
# Python 3.10+
brew install python@3.10

# Git
brew install git
bash12345
```

#### 步骤4：下载并启动PAIR

```bash
# 创建目录
mkdir -p ~/AI-Tools
cd ~/AI-Tools

# 克隆仓库
git clone https://github.com/NVIDIA/PAIR.git
cd PAIR

# 创建虚拟环境
python3 -m venv venv
source venv/bin/activate

# 安装依赖
pip install -r requirements.txt

# 启动服务
python -m pair.server
bash1234567891011121314151617
```

macOS的mDNS服务（Bonjour）是原生支持的，通常不需要额外配置防火墙。

### 五、 Linux 端安装部署（Ubuntu 24.04+）

#### 步骤1：安装NVIDIA Container Toolkit

这是Linux用户最容易卡住的地方。PAIR依赖容器化环境来调度GPU资源。

```bash
# 添加NVIDIA包仓库
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

# 安装
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/$(dpkg --print-architecture) | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit

# 配置Docker使用NVIDIA运行时
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
bash12345678910111213
```

#### 步骤2：安装PAIR

```bash
# 基础依赖
sudo apt update
sudo apt install -y python3.10 python3.10-venv python3-pip git

# 克隆仓库
cd ~/
git clone https://github.com/NVIDIA/PAIR.git
cd PAIR

# 虚拟环境
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# 启动
python -m pair.server
bash12345678910111213141516
```

#### 步骤3：防火墙放行（如果开了ufw）

```bash
sudo ufw allow 5353/udp    # mDNS
sudo ufw allow 8080/tcp    # PAIR API
sudo ufw allow 8090/tcp    # PAIR Web UI
bash123
```

### 六、设备配对流程

这是PAIR最简洁的一步——也是它设计上最聪明的地方。

#### 自动发现（mDNS）

同一WiFi下，只要两台设备都运行着PAIR服务，它们会自动"看见"彼此。打开任意一台设备的PAIR Web界面（ `http://localhost:8080` ），在"Nodes"页面能看到局域网内的其他PAIR节点。

#### 手动配对（6位验证码）

如果自动发现失败（比如某些路由器屏蔽了mDNS广播）：

1. 在主设备（你想用来发请求的设备）上打开PAIR Web界面
2. 点击"Add Node"→"Manual Pairing"
3. 输入目标设备的IP地址和6位验证码
4. 点击"Pair"

#### 验证配对成功

配对完成后，Web界面的Nodes列表会显示：

- 设备名称
- 平台（Windows/macOS/Linux）
- GPU型号和显存
- 当前负载状态
- 已加载的模型列表

**绿灯 = 在线且空闲，黄灯 = 在线但忙碌，灰灯 = 离线**

### 七、Ollama和LM Studio接入配置

PAIR本身不跑模型，它需要知道你的设备上有什么 推理引擎 、加载了什么模型。

#### Ollama配置

Ollama默认监听 `127.0.0.1:11434` ，PAIR需要知道这一点：

```bash
# 编辑PAIR配置文件（在每个节点上执行）
nano ~/.pair/config.yaml
bash12
```

添加：

```yaml
engines:
  ollama:
    host: "127.0.0.1"
    port: 11434
    enabled: true
yaml12345
```

保存后重启PAIR服务：

```bash
# Windows
Ctrl+C 然后重新运行 python -m pair.server

# macOS/Linux
Ctrl+C 然后重新运行 python -m pair.server
bash12345
```

PAIR会自动扫描该Ollama实例上已下载的所有模型，并同步到其他节点可见。

#### LM Studio配置

LM Studio需要开启API服务器模式：

1. 打开LM Studio
2. 左侧栏→"Developer"→勾选"Enable API Server"
3. 默认端口是 `1234` ，保持默认即可
4. 在PAIR的 `config.yaml` 中添加：
```yaml
engines:
  lmstudio:
    host: "127.0.0.1"
    port: 1234
    enabled: true
yaml12345
```

#### 模型一致性检查

PAIR会显示每个节点上可用的模型列表。 **只有当一个请求所需的模型在目标节点上已下载时，该节点才会被纳入调度范围** 。

建议：在主要节点上统一下载常用模型（如 Qwen3.8 、Llama 3.1、DeepSeek-V3），这样调度器有更多选择。

### 八、路由策略选择与优化

PAIR提供三种路由策略，在Web界面的"Settings"→"Routing Strategy"中切换：

#### 1\. 最快空闲优先（默认）

把请求发给当前响应最快的空闲节点。适合：多用户同时提问、追求低延迟的场景。

#### 2\. 显存匹配优先

优先选择显存刚好够用的节点（不选显存远大于需求的节点，避免大材小用）。适合：显存资源紧张、需要精细调度的情况。

#### 3\. 低功耗优先

优先调度到笔记本等低功耗设备，台式机作为后备。适合：在意电费、或者主力机在做其他工作（如渲染、游戏）时。

#### 实际建议

- **日常聊天/写作** ：最快空闲优先，低延迟最重要
- **批量文档处理** ：显存匹配优先，让每台设备都跑满
- **夜间挂机任务** ：低功耗优先，让笔记本扛活，台式机休息

### 九、实际效果验证

英伟达官方给出的基准数据：

| 任务 | 单机（RTX Spark） | 3设备协同 | 提速 |
| --- | --- | --- | --- |
| Hermes智能体+Qwen 3.6 35B | 18分钟 | 8分48秒 | **51%** |
| 多文档并行审阅（5个子智能体） | 15分钟 | 6分20秒 | **58%** |

我自己的测试环境（Windows 11 + RTX 4070 + macOS M3 Pro）：

```bash
# 在"主设备"（发请求的设备）上执行
# PAIR会自动路由到合适的节点

curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3.8:14b",
    "messages": [{"role": "user", "content": "写一段Python快速排序"}]
  }'
bash123456789
```

PAIR的响应头里会包含 `X-Pair-Node` 字段，告诉你这次请求被发到了哪台设备上——方便你验证调度是否生效。

### 十、常见问题与排障

#### Q1：设备互相发现不了？

**排查清单** ：

1. 确认所有设备在同一WiFi/网段（192.168.1.x这种）
2. 检查防火墙是否放行UDP 5353（mDNS）
3. 某些家用路由器会屏蔽mDNS，尝试手动IP配对
4. Windows用户检查"专用网络"是否选成了"公用网络"（控制面板→网络和共享中心）

#### Q2：模型明明下载了但PAIR显示"模型不可用"？

PAIR只认它启动时扫描到的模型。Ollama新下载模型后，需要重启PAIR服务让它重新扫描。

#### Q3：请求被发到错误的设备上？

检查该设备的 `config.yaml` 中 `engines` 配置是否正确。LM Studio用户特别容易忘记开API Server。

#### Q4：macOS M1/M2报错"架构不匹配"？

确认Rosetta 2已安装： `softwareupdate --install-rosetta --agree-to-license`

#### Q5：Linux报错"nvidia-container-toolkit not found"？

按步骤五重新安装NVIDIA Container Toolkit，特别注意 `sudo nvidia-ctk runtime configure --runtime=docker` 这一步。

#### Q6：最多能加多少台设备？

官方标称18个节点。但实际测试中，家庭局域网环境下6-8台设备是甜点区——再多mDNS广播会有性能衰减。

### 十一、进阶玩法

#### 场景1：主力机渲染时，AI请求自动转给副机

设置路由策略为"低功耗优先"，主力机（高功耗）的优先级自动降低，AI请求流向闲置的副机或笔记本。

#### 场景2：不同设备跑不同模型

- 台式机（24GB显存）：跑Qwen 3.8 72B、DeepSeek-V3
- 笔记本（8GB显存）：跑Qwen 3.8 7B、Llama 3.1 8B
- MacBook（统一内存36GB）：跑MLX优化版模型

PAIR会根据请求中的模型名称自动路由到有该模型的设备。

#### 场景3：与Home Assistant联动

已有开发者预告将发布"PAIR for Home Assistant"插件——让AI调度与智能家居场景联动。比如：家里没人时，所有设备全力跑AI；有人回家打开游戏时，自动降低PAIR优先级。

---

### 网盘资源区

以下网盘包含PAIR的安装包、预配置模板和本文涉及的Ollama/LM Studio整合包：

**夸克网盘1 — PAIR安装包+预配置文件**  
链接：https://pan.quark.cn/s/a7d5cb0d9fbe

**夸克网盘2 — Ollama整合包+常用模型资源**  
链接：https://pan.quark.cn/s/a8a75ed5ef5a

---

*本文数据截至2026年9月5日。PAIR目前为测试版，GitHub仓库已开放（github.com/NVIDIA/PAIR），采用Apache 2.0许可证。Windows和macOS的图形界面版本已经可用，Linux用户目前主要使用终端界面。PAIR不是魔法——它不能把GTX 1660变成RTX 4090，但它能让你的1660在4090忙的时候顶上来，不让任何一瓦算力浪费。*