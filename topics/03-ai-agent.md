# 03. AI Agent（智能体）

> 研究时间：2026-04-08 | 研究者：Kiko Sub-Agent #03 | 运行时长：2分53秒

---

## 2024-2025 重要产品/发布

### OpenAI Operator（2025.1）
- 首款 CUA（Computer-Using Agent）产品，基于 GPT-4o 多模态
- 视觉感知 + 模拟鼠标操作，自主执行网页任务
- WebArena 和 WebVoyager 浏览器基准刷新 SOTA
- 生态合作：DoorDash、Instacart、OpenTable 等

### Anthropic Claude Computer Use（2024.3）
- Claude 直接操控鼠标/键盘，在虚拟机中操作真实界面
- 2026.3 扩展至 Windows，支持邮件/文件/PC任务

### OpenAI Deep Research（2025.2）
- 专注于网络深度研究任务的 Agent
- 自主规划、搜索、整合多源信息生成报告

### GPT-5 系列（2025-2026）
- 原生工具调用链（Tool Use Chain）、Computer Use、1M token 超长上下文
- 代码生成和 Agent 工作流显著提升

---

## 核心架构模式

| 模式 | 原理 | 代表 |
|------|------|------|
| **ReAct** | Thought → Action → Observation 循环，推理可解释 | LangChain Agent |
| **Plan-Execute** | 规划器拆解任务 → 执行器逐个执行 | BabyAGI |
| **MRKL** | LLM 作为路由器，动态调用专家模块 | Anthropic Tool Use |
| **AutoGPT** | 完全自主驱动，分解子目标循环执行 | AutoGPT、GPT-Engineer |
| **Reflection** | 执行后自我评估和修正 | Reflexion |
| **Tool Calling** | LLM 原生结构化 API 调用 | OpenAI function calling、MCP |

**MCP 协议**（Anthropic 主导）正在成为 Agent 工具调用的事实标准。

---

## 多智能体协作进展

### 协作模式
| 模式 | 说明 |
|------|------|
| 串行执行 | 流水线，上一个 Agent 输出作为下一个输入 |
| 并行执行 | Map-Reduce，多 Agent 独立处理不同子集 |
| 层级协作 | Manager Agent 协调多个 Worker Agent |
| 对等协作 | Agent 间协商辩论 |
| 竞争协作 | 多 Agent 提案 → 评判 Agent 选优 |

### 主流框架对比
| 框架 | 核心优势 | 适用场景 |
|------|---------|---------|
| **LangGraph** | 图执行引擎，支持循环/条件/状态 | 复杂有状态工作流 |
| **AutoGen（微软）** | 多 Agent 对话原生，人机混合 | 研究实验、软件自动化 |
| **CrewAI** | 角色定义清晰，流程简洁 | 垂直场景 Pipeline |
| **OpenAI Swarm** | 轻量级 Handoff 协调 | 探索性实验 |
| **MetaGPT** | 软件公司角色分工，输出结构化中间产物 | 复杂软件工程任务 |

### 关键挑战
- 通信开销：多 Agent 大量 LLM 调用，成本叠加
- 状态同步：共享上下文需要专门的 Memory 管理
- 死锁/循环：无界循环是最大工程风险
- 评测困难：缺乏可靠的 Multi-Agent Benchmark

---

## 记忆与上下文管理

### 统一记忆分类体系（arXiv:2512.13564）

**形式维度（Forms）**

| 类型 | 机制 | 特点 |
|------|------|------|
| Token-level | 上下文窗口 + RAG 检索注入 | 受限 Context 长度 |
| Parametric | 编码进模型权重（SFT/LoRA） | 更新成本高 |
| Latent | 隐藏层激活向量压缩（MemGPT） | 信息密度高，缺可解释性 |

**功能维度（Functions）**
- Factual Memory：客观世界知识
- Experiential Memory：交互事件、用户偏好
- Working Memory：当前任务临时缓冲区

### Agent RAG vs Q&A RAG
| 维度 | Q&A RAG | Agent RAG |
|------|---------|-----------|
| 检索时机 | 单次查询 | 多跳、链式检索 |
| 数据来源 | 静态文档 | 动态：文档 + 工具结果 + 观察 |
| 上下文 | 单次查询 | 跨多工具调用计划级上下文 |

### 工作记忆 Token 预算（保守估算）
```
系统提示词：   ~2,000 tokens
对话历史：     ~10,000 tokens
检索上下文：   ~5,000 tokens
任务工作区：   ~3,000 tokens
输出预留：     ~2,000 tokens
────────────────────────────
总计：         ~22,000 tokens（保守）
```

---

## 主流框架对比

| 框架 | 上手难度 | 核心优势 |
|------|---------|---------|
| LangChain | ★★☆ | 生态最全，集成最广 |
| LangGraph | ★★★ | 图执行，支持循环/条件/状态 |
| AutoGen | ★★☆ | 微软背书，多 Agent 对话原生 |
| CrewAI | ★★☆ | 角色定义清晰，流程简洁 |
| LlamaIndex | ★★☆ | 数据连接和 RAG 最强 |
| Semantic Kernel | ★★★ | 微软生态深度集成 |

**选型建议**：快速原型 → LangChain；复杂有状态工作流 → LangGraph；微软技术栈 → Semantic Kernel

---

## 实战建议

### 渐进式构建路径
```
第一阶段：ReAct 单 Agent（1-2天）
第二阶段：加入记忆（1周）
  ↓ RAG + 对话历史 + 实体提取
第三阶段：多 Agent 协作（2-3周）
  ↓ CrewAI / LangGraph 编排
第四阶段：生产优化
  ↓ 可观测性 + 成本控制 + 人工确认节点
```

### 工具调用最佳实践
- 优先使用 Function Calling / Tool Use
- MCP 协议尽量采用
- 工具描述要精确：参数名、类型、约束、返回格式

### 成本控制原则
- 步数限制：每个 Agent 最多 N 步
- Token 预算严格执行
- 关键决策节点（支付/删除/发邮件）强制人工确认

---

## 推荐资源

- *[Memory in the Age of AI Agents](https://arxiv.org/abs/2512.13564)* — 记忆分类体系论文
- [Anthropic Computer Use Docs](https://platform.anthropic.com/docs) — Claude 官方文档
- [LangSmith](https://docs.smith.langchain.com/) — Agent 可观测性
- [MCP 协议](https://modelcontextprotocol.github.io/) — Anthropic 主导的工具互操作标准
- [MemGPT](https://github.com/cpacker/MemGPT) — 层级记忆管理
