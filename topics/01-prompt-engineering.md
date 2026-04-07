# 01. Prompt Engineering（提示词工程）

> 研究时间：2026-04-08 | 研究者：Kiko Sub-Agent #01 | 运行时长：1分56秒

---

## 核心技巧（2024-2025 最新）

### 1. Chain-of-Thought（CoT）— 思维链提示
- **原理**：引导模型逐步推理，而非直接输出答案
- **效果**：基础 CoT 提升 15-20%，结构化 CoT 提升 25-35%
- **变体**：
  - **Zero-shot CoT**：附加"Let's think step by step"触发自我推理
  - **Auto-CoT**：自动生成多样化推理链，无需手工设计示例
  - **Complexity-Based Prompting**：优先选择最复杂的示例进行推理采样

### 2. Tree-of-Thought（ToT）— 思维树
- **原理**：并行探索多条推理路径，模型自我评估各分支进展
- **效果**：比 CoT 提升 40-50%
- **适用**：复杂决策、创意生成、多方案评估

### 3. ReAct（Reasoning + Acting）
- **原理**：交替进行推理与外部行动（如搜索、API调用），让模型在真实环境中验证假设
- **优势**：解决幻觉问题，增强工具使用能力
- **代表工具**：Toolformer、Chameleon、GPT4Tools、Gorilla

### 4. Reflexion / Self-Refine
- **原理**：语言 agent 通过口头强化学习反思错误并纠正
- **流程**：执行 → 自我评估 → 生成口头反思 → 改进策略 → 重新执行
- **突破**：显著提升长序列任务的准确率

### 5. Program-Aided Language Models（PALs / PoTs）
- **原理**：将推理过程用 Python 代码表达，执行代码得出最终结果
- **优势**：解决"推理正确但计算错误"的问题

### 6. Least-to-Most Prompting
- **原理**：将复杂问题分解为从简到难的子问题，顺序求解并累积上下文
- **适用**：多步骤推理、长任务拆解

### 7. Self-Consistency（自洽性）
- **原理**：生成多条推理链，取众数答案
- **优势**：推理错误通常不一致，正确推理更稳定 → 有效过滤异常错误路径

### 8. Context Engineering（上下文工程）— 2025 新趋势
- **核心理念**（Andrej Karpathy 提出）：填充 context window 的艺术，而非仅优化 prompt 本身
- **七大组件**：System Instructions、User Context、Memory Management、RAG、Tool Integration、Output Formatting、Evaluation
- **关键数据**：掌握上下文工程的企业 ROI 提升 340%；78% 的 AI 项目失败源于人机通信问题，而非技术限制

---

## 主流框架对比

| 框架 | 核心结构 | 适用场景 | 特点 |
|------|---------|---------|------|
| **CRISP** | Context → Role → Instruction → Steps → Parameters | 通用任务、结构化输出 | 简洁实用，业界广泛采用 |
| **ICIO** | Instruction → Context → Input → Output | API 调用、产品集成 | 四步标准化，适合开发 |
| **CRISPE** | Context + Role + Instruction + Scenario + Persona + Extras | 深度角色扮演，专业咨询 | 最完整，适合复杂角色模拟 |
| **CoT 模板** | Problem → Steps(1-N) → Therefore | 数学、逻辑推理任务 | 强制多步思考 |
| **ToT 结构** | Problem → Branch A/B → Assessment → Best Path | 开放式决策、方案对比 | 并行探索，动态回溯 |
| **Context Engineering** | System + User Context + Memory + RAG + Tools | 生产级多轮对话系统 | 2025主流，工程化完整 |

**框架选用建议**：
- 日常对话/快速任务 → CRISP
- 产品集成/API → ICIO
- 需要深度角色扮演 → CRISPE
- 推理/分析类任务 → CoT / ToT
- 生产级 Agent 系统 → Context Engineering 七组件

---

## 最新研究进展

### 里程碑论文

1. **The Prompt Report: A Systematic Survey**（arXiv:2406.06608，2024.06，2025.02 更新）
   - 31 位作者，系统分析了 1,565 篇论文
   - 建立了 58 种基于文本的提示技术分类法
   - 覆盖 33 个术语词汇，建立了统一术语体系
   - 被引用超过 900 次

