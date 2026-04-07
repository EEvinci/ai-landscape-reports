# 11. AI Reasoning & Test-time Compute

> 研究时间：2026-04-08 | 研究范围：2024–2025 年核心进展

---

## 核心概念

### 什么是 Test-time Compute（测试时计算）

传统 LLM 的推理发生在**训练时**——模型在预训练阶段通过海量数据学习模式，推理时只需一次前向传播即可输出答案。Test-time Compute（推理时计算扩展）则打破了这一范式：允许模型在**推理阶段**动态分配更多计算资源，通过更长的思考过程来提升答案质量。

其核心洞察是：**并非所有问题都需要同等难度**。简单问题（如常识问答）可以在少量 token 内解决，而复杂问题（如数学证明、代码调试）需要模型反复试探、回溯、验证。Test-time Compute 的核心思想是：让模型自行判断何时需要深入思考，并为之分配更多计算预算。

这一范式与**推理模型（Reasoning Model）**的兴起密切相关——2024 年 9 月 OpenAI 发布 o1 系列，标志着 AI 推理从"外部提示技巧"向"模型内在核心能力"的根本性转变。

### 核心术语

| 术语 | 含义 |
|------|------|
| **Chain-of-Thought (CoT)** | 思维链——让模型显式输出逐步推理过程的技术 |
| **Test-time Compute Scaling** | 在推理阶段通过增加计算量（如更多 token、更长思考）提升性能 |
| **Process Reward Model (PRM)** | 过程奖励模型——对推理的每一步而非最终答案打分 |
| **GRPO (Group Relative Policy Optimization)** | DeepSeek 提出的 RL 算法，用于推理模型训练 |
| **Self-Consistency** | 自洽性——对多条推理路径采样，选择最一致的答案 |
| **Tree-of-Thoughts (ToT)** | 思维树——在推理过程中维护多条可能的思考路径 |
| **Distillation（蒸馏）** | 将大模型的推理能力迁移到小模型的技术 |

---

## 2024–2025 重要模型发布

### 1. OpenAI o1 系列（2024 年 9 月 12 日）

**技术原理：**
- 核心突破：将 Chain-of-Thought 从"外部提示工程技巧"转变为**模型内在的、强化学习习得的核心能力**
- 引入"推理令牌（Reasoning Tokens）"——模型在生成最终答案前，生成大量不可见的内部思考 token
- 通过大规模强化学习训练，模型学会：识别错误、回溯修正、将复杂步骤分解、尝试替代方案
- 思维链对用户**隐藏**（仅在 ChatGPT 中展示摘要），出于安全考量和商业保护

**核心基准测试结果：**

| 基准 | 指标 | o1 | GPT-4o |
|------|------|-----|--------|
| AIME 2024（美国数学邀请赛） | pass@1 | **74%** | 12% |
| AIME 2024 | consensus@64 | **83.3%** | 13.4% |
| MATH | pass@1 | **94.8%** | 60.3% |
| GPQA Diamond（PhD 级科学） | pass@1 | **77.3%** | 50.6% |
| Codeforces | Elo 评分 | **1,807** | 808 |
| Codeforces | 百分位 | **89th** | 11th |
| MMLU（54/57 子类） | pass@1 | **90.8%** | 88.0% |
| MMMU（多模态） | pass@1 | **78.2%** | 69.1% |

**安全表现：**
- 有害提示安全完成率：99.5%（GPT-4o 为 99.0%）
- 越狱攻击抵御：Good@0.1 StrongREJECT 达 84%（GPT-4o 仅为 22%）
- 幻觉减少（相比 GPT-4o 和 GPT-4o mini）

**o1 的关键意义：**
- 首个在 GPQA 基准上**超越人类 PhD 专家**的模型
- 性能随训练时计算（更多 RL 训练）和测试时计算（更多思考时间）**持续提升**
- 改变了 AI 能力的边界——在需要严格逻辑、低容错率的领域（数学、编程、科学）实现质的飞跃

**o1-mini（同期发布）：**
- 更小、更快的推理模型
- 聚焦 STEM 推理能力，价格显著低于 o1-preview
- 输出 token 上限更高（65,536 vs 32,768）

