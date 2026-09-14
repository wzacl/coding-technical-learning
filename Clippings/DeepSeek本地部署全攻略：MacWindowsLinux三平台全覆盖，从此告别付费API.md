---
title: "DeepSeek本地部署全攻略：Mac/Windows/Linux三平台全覆盖，从此告别付费API"
source: "https://juejin.cn/post/7628889170318868532"
author:
  - "[[sayamber]]"
published: 2026-04-16
created: 2026-09-10
description: "为什么选择本地部署？ 在聊怎么部署之前，先说清楚为什么要折腾本地部署： 对比项 调用云端API 本地部署 月费用 按token计费，高频使用每月几百到上千 零成本（电费忽略不计） 数据隐私 数据上传到"
tags:
  - "clippings"
---
> 2026年了，DeepSeek 已经迭代到 V4，推理和编码能力全面超越 GPT-4o。今天手把手教你在本地跑起来——零月费、完全隐私、离线可用。

## 为什么选择本地部署？

在聊怎么部署之前，先说清楚 **为什么** 要折腾本地部署：

| 对比项 | 调用云端API | 本地部署 |
| --- | --- | --- |
| 月费用 | 按token计费，高频使用每月几百到上千 | **零成本** （电费忽略不计） |
| 数据隐私 | 数据上传到第三方服务器 | **数据完全不出本机** |
| 网络依赖 | 必须联网 | **断网也能用** |
| 响应速度 | 受网络延迟影响 | **本地推理，延迟极低** |
| 模型定制 | 只能用官方提供的能力 | **可微调、可量化、可控** |

对于企业场景，特别是 **处理敏感数据** （合同、财务、客户信息）的场景，本地部署几乎是刚需。个人开发者用来日常编码辅助，也能省下不少 API 费用。

## 硬件要求：你的电脑能跑吗？

这是大家最关心的问题。DeepSeek 提供了多个参数规模的模型版本：

| 模型版本 | 参数量 | 最低内存 | 推荐内存 | 显存需求 | 适用场景 |
| --- | --- | --- | --- | --- | --- |
| DeepSeek-R1-Distill-Qwen-1.5B | 1.5B | 4GB | 8GB | 无需GPU | 轻量测试 |
| DeepSeek-R1-Distill-Qwen-7B | 7B | 8GB | 16GB | 6GB+ | 日常问答 |
| DeepSeek-R1-Distill-Qwen-14B | 14B | 16GB | 32GB | 10GB+ | 编码/推理 |
| **DeepSeek-V4 (量化版)** | **671B MoE** | **32GB** | **64GB** | **24GB+** | **全场景** |

> **关键点** ：DeepSeek V4 采用 MoE（混合专家）架构，虽然总参数 671B，但每次推理只激活约 37B 参数，所以实际运行资源需求比想象中小很多。

**推荐配置** ：

- 💻 **Mac 用户** ：M2/M3/M4 芯片的 MacBook Pro 或 Mac Studio，16GB 统一内存起步
- 🪟 **Windows 用户** ：i7/R7 以上 CPU + 16GB 内存，有 RTX 3060+ 独显更佳
- 🐧 **Linux 用户** ：同 Windows 硬件要求，服务器部署推荐 RTX 4090

## 方案一：Ollama —— 最简单的部署方式

Ollama 是目前最流行的本地模型运行工具，2026 年 Q1 月下载量已达 5200 万次。它就像大模型的「Steam」，一行命令就能下载运行模型。

### 1\. 安装 Ollama

**macOS** ：

```bash
代码解读复制代码# 方式一：Homebrew 安装
brew install ollama

# 方式二：官网下载安装包
# 访问 https://ollama.com/download 下载 macOS 版本
```

**Windows** ：

```powershell
代码解读复制代码# 直接下载 exe 安装包
# 访问 https://ollama.com/download 下载 Windows 版本
# 安装后自动作为系统服务运行
```

**Linux** （Ubuntu/Debian）：

```bash
代码解读复制代码curl -fsSL https://ollama.com/install.sh | sh
# 安装后自动启动 ollama 服务
sudo systemctl status ollama  # 确认服务状态
```

