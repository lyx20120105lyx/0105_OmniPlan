# 0105_OmniPlan

> 用 GitHub 开源项目 + AI Agent 自由地玩计算机科学。

## 这是什么

0105_OmniPlan 是一套「边玩边学」的计算机学习方案：

- **以 GitHub 优秀学习项目为玩具**——不啃教材，直接拿真项目玩
- **以 AI / Agent 为搭子**——随时问、随时聊、随时陪你造东西
- **以文件夹为单元管理计划**——每个想玩的学科一个文件夹，想到哪记到哪

## 目录结构

```
0105_OmniPlan/
├── README.md              # 本文件：总览与使用说明
└── <学科计划文件夹>/       # 例如：01-操作系统、02-网络、03-数据库...
    ├── PLAN.md            # 想玩什么 + 随手记的一句话进度
    ├── projects/          # 下载的 GitHub 学习项目
    └── notes/             # 想到什么记什么，格式随意
```

## 快速开始

1. **克隆本仓库**
   ```bash
   git clone https://github.com/lyx20120105lyx/0105_OmniPlan.git
   cd 0105_OmniPlan
   ```

2. **创建学科计划**
   ```bash
   mkdir 01-操作系统/projects
   ```

3. **挑个想玩的 GitHub 项目**（按兴趣搜，别管 star 多高）
   ```bash
   git clone <项目地址> 01-操作系统/projects/
   ```

4. **叫上 AI 搭子开玩**
   - 把项目代码丢给 AI：「这项目怎么跑起来？给我讲讲」
   - 玩到哪想到哪，随手把一句话感想写进 `PLAN.md`

## 计划模板（PLAN.md）

```markdown
# 学科：<名称>
- 想玩什么：<一句话，比如"我想爬豆瓣影评做词云">
- 手里有什么：<GitHub 项目列表>
- 进度（一句话就够，想写才写）：
  - 08-17 跑通了 Scrapy 示例
  - 08-18 改选择器抓了自己爱逛的网站
```

## 自由学习法

> 不背概念、不考试、不打时间表。像打游戏一样：想玩什么玩什么，玩着玩着就会了。

### 三条原则

1. **兴趣优先**：想学什么就学什么；卡住了就换个玩，回头再战
2. **做中学**：不先啃书——clone 项目跑起来，改着改着就懂了
3. **想问就问**：AI 是随时在线的搭子，不是出卷子的老师

### 五种玩法（挑你喜欢的）

- **玩项目**：clone 感兴趣的项目 → 跑起来 → 改界面、改参数、加功能
- **拆项目**：读别人源码，看不懂就问 AI；看懂了就自己拼一个
- **造东西**：脑子里冒什么就造什么——爬虫、小游戏、网站、bot、自动化脚本
- **跟教程**：想跟就跟（30 天系列、官方课程），不想跟就跳着看
- **写出来**：玩出好玩的东西就发博客 / GitHub，边写边懂

### Agent 是你的搭子

| 用途 | 怎么聊 |
|------|--------|
| 讲解 | 「这行代码在干嘛？用大白话讲」 |
| 陪玩 | 「我想做个 XX，帮我起个头，剩下的我自己改」 |
| 陪 Debug | 「这个报错我们一起推理，先别直接给答案」 |
| 审查 | 「提交前帮我看一眼：有没有密钥、敏感数据」 |

### 教学模型：Hermes + DeepSeek API

两个模型搭配，各管一摊：

| 分工 | 模型 | 接入方式 |
|------|------|----------|
| 讲解 / 出主意 | Nous Hermes 4 70B | OpenRouter API |
| 陪推理 / 陪 Debug | DeepSeek Reasoner (R1) | DeepSeek API |
| 干活（写代码/脚本/整理） | DeepSeek Chat (V3) | DeepSeek API |

选型理由：

- **Hermes 4**：agentic 能力强、指令跟随好，适合长对话讲解
- **DeepSeek Reasoner**：推理强，适合陪你一步步推 bug、想方案
- **DeepSeek Chat**：响应快又便宜，适合批量生成代码、写脚本

先设置环境变量：

```powershell
$env:OPENROUTER_API_KEY = "sk-or-..."
$env:DEEPSEEK_API_KEY   = "sk-..."
```

Python 调用示例（OpenAI SDK 兼容）：