---

### 2. DeepSeek-R1（2025 年 1 月 20 日）

**DeepSeek 的标志性贡献在于：证明了推理能力可以通过纯强化学习激励，无需人工标注的推理轨迹。**

DeepSeek-R1 于 2025 年 1 月发布，同年登上《Nature》封面，核心价值是**开源 + 纯 RL 路径**。

#### 两个核心模型

**DeepSeek-R1-Zero（纯 RL 路径）：**
- 完全不使用监督微调（SFT），仅依赖强化学习
- 基于 DeepSeek-V3-Base，在思维链推理任务上涌现出强大能力
- 自发产生"反思（Reflect）"行为——模型会重新审视自己的初始方法
- 被称为 DeepSeek 的"Aha Moment"——通过 RL 自然涌现，非人工编程

**DeepSeek-R1（冷启动 + 多阶段 RL）：**
- 引入"冷启动数据"（数千条高质量长思维链示例），解决可读性和语言混合问题
- 完整训练管线：冷启动 SFT → 推理 RL → 拒绝采样 + SFT → 全场景 RL

#### 训练管线详解

```
Stage 1: 冷启动（Cold Start）
  └── 用长思维链数据微调 DeepSeek-V3-Base，改善可读性

Stage 2: 推理导向 RL
  └── 大规模 RL，规则奖励（答案正确性 + 语言一致性）

Stage 3: 拒绝采样 + SFT
  └── 推理数据：60 万条（过滤正确答案）
  └── 非推理数据：20 万条
  └── 2 个 epoch 微调

Stage 4: 全场景 RL
  └── 规则奖励（推理） + 奖励模型（通用任务）
  └── 聚焦有用性和无害性
```

#### 基准测试对比

| 模型 | AIME 2024 Pass@1 | AIME 2024 Majority Vote | MATH-500 | 备注 |
|------|-----------------|------------------------|-----------|------|
| DeepSeek-R1-Zero | 71.0% | 86.7% | — | 纯 RL |
| DeepSeek-R1 | 79.8% | — | 96.3% | 冷启动 + RL |
| OpenAI-o1-1212 | — | 86.6% | — | 对标基准 |
| GPT-4o | 12% | — | — | 基线 |

#### DeepSeek-R1 蒸馏系列

DeepSeek 将推理能力蒸馏至小模型，效果远超同尺寸模型从零 RL 训练：

| 蒸馏模型 | AIME 2024 | MATH-500 | GPQA |
|---------|-----------|----------|------|
| DeepSeek-R1-Distill-Qwen-1.5B | 28.9% | 83.9% | — |
| DeepSeek-R1-Distill-Qwen-7B | 55.5% | — | — |
| DeepSeek-R1-Distill-Qwen-32B | **72.6%** | 94.3% | 73.3% |

**关键发现：蒸馏远优于小模型从头 RL。** DeepSeek-R1-Distill-Qwen-32B（72.6%）远超 RL 训练的 DeepSeek-R1-Zero-Qwen-32B，证明**强基座模型 + 蒸馏**比小模型独立 RL 更高效。

#### 局限性
- 函数调用、多轮对话、角色扮演弱于 DeepSeek-V3
- 非中英文场景存在语言混合
- 少样本提示反而降低性能，建议零样本
- 软件工程能力提升有限（评估耗时过长）

---

### 3. Qwen QwQ-32B（2025 年 3 月 6 日）

阿里云 Qwen 团队发布的开源推理模型，32B 参数，在多项基准上可与 DeepSeek-R1 和 o1-mini 竞争。

**核心特点：**
- 基于大规模连续强化学习（Continuous RL Scaling）训练
- 使用基于规则的奖励信号（数学、代码正确性）
- 在 Hugging Face 和 ModelScope 开源（Apache 2.0 许可），消费级显卡即可运行

**性能亮点：**
- 数学推理：通过逻辑链分解实现复杂问题求解
- 编程能力：代码生成与调试优化双路径
- 通用智能：整合工具调用与环境反馈机制
- 性能远超同等参数量的其他模型，可比肩更大参数的闭源模型

