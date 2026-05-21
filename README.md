# Qoreon

> A local control layer between human intent and AI execution.
> Organize, coordinate, and continuously improve a team of AI agents -- not by using one AI at a time, but by managing an AI team with visible task spaces, channels, feedback loops, and knowledge sediment.

---

## English Summary

Qoreon is a **local-first multi-agent coordination platform** built in Python. It provides a control layer that sits on top of existing AI CLI tools (Codex CLI, Claude Code, OpenCode, Gemini CLI, Trae CLI), organizing them into a structured team with governance channels, task management, session health monitoring, and cross-agent collaboration. Instead of interacting with a single AI, users manage an AI team through a unified dashboard with visible task boards, message flow, agent directories, and relationship maps.

---

## Problem Statement

### 要解决的问题

大多数 AI 工具的工作模式停留在"一次提问，一次回答"。当你有多个 AI CLI 需要同时使用时，缺乏以下能力：

- **不可见**: 多个 Agent 各自运行，协作过程黑盒化，无法全局查看任务状态与流转
- **不可控**: 没有统一的调度层，Agent 之间缺乏结构化的消息收发、回执与阻塞机制
- **不可积累**: 协作过程中产生的知识、经验和决策无法沉淀为可复用的技能包
- **不可接管**: 换一台电脑或换一个 AI，无法无缝接手之前的协作上下文

Qoreon 面向的是一种全新的工作方式：**把任务空间、治理通道、协作回执和 Agent 执行组织成一个可见、可接管、可继续优化的本地系统。**

---

## Core Features

### 核心功能

- **任务看板系统**: 以 Markdown 文件为真源的任务空间，支持任务状态追踪、优先级管理和跨通道分发
- **多 Agent 通道治理**: 内置 12 个治理通道（总控、运行时、前端、数据、测试、文档、运维等），每个通道绑定独立的 AI CLI 会话
- **消息流转可视化**: Message Flow Board 实时展示多 Agent 派发、回执、阻塞状态和跨通道协同
- **会话健康监控**: 自动巡检 Agent 会话状态，提供会话轮换建议和运行检查
- **Agent 关系图谱**: 可视化 Agent 间的协作关系和依赖链路
- **知识沉淀体系**: 每个通道的任务产出自动沉淀为可复用的技能包（Skills），支持 8 个公共技能 + 通道专属技能
- **AI Bootstrap 机制**: 新电脑安装后，生成的 startup-batch.md 可交给本机 AI 自动接管后续协作
- **CLI 适配器架构**: 支持 Codex CLI、Claude Code、OpenCode、Gemini CLI、Trae CLI，通过统一适配器接口接入

---

## Architecture

### 技术架构

```
qoreon/
├── server.py                          # Python HTTP 服务 + CCB (CLI Control Bridge) API
├── build_project_task_dashboard.py    # 静态页面构建引擎
├── config.toml                        # 项目配置 (TOML 格式)
│
├── task_dashboard/                    # 核心 Python 引擎
│   ├── adapters/                      # CLI 适配器层
│   │   ├── codex_adapter.py
│   │   ├── claude_adapter.py
│   │   ├── opencode_adapter.py
│   │   ├── gemini_adapter.py
│   │   └── trae_adapter.py
│   ├── runtime/                       # 运行时子系统
│   │   ├── execution_runtime.py       # CLI 执行、网络重试、进程管理
│   │   ├── session_routes.py          # 会话路由与会话管理
│   │   ├── run_routes.py              # Run 生命周期管理
│   │   ├── heartbeat_registry.py      # 心跳巡检注册表
│   │   ├── scheduler_registry.py      # 自动调度注册表
│   │   └── assist_request_registry.py # 辅助请求注册表
│   ├── routes/                        # HTTP 路由分发器
│   ├── session_store.py               # 会话存储
│   ├── communication_audit.py         # 通信审计分析
│   ├── global_resource_graph.py       # 全局资源图谱
│   └── domain.py                      # 领域模型
│
├── web/                               # 页面模板 (CSS/JS/HTML.tpl)
│   ├── task_*.html.tpl                # 任务看板页面
│   ├── overview_*.html.tpl            # 项目概览页面
│   ├── communication_*.html.tpl       # 通信审计页面
│   ├── agent_directory_*.html.tpl     # Agent 目录页面
│   ├── agent_relationship_board_*.html.tpl  # 关系图谱页面
│   ├── session_health_*.html.tpl      # 会话健康页面
│   └── status_report_*.html.tpl       # 状态报告页面
│
├── examples/standard-project/         # 公开标准项目模板
│   ├── seed/                          # 种子数据 (CCR 花名册、任务种子)
│   ├── skills/                        # 技能包定义
│   └── tasks/                         # Markdown 任务空间
│
├── scripts/                           # 安装与启动脚本
│   ├── start_standard_project.py      # 一键启动入口
│   ├── bootstrap_public_example.py    # 项目引导
│   ├── install_public_bundle.py       # 通用安装器
│   └── activate_public_example_agents.py  # Agent 激活
│
├── docs/public/                       # 公开文档
│   ├── ai-bootstrap.md                # AI 接管入口文档
│   ├── quick-start.md                 # 快速开始
│   └── architecture.md                # 架构说明
│
└── tests/                             # 测试套件 (25+ 测试用例)
```

