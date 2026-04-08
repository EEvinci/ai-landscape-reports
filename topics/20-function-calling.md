# Function Calling & MCP & A2A：函数调用与Agent通信协议

> 研究日期：2025年
> 字数：约2000字

---

## 一、Function Calling：让大模型真正"干活"的核心机制

### 1.1 什么是Function Calling

Function Calling（函数调用）是一种让大语言模型（LLM）与外部工具、API和系统进行可靠交互的技术。它将LLM从被动的"文本生成器"转变为能够**主动执行任务**的智能代理。

核心工作流程如下：

1. **开发者注册**：向模型提供函数/工具的描述（名称、参数schema、用途说明）
2. **模型决策**：用户提出请求，模型判断是否需要调用工具以及调用哪个工具
3. **结构化输出**：模型返回符合JSON Schema的工具调用请求（函数名+参数）
4. **外部执行**：应用层负责实际执行该函数，将结果返回给模型
5. **综合响应**：模型结合工具执行结果，生成最终回答

> **关键区分**：LLM本身并不执行函数，它只"决定调用哪个函数并返回参数"，实际执行由应用程序完成。

### 1.2 标准化之战：OpenAI vs Anthropic格式

2024-2025年，两大主流厂商形成了两种不同的Function Calling格式标准：

| 维度 | OpenAI格式 | Anthropic (Tool Use) 格式 |
|------|-----------|--------------------------|
| **参数位置** | `tools` 数组 + `tool_choice` | `tools` 数组，嵌入 `type: "tool_use"` |
| **Schema定义** | JSON Schema（$schema、type、properties等） | 同样基于JSON Schema |
| **并行调用** | 支持 `parallel_tool_calls` | 原生支持并行工具调用 |
| **强制调用** | `tool_choice: { type: "function", function: {...} }` | 同理，通过参数控制 |
| **代表模型** | GPT-4o、GPT-4 Turbo、GPT-3.5 | Claude 3.5 Sonnet、Claude 3 Opus |

两者本质上都在用JSON Schema描述工具接口，核心差异在于**传输层包装**和**模型层面的行为控制参数**。行业趋势是趋向互操作——多数框架（如LangChain、LlamaIndex）已同时支持两种格式，并提供自动转换层。

---

## 二、MCP：Anthropic提出的Agent通信协议

### 2.1 背景与定位

2024年11月25日，Anthropic正式发布并开源了**Model Context Protocol（MCP）**，定位为AI与外部世界交互的"USB接口"标准。

MCP解决的核心问题是**"M×N问题"**：M个AI模型与N个数据源/工具之间，如果每对之间都需要独立开发连接器，复杂度为O(M×N)。MCP通过统一协议，将复杂度降为O(M+N)。

### 2.2 架构设计

MCP采用**客户端-服务器架构**：

- **MCP Host（主机）**：AI应用本身（如Claude Desktop）
- **MCP Client（客户端）**：嵌入主机的客户端组件
- **MCP Server（服务器）**：提供工具和数据的小型服务程序，每个Server暴露一组Resources（数据）、Tools（工具）和Prompts（提示模板）

```
┌─────────────┐      MCP       ┌──────────────┐
│  AI Model   │◄─────────────►│  MCP Server  │
│  (Host/     │   (标准协议)   │  (工具/数据)  │
│   Client)   │               └──────────────┘
└─────────────┘                    ▲
                                    │ 发现与调用
                         ┌──────────────┐
                         │  MCP Server  │
                         │  (e.g. GitHub│
                         │   Database)  │
                         └──────────────┘
```

### 2.3 生态进展（2024-2025）

截至2025年初，MCP生态已形成相当规模：

- **官方SDK**：Python + TypeScript，GitHub开源（`modelcontextprotocol`组织）
- **Claude Desktop集成**：Anthropic将MCP作为Claude Desktop的核心扩展机制
- **社区服务器爆发**：GitHub上涌现了大量社区MCP Server，覆盖GitHub、Slack、SQL数据库、文件系统、REST API等领域
- **企业采用**：多家企业在内部知识库和开发工作流中引入MCP
- **对标LSP**：MCP设计借鉴了Language Server Protocol（LSP）的成功经验，后者在编程语言领域验证了标准化协议的价值

---

## 三、A2A：多Agent系统间通信协议

### 3.1 为什么需要A2A

当多个AI Agent协同工作时，它们之间需要：
- **相互发现**：知道其他Agent能做什么
- **任务协作**：将子任务委派给合适的Agent
- **状态共享**：同步中间结果与上下文
- **结果汇总**：整合多Agent输出

没有标准协议的情况下，每个Agent系统都需要为其他系统编写定制化适配器。

### 3.2 Google A2A协议（2025年4月）

2025年4月，Google正式发布**A2A（Agent to Agent）协议**，作为与MCP并列的开放标准：

| 协议 | 定位 | 层级 |
|------|------|------|
| **MCP** | AI ↔ 工具/数据（垂直） | 单Agent与外部资源 |
| **A2A** | Agent ↔ Agent（水平） | 多Agent协作层 |

A2A的核心概念：
- **Agent Card**：每个Agent发布自己的能力描述（支持被发现）
- **Task Handoff**：将任务委托给其他Agent
- **Capability Exchange**：协商谁适合处理什么任务
- **结果回传**：被委托Agent完成后将结果返回给发起方