**与 DeepSeek-R1 的方法差异：** QwQ-32B 更强调训练过程中的奖励信号设计优化，DeepSeek-R1 则强调纯 RL 的能力涌现。两者共同验证了 RL + 推理链是提升模型能力的有效路径。

---

### 4. 其他重要模型动态

- **o3 系列（2024 年 12 月后）**：OpenAI 继续迭代 o 系列，o3 在 ARC-AGI 等前沿基准上进一步刷新纪录，展现 Test-time Compute 的 Scaling 潜力
- **Google Gemini 推理增强**：Gemini 系列在推理能力上持续追赶，整合Thinking模式
- **Kimi k0-math（国内）**：月之暗面发布数学推理模型，强化长思维链能力
- **Marco-o1 系列**：阿里巴巴发布专注于推理的新模型系列，探索更宽的蒸馏瓶颈

---

## 技术进展

### 1. Chain-of-Thought 的范式演进

Chain-of-Thought 的发展经历了三个阶段：

**第一阶段：外部提示工程（2022–2023）**
- 2022 年 Wei et al. 证明：仅需在 prompt 中加入"let's think step by step"，即可显著提升 LLM 的推理能力
- 核心机制：将复杂问题分解为逐步子问题，降低每次推理的难度
- **局限性：** 依赖人工设计提示，效果不稳定

**第二阶段：自洽性解码（2023）**
- 2023 年 Wang et al. 提出 Self-Consistency
- 核心思想：从贪婪解码改为多样本采样，生成多条推理路径，选择最一致的答案
- 显著提升 CoT 性能（通常 4–16 个样本最佳）
- **关键洞察：** 复杂问题通常允许多条正确路径，一致性高的答案可信度更高

**第三阶段：内部化推理能力（2024 至今）**
- OpenAI o1 将 CoT 从外部提示转变为**模型内在的 RL 习得能力**
- 模型不再需要"step by step"提示引导，而是自发进行多步推理
- 推理链对用户隐藏（Hidden CoT），仅展示最终答案
- 训练信号来自强化学习，而非人工设计的提示模板

**ACL 2024 综述分类（Chu et al., 2024）：**
CoT 研究可沿三条主线展开：
1. **提示驱动方法**：Few-shot CoT、Zero-shot CoT、自动 CoT
2. **推理策略方法**：Self-Consistency、Tree-of-Thoughts、Reasoning瑶 agent
3. **训练增强方法**：CoT 微调、RL + CoT、蒸馏

---

### 2. Tree-of-Thoughts（ToT）及其扩展

Tree-of-Thoughts 是 CoT 的重要扩展——不是单链推理，而是维护多条并行的推理路径。

**核心思想：**
- 在每个推理节点，模型生成多个可能的下一思考步骤
- 对每条路径进行评估（使用语言模型自评或外部奖励）
- 剪枝低质量路径，继续扩展有前景的分支
- 最终选择最优路径

**相关进展：**
- **Chain of Preference Optimization (CPO, NeurIPS 2024)**：利用 ToT 自我推理过程生成监督信号，增强 CoT 推理质量
- ToT 与 PRM 的结合是当前热点——用过程奖励模型评估每个推理分支，而非依赖最终答案

---

### 3. Test-time Compute Scaling 的核心研究

Test-time Compute Scaling 的核心理论来自 OpenAI 的实验观察：**o1 的性能随测试时计算量（思考 token 数量）持续提升**，且这一趋势没有明显的饱和点。

**Scaling 的两条路径：**

**路径 A：问题难度的自适应分配**
- 模型自动判断问题难度，为简单问题分配少量 token，为复杂问题分配更多推理
- Best-of-N 采样：对同一问题生成 N 个答案，选择最佳者
- 代价：N 倍推理成本
- 收益：简单问题几乎不增加成本，难问题质量显著提升

**路径 B：推理深度的强制扩展**
- 通过 API 参数（如 o1 的 `max_completion_tokens`）强制模型投入更多思考
- Beam Search 变体：在推理空间中进行定向搜索
- 适合场景：已知问题较难，需要强制深入