**设计原则:**
- **Local-First**: 默认绑定 `127.0.0.1:18770`，所有数据在本地
- **Markdown as Source of Truth**: 任务空间、治理结构、知识沉淀均以 Markdown 文件为真源
- **Zero Cloud Dependency**: 不依赖远端服务，所有运行态数据在 `.runtime/` 目录下
- **Extensible CLI Adapters**: 新增 AI CLI 只需实现适配器接口

---

## Technical Highlights

### 技术亮点

1. **CLI 控制桥 (CCB) 架构**: 通过统一的 `CLIAdapter` 抽象层，将 5 种不同的 AI CLI 工具整合为一致的运行时接口，支持进程级生命周期管理、输出解析和会话恢复

2. **Markdown 驱动的任务引擎**: 将文件系统目录结构映射为任务状态机，自动扫描 `tasks/{channel}/{任务|反馈|产出物|沉淀}/` 目录，实时构建看板视图

3. **多注册表运行时**: 运行时层包含心跳注册表、调度注册表、任务推送注册表、辅助请求注册表等多个自治子系统，各自独立注册和调度

4. **进程级会话感知**: 通过 `ps` 扫描系统进程表，使用正则表达式匹配不同 CLI 的参数特征（如 codex 的 `resume <session>`、claude 的 `--resume`），精确判断 Agent 会话的忙闲状态

5. **网络容错与自动重试**: 内置指数退避重试机制，自动识别传输通道断开、TLS 握手失败等瞬态网络错误，支持断线续传和中断恢复

6. **AI 可接管的 Bootstrap 设计**: 安装完成后自动生成 `startup-batch.md` + `ai-bootstrap.md`，另一台电脑上的 AI 可以阅读这些文档并自动完成项目初始化

7. **通信审计分析**: 从 `.runs/` 目录下扫描历史运行产物，分析 Agent 间的消息模式、响应延迟和协作瓶颈

---

## Results

### 成果与进展

| 指标 | 数据 |
|------|------|
| **Python 模块** | 50+ 模块，覆盖运行时、适配器、路由、调度等领域 |
| **CLI 适配器** | 5 种 AI CLI 工具（Codex, Claude, OpenCode, Gemini, Trae） |
| **治理通道** | 12 个标准化治理通道 |
| **公共技能包** | 8 个可复用技能（启动、培训、协作、巡检、轮换等） |
| **测试用例** | 25+ 单元测试覆盖核心路径 |
| **页面仪表盘** | 8 个功能页面（任务看板、概览、通信审计、状态报告、Agent 目录、关系图谱、会话健康、消息风险） |
| **预览版本** | `v1-preview-20260407-b` |

---

## Reflection

### 反思与成长

- **单一项目的力量**: 公开包只保留 `standard_project` 这一条默认路径，避免了"第一次使用就分叉"的问题。这是产品设计上的一次重要收敛：让安装路径、AI 接管路径、治理结构和验收口径全部指向同一个工作区。

- **AI 可接管性是核心设计目标**: 这个项目的本质不是"让用户操作"，而是"让 AI 接手"。Bootstrap 文档、startup-batch、通道沉淀的设计，都是在解决"另一个 AI 如何理解当前上下文并继续工作"这个问题。

- **Markdown 作为真源的双刃剑**: 用 Markdown 文件作为任务系统的真源让数据可读、可版本控制、AI 可直接理解，但也意味着需要处理文件扫描性能、增量更新和一致性问题。通过 TTL 缓存和增量检查器来缓解。

- **进程扫描是权宜之计**: 通过 `ps` 扫描进程表来判断 Agent 忙闲状态是一种 best-effort 方案，依赖正则匹配 CLI 参数特征。更长期的方案应该是 CLI 原生支持进程状态上报。

- **适配器模式的价值**: CLI 适配器架构让系统可以在不修改核心逻辑的前提下接入新的 AI 工具，这是扩展性最好的设计决策之一。

---

## Project Status

| | |
|---|---|
| **Version** | V1 Preview (`v1-preview-20260407-b`) |
| **Status** | Development / Preview |
| **Language** | Python 3.11+ |
| **License** | MIT |
| **Default Port** | `127.0.0.1:18770` |

### Quick Start

```bash
# Python 3.11+ required
python3 scripts/start_standard_project.py

# Then open:
# http://127.0.0.1:18770/project-task-dashboard.html
# http://127.0.0.1:18770/project-overview-dashboard.html
```

### With Agent Activation

```bash
python3 scripts/start_standard_project.py --with-agents
```

---

*Qoreon -- Connect human intent with AI execution.*
