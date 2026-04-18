# MCP：Anthropic 发起的数据互联协议战争

## 【开篇场景】

2024 年 11 月 25 日，Anthropic 发布了 Model Context Protocol（MCP），一个开放标准，目标是让 AI 模型能标准地连接外部数据源和工具。消息发布当天，GitHub 上线了 MCP 官方仓库，Zed、Replit、Codeium、Sourcegraph 四家主流开发工具公司同时宣布支持。这不是 Anthropic 第一次发布技术标准，但这是第一次，整个行业如此迅速地站在了同一个协议旗帜下。

## 【核心是什么】

MCP 要解决的是一个结构性低效问题：**AI 模型和数据之间隔着一堵墙。**

每个企业有大量数据存在 Salesforce、Slack、GitHub、Google Drive、PostgreSQL……过去，每个 AI 助手要连接这些数据，都需要单独开发一个适配器。如果一家公司用 10 种不同的 AI 产品，每个产品都要连 10 个数据源，那就是 100 个需要维护的适配器——这叫"M×N 问题"。Anthropic 称之为"bridge-burning problem"（桥梁燃烧问题）。

MCP 的解法是：**用一套协议替代 M×N 个适配器**。开发者只需要实现一个 MCP Server（连接数据源），就能被所有支持 MCP 的 AI 应用使用。这和当年微软推出 LSP（Language Server Protocol）统一代码提示工具生态的思路完全一致。

## 【关键原理】

MCP 的架构极其清晰：三个核心组件，两类消息通道，三种资源类型。

**架构三组件：**
- **Host（宿主）**：发起连接的 AI 应用，如 Claude Desktop、ChatGPT
- **Client（客户端）**：嵌入 Host 内的连接器，负责与 Server 通信
- **Server（服务器）**：暴露数据源和工具的服务端，如 GitHub MCP Server

**三种 Server 提供的能力：**
- **Resources（资源）**：AI 读取用的数据——文件内容、数据库查询结果、API 响应
- **Prompts（提示模板）**：预定义的 Prompt 片段，供 AI 在特定场景使用
- **Tools（工具）**：AI 可调用的函数——执行代码、发送消息、操作文件

**Client 的反向能力（MCP 1.0 新增）：**
- **Sampling**：Server 可主动触发 AI 推理（用于复杂的 Agent 递归调用）
- **Roots**：Server 告诉 AI 文件系统的边界在哪里（安全边界）
- **Elicitation**：Server 可向用户请求额外信息

MCP 基于 JSON-RPC 2.0 构建，采用有状态的持久连接（而非无状态的 HTTP 请求），这使得 AI 在多轮对话中能保持上下文连贯性。

## 【真实案例】

**案例 1：Anthropic 快速构建 MCP Server 的能力**
2024 年 11 月发布时，Anthropic 展示了 Claude 3.5 Sonnet 可以"直接帮你写 MCP Server"——开发者用自然语言描述需求，Claude 即可生成完整的 MCP Server 实现代码。这降低了生态建设门槛：不需要深厚的基础设施经验，任何会用 Python 的开发者都能构建自己的 MCP Server。

**案例 2：Desktop Extensions 一键安装**
2025 年 6 月，Anthropic 推出 Claude Desktop 的 Desktop Extensions 功能——用户可以在 Claude Desktop 内一键安装社区贡献的 MCP Server。发布后 3 个月内，社区贡献的 MCP Server 数量从 20+ 增长到 200+。覆盖范围包括 Notion、Figma、Linear、Stripe 等主流 SaaS 工具。

**案例 3：企业级采纳**
Block（前 Square，CTO Dhanji R. Prasanna）和 Apollo 是最早的企业级采纳者。Block 在内部用 MCP 连接 Claude 与其交易数据平台，让 Claude 能够实时分析支付数据。Apollo 则用 MCP 让 AI 直接操作 CRM 系统，替代了大量人工的数据录入和报表生成工作。2025 年，Anthropic 将 MCP 捐赠给 Linux 基金会下的 Agentic AI Foundation 管理，标志着 MCP 已从"Anthropic 的项目"转变为行业公共基础设施。

**案例 4：代码执行与 MCP 的结合**
2025 年 11 月，Anthropic 发布博文《Code execution with MCP》，展示了用 MCP 构建高效 Agent 的新范式——Agent 不再需要通过复杂的 API 调用来执行代码，而是通过 MCP Tool 直接调用预配置的代码执行环境。实验数据显示，这种方式将代码执行类 Agent 的任务完成时间平均缩短了 **40%**，API 调用次数减少了 **60%**。

## 【趋势与机会】

**1–3 年判断：MCP 已经赢得协议战争，但"连接"之后还有更大的战场。**

截至 2025 年底，MCP 是 AI Agent 与外部工具连接领域的事实标准。Linux 基金会的背书消除了"这是 Anthropic 私有的、随时可能闭源"的顾虑，生态正在快速扩张。

**机会点：**
- **MCP Server 市场**：类似 Chrome 扩展商店，企业和开发者需要一个 marketplace 来发现、分发、评估 MCP Server 的质量
- **MCP 安全产品**：随着 MCP Server 数量爆炸，恶意 MCP Server 注入攻击开始出现——类似传统软件安全市场的 MCP 安全工具（扫描、审计、沙箱运行）是蓝海
- **垂直行业 MCP Hub**：金融、医疗、法律等行业的专用数据连接标准，一套协议搞定行业内的数据孤岛问题

**值得关注的信号：** 2026 年，OpenAI 和 Google 是否会采纳 MCP（而非继续用各自私有协议）将是关键观察点。如果头部厂商继续各自为战，MCP 的价值会大打折扣；如果他们加入，MCP 就真正成为了 AI 时代的"USB 接口"。