**与 Pretraining Scaling 的对比：**
- Pretraining Scaling（训练时扩展）：性能随训练 token 数量幂律增长，但边际收益递减
- Test-time Compute Scaling：对于推理任务，边际收益递减更慢——增加推理时间可以不断发现新错误并纠正
- **理论意义：** 为 AI 能力提升开辟了新的 Scaling 维度，不完全依赖更大的预训练

**Google DeepMind 的相关工作（AlphaProof）：**
- 结合形式化数学证明系统（Lean）与强化学习
- 在 IMO（国际数学奥林匹克）题目上达到银牌水平
- 验证了形式化验证 + RL 是另一条推理提升路径

---

## 推理效率优化

推理能力的提升往往伴随计算成本的急剧增加。效率优化成为工程落地的核心挑战。

### 1. 知识蒸馏（Distillation）：让小模型也能深度思考

DeepSeek 的核心发现证明了**蒸馏优于小模型从头 RL**：
- 蒸馏的本质是将大模型的"思考过程"（长 CoT 推理轨迹）作为训练数据
- 800k 蒸馏样本训练出的 Qwen-32B，超越 RL 训练的同尺寸模型

**最新蒸馏研究：**

| 研究 | 核心贡献 |
|------|---------|
| **Skill-Aware Distillation** | 以技能为中心的蒸馏，让小推理模型更稳健、可解释 |
| **Marco-o1 v2** | 发现蒸馏存在"瓶颈"——识别蒸馏数据中的质量边界 |
| **DualDistill (EMNLP 2025)** | 双重蒸馏，结合自蒸馏进一步增强小模型推理能力 |
| **Efficient Long CoT in Small Models** | 解决小模型（~7B）学习长推理链的挑战 |

**小模型推理能力一览（基于蒸馏）：**

| 模型 | 参数量 | AIME 2024 | MATH-500 |
|------|--------|-----------|----------|
| DeepSeek-R1-Distill-Qwen-1.5B | 1.5B | 28.9% | 83.9% |
| DeepSeek-R1-Distill-Qwen-7B | 7B | 55.5% | — |
| DeepSeek-R1-Distill-Qwen-32B | 32B | 72.6% | 94.3% |
| QwQ-32B | 32B | ~72% | ~95% |

---

### 2. 推理效率优化技术

**ConCISE（2024）：**
- 核心思想：Confidence-guided Compression in Step-by-step Efficient Reasoning
- 在保持准确率不降的前提下，将推理过程 token 减少约 50%
- 关键方法：对置信度高的中间步骤进行压缩/跳过

**PRM 引导的早停（Early Stopping）：**
- 使用 Process Reward Model 在推理过程中实时评估每个步骤
- 一旦模型在当前路径上置信度足够高（即 PRM 打分高），立即停止继续推理
- 节省不必要的推理 token 开销

**自适应采样（Adaptive Sampling）：**
- 根据问题难度动态调整采样数量 N
- 简单问题：N=1（贪婪解码）
- 困难问题：N=32 或更多
- 难题识别：使用难度预测模型或 PRM 置信度

**Speculative Decoding：**
- 使用小型"草稿模型"快速生成候选 token
- 大型推理模型并行验证
- 适合推理场景：推理模型的思考 token 序列长，speculative decoding 可加速自回归生成

---

### 3. GRPO 算法详解

DeepSeek-R1 使用的 GRPO（Group Relative Policy Optimization）是对 PPO 的高效改进：

**PPO 的局限：** 需要单独训练一个判别器（Critic）网络来估算价值函数，计算开销大。

**GRPO 的创新：**
- 无需单独 Critic，在采样组内用相对排名替代绝对价值估计
- 对同一问题采样 G 个响应，基于奖励排序，计算相对优势
- 大幅降低训练成本，同时保持 RL 训练的有效性

```
GRPO 核心流程（简化）：
1. 对每个问题采样 G 个响应 {o1, o2, ..., oG}
2. 计算每个响应的奖励 {r1, r2, ..., rG}
3. 计算相对优势：Aj = (rj - mean(r)) / std(r)
4. 使用 PPO-style 裁剪目标更新策略
```

DeepSeek 验证了 GRPO 在推理任务上的有效性，为社区提供了低成本的 RL 推理训练方案。

---

## 最新研究

### 1. Process Reward Model（PRM）进展