```python
import os
from openai import OpenAI

def ask(model, base_url, api_key, prompt):
    client = OpenAI(base_url=base_url, api_key=api_key)
    resp = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}],
    )
    return resp.choices[0].message.content

# 讲解（Hermes via OpenRouter）
print(ask("nousresearch/hermes-4-70b",
          "https://openrouter.ai/api/v1",
          os.environ["OPENROUTER_API_KEY"],
          "这行代码在干嘛？用类比讲清 HTTP 缓存"))

# 陪 Debug（DeepSeek R1）
print(ask("deepseek-reasoner",
          "https://api.deepseek.com",
          os.environ["DEEPSEEK_API_KEY"],
          "这个报错我们一起推理，先别直接给答案：<粘贴报错>"))

# 干活（DeepSeek V3）
print(ask("deepseek-chat",
          "https://api.deepseek.com",
          os.environ["DEEPSEEK_API_KEY"],
          "帮我把「爬取豆瓣影评做词云」这个想法变成一个能跑的 Flask 小项目"))
```

opencode 配置示例（把两个模型接入 opencode）：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "openrouter": {
      "options": { "apiKey": "{env:OPENROUTER_API_KEY}" },
      "models": {
        "nousresearch/hermes-4-70b": { "name": "Hermes4 讲解" }
      }
    },
    "deepseek": {
      "options": { "apiKey": "{env:DEEPSEEK_API_KEY}" },
      "models": {
        "deepseek-chat": { "name": "DeepSeek 干活" },
        "deepseek-reasoner": { "name": "DeepSeek 陪练" }
      }
    }
  }
}
```

## 建议学科清单

- 操作系统 / 计算机网络 / 数据库
- 编译原理 / 数据结构与算法
- 编程语言与框架 / Web 开发
- 安全（可结合 [OWASP](https://owasp.org) 与安全类开源项目）
- 系统设计 / 分布式系统

## 推荐学习项目（40 个）

> 全部为真实存在的 GitHub 公开仓库，均已验证可访问。挑选原则：官方出品优先、star 高、维护活跃、适合新手跟学。

### Python · 基础

| 项目 | 说明 |
|------|------|
| [microsoft/Python-for-Beginners](https://github.com/microsoft/Python-for-Beginners) | 微软官方 Python 入门课程，含 Notebook 实验 |
| [Asabeneh/30-Days-Of-Python](https://github.com/Asabeneh/30-Days-Of-Python) | 30 天系统学 Python，循序渐进 |
| [jackfrued/Python-100-Days](https://github.com/jackfrued/Python-100-Days) | Python 100 天（中文），覆盖面广 |
| [michaelliao/learn-python3](https://github.com/michaelliao/learn-python3) | 廖雪峰 Python 3 教程（中文） |
| [TheAlgorithms/Python](https://github.com/TheAlgorithms/Python) | 用 Python 实现各种算法的经典合集 |
| [realpython/python-guide](https://github.com/realpython/python-guide) | Python 最佳实践指南（Hitchhiker's Guide） |
| [vinta/awesome-python](https://github.com/vinta/awesome-python) | Python 生态资源大全，按需查库 |

### Python · 爬虫

| 项目 | 说明 |
|------|------|
| [scrapy/scrapy](https://github.com/scrapy/scrapy) | 最流行的 Python 爬虫框架，官方仓库即最佳教材 |
| [Python3WebSpider/ProxyPool](https://github.com/Python3WebSpider/ProxyPool) | 崔庆才的代理池实战项目，爬虫进阶必学 |
| [jhao104/proxy_pool](https://github.com/jhao104/proxy_pool) | 高可用代理池（采集/验证/API 接口） |
| [luyishisi/Anti-Anti-Spider](https://github.com/luyishisi/Anti-Anti-Spider) | 反反爬虫教程，学懂爬虫对抗原理 |

### Python · Web（Flask）

| 项目 | 说明 |
|------|------|
| [pallets/flask](https://github.com/pallets/flask) | Flask 官方仓库，源码 + 文档双学 |
| [greyli/helloflask](https://github.com/greyli/helloflask) | 《Flask Web 开发实战》配套代码（中文） |
| [greyli/flask-tutorial](https://github.com/greyli/flask-tutorial) | Flask 官方教程中文精讲版 |

### Python · 深度学习（PyTorch）

| 项目 | 说明 |
|------|------|
| [pytorch/tutorials](https://github.com/pytorch/tutorials) | PyTorch 官方教程（入门到实战） |
| [pytorch/examples](https://github.com/pytorch/examples) | PyTorch 官方示例代码合集 |
| [yunjey/pytorch-tutorial](https://github.com/yunjey/pytorch-tutorial) | 经典 PyTorch 教程，代码精炼易跟学 |

### C / C++

| 项目 | 说明 |
|------|------|
| [TheAlgorithms/C](https://github.com/TheAlgorithms/C) | C 语言算法实现合集 |
| [TheAlgorithms/C-Plus-Plus](https://github.com/TheAlgorithms/C-Plus-Plus) | C++ 算法实现合集 |
| [AnthonyCalandra/modern-cpp-features](https://github.com/AnthonyCalandra/modern-cpp-features) | 现代 C++ 特性速览（C++11 到 C++23） |
| [isocpp/CppCoreGuidelines](https://github.com/isocpp/CppCoreGuidelines) | C++ 核心准则，官方编码规范圣经 |
| [fffaraz/awesome-cpp](https://github.com/fffaraz/awesome-cpp) | C++ 资源大全 |
| [google/styleguide](https://github.com/google/styleguide) | Google 编码规范（含 C++、Python 等） |

### C# / .NET

| 项目 | 说明 |
|------|------|
| [TheAlgorithms/C-Sharp](https://github.com/TheAlgorithms/C-Sharp) | C# 算法实现合集 |
| [dotnet-architecture/eShopOnWeb](https://github.com/dotnet-architecture/eShopOnWeb) | 微软官方 .NET 参考架构，学 ASP.NET Core 首选 |
| [dotnet/csharp-notebooks](https://github.com/dotnet/csharp-notebooks) | C# 交互式 Notebook 教程 |
| [quozd/awesome-dotnet](https://github.com/quozd/awesome-dotnet) | .NET 资源大全 |

### AI / 机器学习课程

| 项目 | 说明 |
|------|------|
| [microsoft/AI-For-Beginners](https://github.com/microsoft/AI-For-Beginners) | 微软「全民 AI」课程，AI 入门首选 |
| [microsoft/ML-For-Beginners](https://github.com/microsoft/ML-For-Beginners) | 微软「全民机器学习」课程 |
| [microsoft/Data-Science-For-Beginners](https://github.com/microsoft/Data-Science-For-Beginners) | 微软「全民数据科学」课程 |
| [microsoft/Generative-AI-for-beginners](https://github.com/microsoft/Generative-AI-for-beginners) | 微软生成式 AI 课程（LLM、RAG、Agent） |
| [microsoft/Web-Dev-For-Beginners](https://github.com/microsoft/Web-Dev-For-Beginners) | 微软「全民 Web 开发」课程 |
| [fastai/fastbook](https://github.com/fastai/fastbook) | fast.ai 深度学习教材（代码随书） |
| [d2l-ai/d2l-zh](https://github.com/d2l-ai/d2l-zh) | 《动手学深度学习》中文版，理论与代码兼备 |
| [huggingface/deep-rl-class](https://github.com/huggingface/deep-rl-class) | Hugging Face 深度强化学习课程 |
| [dair-ai/Prompt-Engineering-Guide](https://github.com/dair-ai/Prompt-Engineering-Guide) | 提示工程指南，AI 时代必备 |

### 通用 / 项目驱动学习

| 项目 | 说明 |
|------|------|
| [practical-tutorials/project-based-learning](https://github.com/practical-tutorials/project-based-learning) | 项目驱动学习大全（多语言），边做边学 |
| [ossu/computer-science](https://github.com/ossu/computer-science) | 开源计算机科学学位路线图 |
| [kamranahmedse/developer-roadmap](https://github.com/kamranahmedse/developer-roadmap) | 开发者成长路线图（前端/后端/运维/安全） |
| [jwasham/coding-interview-university](https://github.com/jwasham/coding-interview-university) | 计算机系统学习 + 面试准备 |
| [EbookFoundation/free-programming-books](https://github.com/EbookFoundation/free-programming-books) | 免费编程书籍/课程大全 |

### 使用方法

把想学的项目 `git clone` 到对应学科文件夹，再交给 AI Agent：

```bash
git clone https://github.com/microsoft/AI-For-Beginners.git 05-AI/projects/AI-For-Beginners
```

## 许可

MIT
