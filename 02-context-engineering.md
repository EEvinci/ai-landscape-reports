# Context Engineering：AI Agent 的"世界观"设计学

## 【开篇场景】

2025 年 9 月，Anthropic 发布了一篇工程博客，标题直接是《Effective context engineering for AI agents》。同月，Andrej Karpathy 在 Twitter 上发了一条被广泛引用的观点："The context is the product."（上下文就是产品本身）。这两件事几乎同时发生，标志着一个新工程学科的正式登场：**Context Engineering（上下文工程）**——不再是调调 Prompt 那么简单，而是系统性地设计 AI 的"世界观"。

## 【核心是什么】

Context Engineering 是设计、构建和优化动态自动化系统的学科，核心目标是：在正确的时间、以正确的格式、为 AI Agent 提供正确的信息和工具。

这和 Prompt Engineering 的区别是什么？Prompt Engineering 优化的是"人给 AI 说了什么"，Context Engineering 优化的是"AI 思考时看到了什么"。一个是对话技巧，一个是工程系统。一个优秀的 Context Engineering 系统，可以让同一个 LLM 在不同任务中表现出截然不同的专业水平。

## 【关键原理】

把 LLM 想象成一个记忆力极强的实习生。Context Engineering 就是给他准备一个精心组织的工位——桌上只有跟当前任务相关的资料，其他干扰全部清除。

**第一层：上下文窗口管理**

LLM 的上下文窗口是根本性约束。2025 年，一个 200K token 的上下文窗口看似很大，但一个真实项目可能包含数百万行代码和文档。Context Engineering 的核心任务之一是**选择性检索**：只把最相关的内容注入上下文，避免"淹没在噪音里"。

Anthropic 在 2024 年 9 月提出了"Contextual Retrieval"方法——不只检索文档片段，还为每个片段附加描述性上下文（chunk inscription），让模型理解"这个片段在整本书/项目中的位置和意义"。实验显示，配合 reranker 重排序后，检索准确率提升了 30–50%。

**第二层：工具与信息路由**

Context Engineering 决定了一个 Agent"何时调用什么工具"以及"把工具返回结果放在上下文的什么位置"。这不是写一条 instruction 能解决的，而是需要设计一套动态路由机制。

**第三层：上下文质量控制**

Context Engineering 领域流传一句话："Garbage in, garbage out"（垃圾进、垃圾出）。但更准确的说法是：**上下文的质量比数量更重要**。给 AI 100 页相关文档，不如给 AI 10 页高度相关、格式清晰、有结构组织的文档。

## 【真实案例】

**案例 1：Manus 的多 Agent 调度系统**
2025 年 3 月，Manus 以"全球首个通用 AI Agent"爆红出圈。其核心技术之一就是精细的 Context Engineering——每个子 Agent 在启动时获得的上下文是经过"裁剪"和"增强"的：裁剪掉无关信息，增强任务相关的背景知识。这使得每个 Agent 即使在同一个大任务中，也能保持专注而不被全局信息压垮。

**案例 2：Context7 MCP Server**
Context7 是 2025 年快速崛起的 MCP Server 之一，专门提供实时技术文档检索。当 Agent 编写 React 代码时，Context7 自动注入最新的 React 19 API 文档；当 Agent 调试 Kubernetes 问题时，Context7 注入相关版本的配置参考。无需人工干预，Context Engineering 全自动完成。

**案例 3：Anthropic 的多 Agent 研究系统**
2025 年 6 月，Anthropic 在博客中透露了他们如何构建内部多 Agent 研究系统。关键设计是：每个 Agent 的上下文是"任务特定的"——规划 Agent 看到的是研究目标+进度追踪；执行 Agent 看到的是具体任务+所需代码片段；审核 Agent 看到的是输出+评估标准。三个 Agent 互不干扰，但通过标准化的中间格式实现信息交换。

## 【趋势与机会】

**1–3 年判断：这是 2025–2026 年最值得投入的工程能力。**

Context Engineering 之所以重要，因为它是目前投入产出比最高的优化手段——换一个大模型可能要花数百万美元，而优化上下文设计可能只需要几周工程时间，就能让现有模型表现提升一个档次。

**机会点：**
- **RAG 2.0 基础设施**：从简单的向量检索升级为"上下文质量优化"平台，加入意图识别、动态上下文压缩、相关性评分
- **Context Monitoring Tools**：实时监控 Agent 上下文的"健康度"，发现上下文漂移（context drift）时自动干预
- **垂直领域 Context 库**：为金融、医疗、法律等高价值领域构建专用上下文模板和知识组织方式

**值得关注的信号：** Anthropic 的工程博客在 2025 年密集发布了多篇与上下文相关的文章，从 Contextual Retrieval 到 Context Engineering，脉络清晰。这不是偶然——这是行业领头羊在告诉你：下一个战场不在模型层，在上下文层。