### 3.3 MCP与A2A的互补关系

业界共识：**MCP + A2A = Agent生态的基础设施栈**。MCP负责"单兵装备"（工具和数据），A2A负责"部队协同"（多Agent配合）。两者由不同厂商推动，但定位互补、不存在直接竞争。

---

## 四、主流框架中的Function Calling实现

### 4.1 LangChain Tools

LangChain提供了完整的工具抽象层：
- **`@tool` 装饰器**：将Python函数快速注册为LangChain工具
- **ToolKit**：工具包概念（如SQLDatabaseToolkit）
- **OpenAI Functions**：原生支持OpenAI格式的Function Calling
- **ReAct Agent**：通过思考-行动-观察循环调用工具
- **OpenAI Tools Agent**：专门处理并行函数调用

```python
from langchain.tools import tool

@tool
def search_database(query: str) -> str:
    """Execute SQL query against the analytics database."""
    # actual implementation
    return execute_sql(query)
```

### 4.2 LlamaIndex Function Calling

LlamaIndex在数据索引和检索领域有独特优势：
- **`FunctionCallingProgram`**：支持单函数和并行函数调用
- **结构化数据提取**：直接从非结构化文本中提取符合Schema的数据
- **与LangChain集成**：LlamaIndex的工具可无缝桥接到LangChain生态

### 4.3 OpenAI Assistants API & ChatGPT Plugins

- **Assistants API**：提供Code Interpreter、File Search、Function Calling三大工具
- **ChatGPT Plugins**：以Function Calling为底层，实现Web搜索、代码执行、第三方API调用
- **GPT Store**：基于插件生态构建的AI应用 marketplace

---

## 五、Function Calling评测基准

### 5.1 API-Bank Benchmark

由阿里巴巴DAMO-ConvAI团队于2023年发布，是**首个系统性评估工具增强LLM的基准**：
- **53个API工具**：涵盖搜索、天气、股票、数据库等领域
- **264个对话场景**：测试Agent的规划与推理能力
- **314个工具使用场景**：覆盖单步到多步工具链
- **753次API调用**：用于端到端评测
- 评测指标：API调用准确率、意图识别准确率、工具链规划成功率

### 5.2 BIG-bench Function Calling任务

BIG-bench中包含专门的Function Calling评估集，测试模型：
- 从自然语言指令中识别需要调用的工具
- 正确填充工具参数
- 处理工具返回的嵌套结果

### 5.3 ToolBench（2023）

清华&微软发布的工具调用评测，评估GPT-4等模型在工具选择、参数生成、执行效果上的综合能力。

---

## 六、实际应用案例

| 场景 | 具体实现 | 代表案例 |
|------|---------|---------|
| **数据库查询** | Text-to-SQL，通过Function Calling生成SQL并执行 | ChatGPT数据分析助手、Claude数据库助手 |
| **代码执行** | 模型生成代码片段，由沙箱执行并返回结果 | OpenAI Code Interpreter、Anthropic Claude |
| **API调用** | 将用户意图映射为REST/GraphQL调用 | ChatGPT Plugins中的第三方服务集成 |
| **文件系统操作** | MCP Server暴露文件读写接口 | Claude Desktop文件系统扩展 |
| **实时信息获取** | 搜索、天气、新闻等实时数据 | GPT-4实时搜索、Claude网络搜索 |

---

## 七、Function Calling安全性

### 7.1 主要安全威胁

1. **提示词注入（Prompt Injection）**：攻击者通过恶意输入让模型忽略安全边界，执行未授权的工具调用
2. **越权访问**：工具权限过大，导致模型能访问超出任务需要的资源
3. **工具参数污染**：恶意构造的工具返回值污染模型后续决策
4. **无限循环调用**：模型陷入反复调用同一工具的死循环

### 7.2 防护策略

- **权限最小化原则**：每个工具仅授予完成当前任务所必需的权限
- **输入验证**：对所有外部返回数据进行严格校验和清洗
- **工具执行隔离**：高危操作在沙箱/容器中执行
- **调用审计日志**：记录所有工具调用请求与返回值
- **模型层面防护**：OWASP LLM Top 10（2025版）将"工具操作滥用"列为重要威胁
- **PromptArmor等框架**：通过额外LLM Guardrail层检测注入攻击

---

## 八、总结与趋势

**Function Calling**已从"可选特性"演变为Agent系统的**基础设施**。标准化进程在2024-2025年显著加速：

- **MCP**为AI模型连接外部世界提供了统一协议，生态正处于爆发期
- **A2A**补全了多Agent协作层的空白，与MCP形成互补格局
- **评测体系**（API-Bank等）逐步成熟，为模型迭代提供量化依据
- **安全**问题随着Agent能力增强而愈发重要，防护框架正在快速完善

未来的竞争焦点将不在"是否支持Function Calling"，而在：**协议的互操作性**（MCP↔A2A↔厂商私有协议）、**工具生态的丰富程度**，以及**多Agent系统的可靠性与安全性**。

---

*参考来源：OpenAI Developer Docs, Anthropic MCP官方文档, Google A2A协议公告, API-Bank论文(arXiv:2304.08244), OWASP LLM Top 10 (2025), LangChain/LlamaIndex官方文档*