**PRM vs ORM（结果奖励模型）：**
- ORM 仅在最终答案处打分，无法引导中间推理步骤
- PRM 对推理的**每一步**打分，可以引导模型选择更好的推理路径

**主流 PRM 方法：**

| 方法 | 机构 | 核心贡献 |
|------|------|---------|
| **Qwen2.5-Math-PRM** | 阿里 Qwen（2025.01） | 指出蒙特卡洛估计存在"对推错"问题，提出基于 Q-value 排序的改进 |
| **Process Reward Model with Q-value Rankings**（arXiv 2024） | — | 解决步骤间相互依赖导致的次优奖励分配问题 |
| **R-PRM**（EMNLP 2025） | — | 推理驱动的 PRM，利用更强 LLM 从有限标注生成种子数据 |
| **DreamPRM**（NeurIPS 2025） | — | 领域重加权 PRM，提升多模态推理的过程评估 |
| **ORM-to-PRM 迁移** | — | 将 ORM 的结果监督信号迁移用于 PRM 训练 |

**PRM 的核心挑战：**
1. **步骤标注成本高**：需要对每个推理步骤人工标注正确/错误
2. **蒙特卡洛估计偏差**：使用"后续步骤正确"的弱标签估计当前步骤价值
3. **步骤间依赖**：一个步骤正确与否取决于前面所有步骤的上下文
4. **可扩展性**：高质量 PRM 通常需要比 ORM 更多的训练数据

**PRM 与推理搜索的结合：**
PRM 最直接的应用是引导 Beam Search 或 Tree Search——在每个推理节点，用 PRM 评估所有候选步骤，选择得分最高者继续扩展。这比纯采样的 Self-Consistency 效率更高。

---

### 2. 可验证推理与形式化方法

随着推理能力的提升，**答案可验证性**成为关键瓶颈：
- 数学：可形式化证明（Lean、Coq、Isabelle）
- 代码：可执行测试
- 科学：可与实验数据对比

**AlphaProof（Google DeepMind）：**
- 将自然语言推理转化为形式化证明语言（Lean）
- 在 IMO 题目上达到银牌水平
- 证明了形式化验证 + RL 是数学推理的有效路径

**OpenAI 的安全对齐收益：**
- o1 的隐藏思维链为安全对齐提供了新的观察窗口——可以在思维层面检测有害意图，而非仅在输出层拦截
- 实验表明：o1 在有害内容越狱攻击上的抵御能力显著增强

---

### 3. 前沿研究方向

| 方向 | 描述 | 代表工作 |
|------|------|---------|
| **Long CoT in Small Models** | 让 7B 左右的小模型学习数千 token 的长推理链 | Efficient Long CoT (2025) |
| **Agentic Reasoning** | 推理模型 + 工具使用 + 环境反馈的闭环 | Real-Time Reasoning Agents (清华/上交/斯坦福 2025) |
| **Multimodal Reasoning** | 将推理能力扩展到视觉、音频等多模态输入 | MMMU, DreamPRM |
| **Self-Correction** | 模型在推理过程中主动发现并修正错误 | DeepSeek-R1-Zero 的"Aha Moment" |
| **DMLR** | 动态多模态交织推理，无需额外训练 | 清华/上海交大 (2025) |
| **ConCISE** | 推理过程压缩，保持准确率同时减少 token | 2024 |

---

### 4. 前沿基准测试进展

| 基准 | 描述 | 2023 水平 | 2025 水平 |
|------|------|---------|---------|
| AIME 2024 | 美国数学邀请赛 | ~12%（GPT-4） | **74%（o1）–79.8%（DeepSeek-R1）** |
| GPQA Diamond | PhD 级科学题 | ~50%（GPT-4） | **77.3%（o1）** |
| MATH-500 | 竞赛数学题 | ~60%（GPT-4o） | **94.8%（o1）** |
| Codeforces | 编程竞赛 | Elo 808（GPT-4o） | **Elo 1807（o1）** |
| ARC-AGI | 抽象推理 | 人类水平以下 | **接近人类水平（o3）** |