### 2\. 下载并运行 DeepSeek

```bash
代码解读复制代码# 运行 7B 版本（适合大多数电脑）
ollama run deepseek-r1:7b

# 运行 14B 版本（需要 16GB+ 内存）
ollama run deepseek-r1:14b

# 运行 V4 完整版（需要 32GB+ 内存）
ollama run deepseek-v4

# 运行量化版（更省内存）
ollama run deepseek-r1:14b-q4_K_M
```

第一次运行会自动下载模型文件（7B 约 4.4GB，14B 约 8.7GB）。下载完成后即可在终端直接对话。

### 3\. 验证运行

```bash
代码解读复制代码# 查看已安装的模型
ollama list

# 测试推理能力
ollama run deepseek-r1:7b "用Python写一个快速排序，并解释时间复杂度"
```

### 4\. 开放 API 接口

Ollama 默认在 `localhost:11434` 提供 OpenAI 兼容的 API，这意味着 **你现有的代码几乎不用改** 就能切换到本地模型：

```bash
代码解读复制代码# 启动时指定监听地址（让局域网其他设备也能访问）
OLLAMA_HOST=0.0.0.0 ollama serve
```
```python
代码解读复制代码# Python 调用示例——和调用 OpenAI 几乎一样
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama"  # 随意填写，Ollama 不校验
)

response = client.chat.completions.create(
    model="deepseek-r1:14b",
    messages=[
        {"role": "system", "content": "你是一个专业的Python编程助手。"},
        {"role": "user", "content": "帮我写一个RESTful API，用FastAPI框架"}
    ],
    temperature=0.7,
    max_tokens=2000
)

print(response.choices[0].message.content)
```

## 方案二：LM Studio —— 可视化界面，新手友好

如果你不习惯命令行，LM Studio 是个很好的选择。它提供图形界面，下载模型、对话测试、API服务一站式搞定。

### 安装步骤

