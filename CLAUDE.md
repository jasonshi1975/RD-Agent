# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Language Preference

- 所有输入输出尽量使用中文
- UI界面、日志信息、提示信息等应使用中文
- 代码注释可使用英文，但面向用户的文本应使用中文

## Platform Support

**重要：RD-Agent 目前仅支持 Linux 平台。**

## Project Overview

RD-Agent 是微软的一个项目，通过 LLM 智能体自动化数据驱动场景下的研发流程。框架包含两个核心组件：'R'（Research/研究）负责提出想法，'D'（Development/开发）负责通过迭代演化实现这些想法。

## 常用命令

### 开发环境配置
```bash
# 创建 conda 环境（支持 Python 3.10 和 3.11）
conda create -n rdagent python=3.10
conda activate rdagent

# 以开发模式安装（包含所有可选依赖）
make dev
```

### 代码检查
```bash
make lint          # 运行所有检查器 (mypy, ruff, isort, black, toml-sort)
make auto-lint     # 自动修复代码风格问题
make ruff          # 仅运行 ruff（检查 rdagent/core）
make mypy          # 仅运行 mypy（检查 rdagent/core）
```

### 测试
```bash
make test          # 运行所有测试并生成覆盖率报告
make test-offline  # 仅运行离线测试（无需 API 调用）
pytest test/utils/test_env.py  # 运行单个测试文件
pytest -m offline  # 运行标记为 offline 的测试
```

### CLI 应用
```bash
rdagent health_check              # 验证环境配置（Docker、LLM 配置）
rdagent fin_quant                 # 量化交易因子与模型联合演化
rdagent fin_factor                # 金融因子演化
rdagent fin_model                 # 金融模型演化
rdagent fin_factor_report         # 从研报提取因子
rdagent general_model <paper_url> # 从论文提取模型
rdagent data_science --competition <name>  # 数据科学竞赛
rdagent ui --port 19899 --log-dir log/     # 启动监控界面
```

## 架构

### 核心框架 (`rdagent/core/`)
R&D 自动化框架的基础：
- **`experiment.py`**: Task、Workspace、Experiment 类 - 组织研发工作的核心抽象
- **`proposal.py`**: Hypothesis 和反馈机制，用于想法生成
- **`evolving_framework.py`**: EvolvingStrategy、RAGStrategy 用于迭代改进
- **`evolving_agent.py`**: RAGEvoAgent - 带 RAG 和评估的主演化循环
- **`scenario.py`**: Scenario 基类，用于特定领域实现
- **`evaluation.py`**: 评估相关功能
- **`knowledge_base.py`**: 知识存储与管理

### 组件 (`rdagent/components/`)
实现框架的可复用模块：
- **`coder/`**: 代码生成（CoSTEER、factor_coder、model_coder、data_science）
- **`agent/`**: 智能体基础设施（RAG、MCP 集成）
- **`proposal/`**: 想法提案机制
- **`runner/`**: 实验执行环境
- **`interactor/`**: 用户交互组件
- **`knowledge_management/`**: 知识存储与检索

### 场景 (`rdagent/scenarios/`)
特定领域实现：
- **`qlib/`**: 量化金融（因子、模型、策略）
- **`data_science/`**: 通用数据科学竞赛
- **`kaggle/`**: Kaggle 竞赛自动化
- **`general_model/`**: 论文到模型实现

### LLM 集成 (`rdagent/oai/`)
- 使用 **LiteLLM** 作为默认后端，支持多提供商
- 支持 OpenAI、Azure OpenAI、DeepSeek 等提供商
- 通过 `.env` 文件中的环境变量配置

## 核心模式

### 演化循环
核心模式是迭代演化：
1. **Query（查询）** 从 RAG 获取知识
2. **Evolve（演化）** 使用 EvolvingStrategy 演化对象
3. **Evaluate（评估）** 评估结果并获取反馈
4. **Generate knowledge（生成知识）** 从轨迹生成新知识
5. 重复直到完成或达到最大循环次数

### Task/Workspace 模式
- **Task**: 带描述的抽象工作单元
- **Workspace**: 具有执行能力的基于文件的实现区域
- **Experiment**: 包含假设和运行信息的任务集合

### Scenario 模式
每个场景提供：
- `background`: 领域背景
- `get_source_data_desc()`: 数据描述
- `get_runtime_environment()`: 环境设置

## 配置

### 环境变量 (`.env`)
```bash
# LLM 配置（LiteLLM 格式）
CHAT_MODEL=gpt-4o
EMBEDDING_MODEL=text-embedding-3-small
OPENAI_API_KEY=your_key

# DeepSeek 配置
CHAT_MODEL=deepseek/deepseek-chat
DEEPSEEK_API_KEY=your_key

# 推理模型（包含  sharedInstance<think> 标签）
REASONING_THINK_RM=True
```

### Docker 要求
大多数场景需要 Docker 进行隔离的代码执行。确保已安装 Docker，且当前用户可以不使用 sudo 运行 `docker`。

## 代码风格

- 行长度：120 字符
- 支持 Python 3.10 和 3.11
- 需要类型提示（mypy 在 `rdagent/core/` 上强制执行）
- 使用 Ruff 进行代码检查，选择性忽略规则（见 `pyproject.toml`）
- 异步操作使用 `asyncio`

## 测试标记

测试使用 pytest 标记：
- `@pytest.mark.offline` - 不需要 API 调用的测试
- `@pytest.mark.skip` - 暂时跳过的测试