**o3 在 ARC-AGI 上的突破**具有标志性意义——该基准专门设计为人类可以轻松解决但传统 AI 难以通过的视觉推理任务，o3 的出现意味着推理模型开始触及真正的通用智能边界。

---

## 推荐资源

### 论文

| 论文 | 会议/期刊 | 链接 |
|------|---------|------|
| DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning | Nature 2025 | [arXiv](https://arxiv.org/abs/2501.12599) |
| Learning to Reason with LLMs | OpenAI 官方博客 | [openai.com](https://openai.com/index/learning-to-reason-with-llms/) |
| Chain-of-Thought Prompting Elicits Reasoning in Large Language Models | NeurIPS 2022 | [arXiv](https://arxiv.org/abs/2201.11903) |
| Self-Consistency Improves Chain of Thought Reasoning in Language Models | ICLR 2023 | [OpenReview](https://openreview.net/forum?id=1PL1NIMMrw) |
| Navigate through Enigmatic Labyrinth: A Survey of Chain of Thought Reasoning | ACL 2024 | [ACL Anthology](https://aclanthology.org/2024.acl-long.65/) |
| Chain of Preference Optimization | NeurIPS 2024 | [NeurIPS](https://proceedings.neurips.cc/paper/2024/file/00d80722b756de0166523a87805dd00f-Paper.pdf) |
| Process Reward Model with Q-value Rankings | arXiv 2024 | [arXiv](https://arxiv.org/abs/2410.11287) |
| QwQ-32B: Embracing the Power of Reinforcement Learning | Qwen 官方博客 | [qwen.ai](https://qwen.ai/blog?id=qwq-32b) |
| DualDistill: Reasoning Distillation in Small Models | EMNLP 2025 | [GitHub](https://github.com/StigLidu/DualDistill) |
| R-PRM: Reasoning-Driven Process Reward Modeling | EMNLP 2025 | [ACL Anthology](https://aclanthology.org/2025.emnlp-main.679/) |

### 开源模型

| 模型 | 链接 | 许可 |
|------|------|------|
| DeepSeek-R1 系列 | [HuggingFace](https://huggingface.co/deepseek-ai/DeepSeek-R1) | MIT / 部分模型开源 |
| DeepSeek-R1 蒸馏系列 | [HuggingFace](https://huggingface.co/deepseek-ai) | MIT |
| QwQ-32B | [HuggingFace](https://huggingface.co/Qwen/QwQ-32B) | Apache 2.0 |
| QwQ-32B-Preview | [ModelScope](https://www.modelscope.cn/models/Qwen/QwQ-32B-Preview) | Apache 2.0 |
| Awesome-Process-Reward-Models | [GitHub](https://github.com/RyanLiu112/Awesome-Process-Reward-Models) | — |
| CoT-Reasoning-Survey | [GitHub](https://github.com/zchuz/CoT-Reasoning-Survey) | — |

### 优质博客与解读

- [DeepSeek-R1 论文精读（中文）](https://arthurchiao.art/blog/deepseek-r1-paper-zh/) — ArthurChiao 翻译与解读
- [OpenAI o1 技术解析（知乎）](https://zhuanlan.zhihu.com/p/720181256) — 中文深度解析
- [DeepSeek-R1 知乎论文解读](https://zhuanlan.zhihu.com/p/24389442848) — GRPO 算法详解
- [Qwen2.5-Math-PRM 解读](https://zhuanlan.zhihu.com/p/18389486329) — PRM 蒙特卡洛估计问题分析
- [QwQ vs DeepSeek-R1 对比分析](https://zhuanlan.zhihu.com/p/28357125871) — 两种 RL 路径对比
- [大模型思维链技术原理](https://zhuanlan.zhihu.com/tardis/bd/art/629087587) — CoT 前世今生

### 延伸阅读方向

1. **RL 算法基础**：GRPO 是推理模型训练的核心，建议深入理解 PPO → GRPO 的演进逻辑
2. **形式化推理**：AlphaProof 展示了 Lean + RL 的潜力，是数学推理的重要方向
3. **推理效率**：ConCISE、Speculative Decoding 等方向仍在快速演进
4. **Agentic Reasoning**：推理 + 工具调用 + 反馈闭环，是 2025 年最具落地潜力的方向之一