2. **Auto-CoT**（arXiv:2210.03493）
   - 自动生成多样化推理链，无需人工设计 CoT 示例
   - 匹配或超越手工 CoT 效果

3. **Decomposed Prompting (DecomP)**（arXiv:2210.02406）
   - 将复杂任务分解为多个子任务，每个子任务由专门 prompt 或模型处理
   - 模块化可优化、可替换

4. **Hypotheses-to-Theories (HtT)**（arXiv:10.07064）
   - 模拟科学发现流程：归纳阶段生成规则库 → 演绎阶段应用规则
   - 规则可在不同模型和问题形式间迁移

### 2024-2025 关键趋势
- **从 Prompt Engineering → Context Engineering**：重心从写好单条 prompt 转向管理整个 context window
- **多模态 CoT**：将思维链扩展到图像+文本的联合推理
- **Active-Prompt**：基于不确定性主动选择最有价值的示例标注
- **Skeleton-of-Thought (SoT)**：先生成大纲 → 并行填充细节，降低延迟
- **Chain of Density (CoD)**：迭代压缩摘要，在固定长度内最大化信息密度

---

## 实战建议

**1. 优先使用 Few-Shot + CoT，而非盲目追求 Zero-Shot**
Few-Shot 示例选取得当时可提升 23% 准确率。示例应展示推理模式，而非简单的输入-输出对。"Let's think step by step"几乎免费，却能带来 15-20% 的推理提升。

**2. 根据任务类型选择框架**
推理型任务 → CoT / ToT；角色扮演 → CRISPE；产品集成 → ICIO；生产级系统 → Context Engineering 七组件。不要一套 prompt 打天下。

**3. 用 Self-Consistency 换取关键任务可靠性**
高风险决策场景，多采样几条推理路径取众数。牺牲 token 成本，换取错误过滤。自验证（让模型检查自己的答案）也是高性价比的实战技巧。

**4. 上下文管理是 2025 年的核心竞争力**
- "Lost in the Middle"：重要信息放开头或结尾，中间容易被忽略
- 无关上下文会严重干扰模型表现
- 长对话场景：定期摘要旧上下文，保留关键事实与偏好
- 学会用 RAG 精准注入相关知识，而非堆砌上下文

**5. 构建反馈迭代闭环**
Prompt 工程本质上是迭代过程。初始 prompt → 评估输出 → 分析失败原因 → 针对性修改 → 重新评估。A/B 测试不同变体，保留 prompt 版本记录，逐步积累最优模板库。

---

## 推荐资源

### 必读指南
- [Prompt Engineering Guide](https://promptingguide.ai/) — 最完整的提示技术索引，覆盖 18+ 种核心技术
- [The Prompt Report（arXiv:2406.06608）](https://arxiv.org/abs/2406.06608) — 学术最全面综述，58 种技术分类，1,565 篇论文分析
- [Google Prompt Engineering Whitepaper（2024）](https://zhuanlan.zhihu.com/p/1916161218042918259) — Google 官方白皮书

### 核心论文
- "Chain-of-Thought Prompting Elicits Reasoning in LLMs" — CoT 开山之作
- "ReAct: Synergizing Reasoning and Acting in Language Models"（arXiv:2210.03629）
- "Tree of Thoughts: Deliberate Problem Solving with Large Language Models"
- "Reflexion: Language Agents with Verbal Reinforcement Learning"
- "Self-Consistency Improves Chain of Thought Reasoning in Language Models"

### 实战工具
- **Vellum.ai** — Prompt 版本管理与 A/B 测试平台
- **LangSmith** — LLM 应用监控与 prompt 评估
- **Anthropic Cookbook** — Claude 高阶使用 cookbook

### 社区与更新
- **DAIR.AI GitHub**（github.com/dair-ai/Prompt-Engineering-Guide）— 持续更新的开源指南
- **Hugging Face Gradio Demo** — 各技术的交互式演示

---

> **一句话总结**：2025 年的 Prompt Engineering 已从"写好一句话"进化为"管理整个上下文系统"。掌握 Context Engineering + CoT/ToT + Self-Consistency 组合，是当前 AI 应用开发者的核心竞争力。