1. 访问 [lmstudio.ai](https://link.juejin.cn/?target=https%3A%2F%2Flmstudio.ai "https://lmstudio.ai") 下载对应平台安装包
2. 安装后打开，在搜索栏输入 `DeepSeek`
3. 选择模型版本，点击下载
4. 下载完成后在左侧选择模型，点击「Load」
5. 在右侧聊天界面直接开始对话

### LM Studio 的优势

- ✅ **图形界面** ：搜索、下载、运行、对话，全程可视化
- ✅ **硬件监控** ：实时显示 GPU/CPU/内存占用
- ✅ **模型量化** ：内置 GGUF 格式转换，按需压缩模型
- ✅ **兼容 API** ：同样提供 OpenAI 兼容的本地 API 服务
- ✅ **跨平台** ：Mac/Windows/Linux 全支持

## 方案三：Docker 部署 —— 适合服务器/团队共享

如果是云服务器部署，给团队共享使用，Docker 方式最合适：

```yaml
代码解读复制代码# docker-compose.yml
version: '3.8'
services:
  ollama:
    image: ollama/ollama:latest
    container_name: deepseek-ollama
    ports:
      - "11434:11434"
    volumes:
      - ./ollama_data:/root/.ollama
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
    restart: unless-stopped

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: deepseek-webui
    ports:
      - "3000:8080"
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
    volumes:
      - ./open-webui_data:/app/backend/data
    depends_on:
      - ollama
    restart: unless-stopped
```
```bash
代码解读复制代码# 启动服务
docker compose up -d

# 进入容器下载模型
docker exec -it deepseek-ollama ollama pull deepseek-r1:14b

# 访问 Web 界面
# 浏览器打开 http://你的服务器IP:3000
```

这样就得到了一个 **带 Web 界面的本地 AI 服务** ，团队成员都可以通过浏览器访问。

## 进阶优化：让推理更快

### 1\. 模型量化

如果你的显存不够跑完整模型，可以用量化版（Q4、Q5、Q8）：

```bash
代码解读复制代码# 下载量化版模型（Q4_K_M 是性价比最高的量化级别）
ollama run deepseek-r1:14b-q4_K_M
```

| 量化级别 | 模型大小(14B) | 质量损失 | 显存需求 |
| --- | --- | --- | --- |
| Q8\_0 | ~14GB | 几乎无损 | 12GB+ |
| Q5\_K\_M | ~10GB | 极小 | 8GB+ |
| **Q4\_K\_M** | **~8.7GB** | **可接受** | **6GB+** |
| Q3\_K\_M | ~6.5GB | 明显 | 4GB+ |

### 2\. GPU 加速配置

**Linux（NVIDIA显卡）** ：

```bash
代码解读复制代码# 确认 CUDA 已安装
nvidia-smi

# 设置 Ollama 使用 GPU
export CUDA_VISIBLE_DEVICES=0  # 指定GPU编号
ollama serve
```

**Mac（Apple Silicon）** ： Ollama 默认使用 Metal 加速，无需额外配置。M 系列芯片的统一内存架构天然适合跑大模型。

### 3\. 并发请求优化

```bash
代码解读复制代码# 设置并发数（根据显存调整）
OLLAMA_NUM_PARALLEL=4 ollama serve

# 设置最大加载模型数
OLLAMA_MAX_LOADED_MODELS=2 ollama serve
```

## 接入现有项目：3分钟迁移指南

本地部署最爽的一点是—— **你的现有代码几乎不用改** 。以常见的几个场景为例：

### 接入 LangChain

```python
代码解读复制代码from langchain_community.chat_models import ChatOllama
from langchain_core.messages import HumanMessage

# 只需要改这一行
chat = ChatOllama(model="deepseek-r1:14b", base_url="http://localhost:11434")

response = chat.invoke([HumanMessage(content="解释什么是向量数据库")])
print(response.content)
```

### 接入 Dify / FastGPT

这些低代码 AI 平台都支持自定义模型端点：

1. 进入平台设置 → 模型管理
2. 添加自定义模型供应商
3. API Base 填： `http://localhost:11434/v1`
4. API Key 填： `ollama`
5. 模型名称填： `deepseek-r1:14b`

### 接入 Cursor / VS Code 插件

在编辑器的 AI 助手设置中：

- API Endpoint: `http://localhost:11434/v1`
- API Key: `ollama`
- Model: `deepseek-r1:14b`

这样你就能在编辑器里用本地 DeepSeek 做代码补全和问答了。

## 常见问题排查

**Q: 下载模型太慢怎么办？**

```bash
代码解读复制代码# 设置国内镜像源
export OLLAMA_ORIGINS="https://registry.ollama.ai"
# 或者手动下载 GGUF 文件后导入
ollama create my-model -f Modelfile
```

**Q: 内存不够报错？**

使用量化版模型（Q4\_K\_M），或者减少并发数。8GB 内存的电脑建议跑 7B 版本。

**Q: Mac M1 能跑吗？**

可以，M1 芯片 16GB 内存跑 7B 版本没问题，14B 量化版也能勉强跑。推荐 M2 及以上。

**Q: 推理速度太慢？**

- 确认 GPU 加速是否生效（运行时查看 `nvidia-smi` 或 Mac 的活动监视器）
- 降低模型参数规模
- 使用量化版本

## 总结

| 平台 | 推荐方案 | 一句话评价 |
| --- | --- | --- |
| Mac | Ollama | Apple Silicon 天然优势，开箱即用 |
| Windows | Ollama + LM Studio | Ollama 命令行 + LM Studio 可视化配合使用 |
| Linux 服务器 | Docker Compose | 团队共享，一键部署 Web 界面 |

本地部署 DeepSeek 已经不是什么高门槛的事了。一个 `ollama run` 命令就能跑起来，配合 OpenAI 兼容 API，现有项目无缝迁移。如果你还没试过，强烈建议今天就开始—— **当你在本地跑通第一个模型的时候，那种「数据完全在自己手里」的安全感，是云端 API 永远给不了的。**

---

**关于作者**

长期关注大模型应用落地与云服务器实战，专注技术在企业场景中的落地实践。

个人博客： **yunduancloud.icu** —— 持续更新云计算、AI大模型实战教程，欢迎访问交流。