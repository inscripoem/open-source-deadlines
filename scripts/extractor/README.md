> **Note**: This README originated from the
> [open-source-deadlines-data](https://github.com/hust-open-atom-club/open-source-deadlines-data)
> repository and was imported on 2026-06-22 as part of the data-repo
> integration plan (refs #116). In this monorepo the extractor lives at
> `scripts/extractor/`, alongside the existing `scripts/check_data.py` data
> validator. When the README below mentions paths like
> `extractor/extract_activity.py`, use `scripts/extractor/extract_activity.py`
> (or `cd scripts/extractor` and drop the prefix). The `--data-dir` default
> has been updated to resolve to the repo-root `data/` directory automatically.

# OSEDDL 数据提取工具

使用 AI Agent 从网页自动提取开源会议、竞赛、活动等信息，并保存为结构化的 YAML 数据。

## 功能

- 基于 LLM 的智能信息提取（支持 OpenAI、GitHub Models、DashScope）
- 自动生成结构化 YAML 格式数据
- 智能标签识别和去重
- 避免重复提取（基于 ID 检查）
- 支持多种工作模式（交互式、自动化、GitHub Actions）
- 支持从 URL 或本地文件提取

## 快速开始

### 前置要求

- Python 3.13+
- 已安装 [uv 包管理器](https://docs.astral.sh/uv/)
- AI API 凭证（三选一）

### 安装依赖

```bash
# 同步项目依赖
uv sync
```

### 环境变量配置

在项目根目录创建 `.env` 文件，选择一种 AI 供应商：

**方案 1: GitHub Models**
```env
AI_PROVIDER=github
GITHUB_TOKEN=your_github_token
AI_MODEL=gpt-4o-mini
```

**方案 2: DashScope（阿里云）**
```env
AI_PROVIDER=dashscope
DASHSCOPE_API_KEY=your_dashscope_key
DASHSCOPE_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
AI_MODEL=qwen-plus
```

**方案 3: OpenAI（默认）**
```env
AI_PROVIDER=openai
OPENAI_API_KEY=your_openai_api_key
AI_MODEL=gpt-4o-mini
```

## 使用方式

### 1. 交互式提取（本地开发）

```bash
uv run python extractor/extract_activity.py --url https://example.com/activity
```

特点：
- 交互式展示提取结果
- 提取失败时询问如何处理
- 提示是否保存到数据文件
- 适合逐条录入和验证

**可选参数：**
```bash
--url URL                  # 活动网页 URL（必需）
--data-dir PATH           # 数据目录，默认为脚本所在目录的上一级 data/（即仓库根 data/）
--from-file FILE          # 从本地文件提取（二选一）
```

### 2. 自动提取（GitHub Actions 专用）

```bash
uv run python extractor/extract_for_actions.py \
  --url https://example.com/activity \
  --data-dir ./data
```

特点：
- 自动确认保存，无需人工交互
- 输出格式化结果供 GitHub Actions 使用
- 出错时自动记录并退出

**可选参数：**
```bash
--url URL                 # 活动网页 URL（必需）
--data-dir PATH          # 数据目录，默认解析到仓库根 data/（无需手动指定）
--no-save               # 仅输出提取结果，不保存
```

### 3. GitHub Actions 工作流集成
预计在工作流中集成

## 数据格式

提取的活动信息会保存为 YAML 格式，结构如下：

```yaml
- title: "活动名称"
  category: "activity"  # activity | competition | conference
  description: "活动描述"
  tags: ["开源", "技术"]
  website: "https://example.com"
  location: "线上/城市名"
  start_date: "2024-01-01"
  end_date: "2024-01-31"
  events:
    - id: "unique-event-id"
      title: "具体赛题/环节"
      deadline: "2024-01-15"
      link: "https://example.com/event"
```

## 项目结构

在主仓 `open-source-deadlines` 中，工具与数据的相对位置如下：

```
open-source-deadlines/
├── data/                               # 数据文件目录（仓库根）
│   ├── activities.yml                  # 活动数据
│   ├── competitions.yml                # 比赛数据
│   └── conferences.yml                 # 会议数据
└── scripts/
    ├── check_data.py                   # 数据校验脚本（CI 使用，独立于本工具）
    ├── synonyms.yml                    # 校验脚本的同义词词典
    └── extractor/                      # 本工具（独立的 uv 项目）
        ├── ai_agent.py                 # AI Agent 核心（支持多个 AI 供应商）
        ├── extract_activity.py         # 交互式提取脚本
        ├── extract_for_actions.py      # GitHub Actions 自动化脚本
        ├── pyproject.toml              # 项目配置（uv 管理）
        ├── uv.lock                     # 依赖锁定文件
        ├── .python-version             # uv/pyenv 锁定的 Python 版本
        ├── .gitignore                  # 本工具专属的 ignore 规则
        └── README.md                   # 本文件
```

## 依赖说明

| 包名 | 用途 | 版本 |
|-----|------|------|
| `openai` | LLM API 调用（OpenAI/GitHub Models/DashScope 兼容） | >=1.0.0 |
| `httpx` | 异步 HTTP 客户端，用于网页抓取 | >=0.27.0 |
| `pyyaml` | YAML 文件处理 | >=6.0.0 |
| `python-dotenv` | 环境变量加载 | >=1.0.0 |
