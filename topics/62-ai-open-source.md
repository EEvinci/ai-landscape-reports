# 开源AI生态全景报告

**主题编号：** 62  
**报告名称：** AI-Open-Source（开源AI生态）  
**撰写日期：** 2026年4月  
**报告字数：** 约6000字

---

## 一、开源AI全景：规模、生态与格局

### 1.1 Hugging Face：从聊天机器人到AI界的"GitHub"

Hugging Face 已成为开源AI生态的核心枢纽。截至2024年9月，Hugging Face Hub 平台上的AI模型数量历史首次突破**100万个**，成为机器学习领域急速扩张的里程碑事件（Ars Technica, 2024）。这100万个模型涵盖了从BERT、GPT-2开源权重到LLaMA、Qwen等最新大模型的所有层级。

平台不仅托管模型，还提供：
- **模型托管与版本管理**：超过100万个模型条目
- **数据集仓库**：数十万个公开数据集
- **Spaces应用**：数千个基于Gradio/Streamlit的交互式AI演示
- **模型评测**：Open LLM Leaderboard等标准化评测平台
- **推理API**：Inference API支持快速调用

GitHub同样是开源AI生态的关键基础设施。根据GitHub Octoverse报告，AI相关项目的增长速度远超其他类别，Python已超越JavaScript成为GitHub上最受欢迎的编程语言，这在很大程度上由机器学习驱动。

### 1.2 开源 vs 闭源：核心维度对比

| 维度 | 开源AI | 闭源AI（OpenAI/Google/Anthropic） |
|------|--------|----------------------------------|
| **代表模型** | LLaMA 3.1、Mistral、Qwen2.5、DeepSeek-R1 | GPT-4o、Claude 3.5、Gemini 1.5 |
| **权重开放度** | 完整或部分开放权重 | 闭源，不公开权重 |
| **部署灵活性** | 任意环境（本地/云/边缘） | 依赖厂商API |
| **成本** | 推理硬件成本高，但无调用费用 | 按token付费，成本可控 |
| **定制能力** | 可全量微调、RLHF、二次训练 | 有限微调（GPTs等） |
| **透明度** | 训练数据、架构可审计 | 黑盒，不可审计 |
| **安全性** | 存在滥用风险，但可本地审计 | 可控，但依赖厂商 |
| **最新能力** | 通常落后6-18个月 | 最前沿 |
| **许可证** | 多种（Apache 2.0、MIT、Llama License等） | 专有 |

**开源AI的战略优势**在于：打破闭源垄断、为中小企业和个人开发者提供平权访问、推动AI安全研究透明化、以及催生繁荣的本地推理工具链。然而，闭源模型在推理效率、长上下文、多模态集成等方面仍保持领先。

---

## 二、开源大模型生态：从GPT-2到LLaMA3的演进图谱

### 2.1 开放权重的历史脉络

开源大语言模型的发展史，本质上是一部"开放边界逐步扩展"的历史：

**GPT-2（2019）**：OpenAI发布GPT-2时以"太危险"为由拒绝开源完整权重，仅公开部分小型版本。社区随后通过"复现"项目填补了这一空白，这成为开放权重运动的早期注脚。

**GPT-J / GPT-NeoX（2021）**：EleutherAI以"做OpenAI不想做的事"为使命，训练并开源了GPT-J（60亿参数）和GPT-NeoX（200亿参数），成为早期高质量开源大模型的标杆。

**LLaMA系列（2023-）**：Meta于2023年2月发布LLaMA，首次将"可在消费级硬件运行"的高性能语言模型带入开源世界。LLaMA-7B在MMLU等基准上接近GPT-3（1750亿参数），而体积缩小了约25倍。

**Mistral 7B（2023.9）**：法国初创公司Mistral AI发布的Mistral 7B在多项基准上超越LLaMA 2 13B，采用Sparse Mixture of Experts（SMoE）架构，效率极高。

**LLaMA 2（2023.7）**：Meta首次允许LLaMA 2用于商业用途（免费，但有月活用户限制），引爆了开源微调社区。Alpaca、Vicuna、Orca等微调变体大量涌现。

**Qwen系列（2023-）**：阿里巴巴开源通义千问（Qwen）系列，从Qwen-1.8B到Qwen-72B全线覆盖，中文能力领先，Apache 2.0许可证（部分型号）降低了商业使用门槛。

**LLaMA 3（2024.4）**：Meta发布LLaMA 3，8B和70B版本全面超越GPT-3.5，并在某些任务上接近GPT-4。LLaMA 3.1进一步扩展到405B参数，逼近闭源前沿。

**DeepSeek-R1（2025.1）**：中国公司深度求索（DeepSeek）发布DeepSeek-R1，以极低成本训练出与OpenAI o1性能相当的推理模型，采用MIT许可证开源，彻底改变了"开源=落后"的叙事。

### 2.2 主要开源模型家族一览

| 模型家族 | 机构 | 代表模型 | 许可证 | 特点 |
|----------|------|----------|--------|------|
| LLaMA | Meta | LLaMA 3.1 (8B-405B) | Llama License（定制） | 生态最庞大 |
| Mistral | Mistral AI | Mistral 7B, Mixtral 8x7B | Apache 2.0 | 效率高，SMoE |
| Qwen | 阿里云 | Qwen2.5 (0.5B-72B) | Apache 2.0（部分） | 中文领先 |
| Phi | Microsoft | Phi-3 (3.8B-14B) | MIT | 小而精，数据质量驱动 |
| Gemma | Google | Gemma 2 (2B-27B) | Gemma Terms | 轻量高性能 |
| ChatGLM | 智谱AI | ChatGLM3 (6B) | Apache 2.0 | 中文对话优化 |
| DeepSeek | 深度求索 | DeepSeek-R1 (671B) | MIT | 推理能力强 |
| Baichuan | 百川智能 | Baichuan2 (7B-13B) | Baichuan License | 中文开源 |

---

## 三、LLaMA生态深度：Meta的战略与开源的意义

### 3.1 Meta的开源战略

Meta是当前AI开源领域最重要的推手，其开源战略背后有清晰的商业逻辑：

**竞争维度**：Google、OpenAI、Anthropic三足鼎立的闭源格局对Meta不利。开源LLaMA系列能搅动竞争格局，迫使竞争对手在定价和服务上做出让步，同时让基于Meta生态的开发者获得更多选择。

**生态锁定**：开源模型虽然"免费"，但会带动PyTorch生态（Meta旗下）、云计算消耗和AI基础设施需求，形成间接商业价值。

**品牌与技术影响力**：LLaMA成为事实上的开源AI标准，吸引顶尖研究者参与Meta的AI研究生态。

**数据飞轮**：开源模型让Meta能获取大量来自开源社区的微调数据、RLHF信号和对齐研究成果。

### 3.2 许可证演进：从限制到开放

**LLaMA 2（2023年7月）**：首次允许商业使用，但有附加条件——月活超过7亿用户的商业应用需向Meta申请特别许可。这被称为"半开源"或"开放权重"（Open Weights）。

**LLaMA 3（2024年4月）**：延续LLaMA License，但注册流程更简便。Meta明确表示鼓励开源社区参与，同时保留了"可接受使用政策"（AUP）限制。

**LLaMA 3.1（2024年7月）**：405B参数的加入使LLaMA进入"前沿模型"级别，Meta同期发布了PurpleLlama安全工具包，体现"开源+安全"并行推进的策略。

**许可证争议**：OSI（开放源代码倡议组织）明确表示LLaMA License不是开源许可证，因为它对商业使用和用户规模做了限制。这一定性至今仍有争议。

### 3.3 开源对AI民主化的影响

开源LLaMA生态对AI民主化的贡献是深远的：

1. **降低门槛**：个人开发者可用消费级GPU（RTX 3090/4090）运行70B参数模型
2. **隐私保护**：数据无需上传到第三方服务器，在医疗、法律、金融等敏感领域尤为关键
3. **微调民主化**：LoRA、QLoRA等技术的普及使个性化微调从精英研究机构走向大众
4. **创新加速**：开源模型催生了数万种微调变体，覆盖多语言、代码、推理等垂直场景
5. **审计透明**：研究者可完整审查模型行为，有助于发现偏见和有害输出

---

## 四、开源AI工具链：繁荣的中间层生态

围绕开源大模型，一套完整的工具链生态已经形成：

### 4.1 推理引擎

| 工具 | GitHub Stars | 核心功能 |
|------|-------------|---------|
| **vLLM** | ~40k | PagedAttention，Tensor并行，高吞吐量推理 |
| **Ollama** | ~80k | 一键本地运行，支持模型库管理，兼容OpenAI API |
| **llama.cpp** | ~60k | C/C++实现，量化算法丰富，跨平台（苹果芯片优化） |
| **text-generation-inference (TGI)** | ~15k | Hugging Face官方推理服务器，支持Flash Attention |
| **DeepSpeed-MII** | ~12k | 微软出品，多节点推理优化 |
| **SGLang** | 新兴 | RadixAttention，支持长上下文和多模态 |

### 4.2 应用开发框架

**LangChain**（GitHub ~100k stars）：最流行的LLM应用开发框架，提供Prompt模板、Chain（链式调用）、Agent（自主代理）、Memory（记忆）等抽象层。支持接入几乎所有主流LLM API和本地模型。

**LlamaIndex**（原GPT-Index，约15k stars）：专注于知识增强检索（RAG，Retrieval-Augmented Generation），为本地知识库问答场景设计，是企业私有知识问答的事实标准。

**LangGraph**：LangChain团队推出的下一代图式Agent编排框架，支持复杂的多步骤推理循环。

**Semantic Kernel**：微软出品，面向企业应用的AI编排框架，与Azure深度集成。

### 4.3 微调工具

| 工具 | 核心能力 |
|------|---------|
| **LLaMA-Factory** | 统一微调框架，支持全参数/Lora/QLoRA，中文友好 |
| **Axolotl** | 多框架支持的微调配置管理 |
| **trl（Transformer Reinforcement Learning）** | Hugging Face官方RLHF工具包 |
| **DeepSpeed Chat** | 微软RLHF训练优化 |

### 4.4 模型部署与托管

**Hugging Face Inference Endpoints**：托管推理服务，企业级SLA，按需计费。

**Replicate**：云端模型托管与API服务，支持自定义推理容器。

**Modal**：Serverless GPU计算平台，适合间歇性推理负载。

---

## 五、开源AI风险：安全、滥用与治理困境

### 5.1 滥用风险的光谱

开源AI模型面临多层次的滥用风险：

**低门槛生物/化学武器设计**：MIT Technology Review等机构的研究表明，开源模型在生物知识方面的风险等级高于闭源模型，因为无法施加使用限制。LLaMA 2的早期版本中，研究者确实发现了潜在的生物威胁信息生成能力。

**大规模虚假信息生成**：开源模型可被用来生成个性化钓鱼邮件、深度伪造内容、舆论操纵文章，且无法溯源和管控。

**绕过安全对齐**：部分开源社区提供了"去对齐"（de-alignment）工具，可移除模型的安全限制。

**隐私泄露**：模型可能从训练数据中记忆敏感个人信息，本地部署后缺乏隐私保护机制。

### 5.2 安全治理机制

**RIST（Responsible AI Scaling）**：Meta提出的AI扩展责任框架，要求在大规模训练前完成安全评估和红队测试。

**PurpleLlama**：Meta开源的安全工具包，包括模型水印、对抗性测试工具和安全评估基准。

**MLCommons**：跨行业联盟，发布MLPerf基准、AI安全评估标准和模型卡片规范，推动AI安全评测的标准化。

**LAION & BigScience**：学术开源社区在数据治理方面的先行实践，建立了数据溯源和清洗的标准化流程。

**开放与监管的张力**：监管机构（如EU AI Act）对开源模型设置了豁免边界，但"全面开源"是否等同于"免除责任"仍存争议。批评者认为，开放权重让监管链条断裂；支持者则认为开源是审计透明的前提。

---

## 六、中国开源AI：本土力量崛起

### 6.1 阿里Qwen：国际化最成功的中文开源模型

阿里云通义千问（Qwen）是目前国际影响力最大的中国开源大模型家族：

- **Qwen1.5/Qwen2**：全系列开源（0.5B到72B），Apache 2.0许可证（部分型号），支持128K上下文
- **Qwen2.5**：在代码生成、数学推理等任务上与LLaMA 3匹敌
- **Qwen-Coder**：专注文档级代码生成，在HumanEval上表现优异
- **生态覆盖**：ModelScope（魔搭）、Hugging Face双平台发布，国际开发者社区活跃
- **技术报告**：《Qwen Technical Report》（arXiv:2309.16609）

### 6.2 智谱ChatGLM：学术驱动的中文对话模型

智谱AI（Zhipu AI）以学术背景驱动，ChatGLM系列在中国高校和科研机构中应用广泛：

- **ChatGLM3-6B**：采用Prefix LM架构，中文对话流畅，Apache 2.0许可
- **GLM-4**：更大规模闭源版本，对标GPT-4 Turbo
- 智谱同时维护**ModelScope社区**（国内最大模型开源平台之一，与Hugging Face对应）

### 6.3 深度求索DeepSeek：开源推理的破局者

DeepSeek于2025年1月20日发布**DeepSeek-R1**，震惊AI行业：

- **性能对标OpenAI o1**：在MATH、AIME等推理基准上与o1旗鼓相当
- **极低成本训练**：据报告仅用约600万美元训练成本（相比GPT-4的约1亿美元）
- **MIT许可证**：完全开源，可自由商用，彻底改变了开源推理模型的格局
- **DeepSeek-R1-0528（2025.5）**：进一步提升，在GPQA Diamond等基准上超越GPT-4o
- 深度求索同时开源了**DeepSeek-V3**（MoE架构，671B参数），采用多头潜在注意力（MLA）和DeepSeekMoE架构

### 6.4 其他重要玩家

- **百度ERNIE Bot**：以ERNIE 4.0为核心，但商业策略以API调用为主，开放权重有限
- **讯飞星火**：科大讯飞开源了部分星火系列模型，但商业版能力更强
- **百川智能**：Baichuan 2开源7B/13B模型，定位中英双语商业应用
- **小米MiMo**：专注推理优化的开源模型
- **Yi系列**（零一万物）：李开复团队发布的Yi-34B等模型，在Hugging Face上获得大量关注

### 6.5 中文AI开源生态的特点

| 特点 | 说明 |
|------|------|
| **平台互补** | ModelScope（魔搭）+ Hugging Face双轨并行 |
| **中文优先** | 预训练中文数据比例高，OCR/拼音处理更优 |
| **监管敏感** | 内容安全对齐要求更严格 |
| **硬件适配** | 对国产GPU（昇腾910B等）的优化支持更好 |
| **成本驱动** | DeepSeek等以"极致性价比"为主要卖点 |

---

## 七、开源AI商业模式：Red Hat模式的现代演绎

### 7.1 开源AI的商业逻辑

开源AI不能直接销售软件，因此商业模式需要围绕"开源之上的服务层"展开：

**企业服务订阅（Hugging Face Enterprise）**：
- 企业级模型托管与推理服务
- SSO、RBAC权限管理、审计日志
- SLA保障和技术支持
- 预计年营收已达数千万美元级别，增长迅速

**Red Hat模式（开源+商业发行版）**：
- 提供优化版开源模型（性能调优、安全加固、兼容性保证）
- 典型代表：Databricks的DBRX、Snowflake的Arctic
- 企业为"开箱即用"和"企业级保证"付费

**云API服务（开源模型商业化）**：
- Together AI、Anyscale、Replicate等平台托管开源模型，按调用计费
- 模型权重免费，但云服务收费
- Together AI获得了a16z等明星VC多轮注资

**平台佣金（Marketplace模式）**：
- 模型托管平台对模型推理调用收取佣金
- Hugging Face Inference API采用此模式

**硬件合作**：
- 英伟达、AMD、英特尔等硬件厂商与开源AI框架深度合作
- 开源优化工具（TensorRT-LLM、vLLM）绑定自家硬件生态

### 7.2 典型企业案例

| 企业 | 模式 | 营收来源 |
|------|------|---------|
| Hugging Face | 平台+企业服务 | 企业订阅、API佣金 |
| Red Hat（IBM） | 开源发行版+支持 | 企业订阅 |
| Databricks | 开源基础+数据平台 | SaaS订阅 |
| Together AI | 开源模型云托管 | API调用计费 |
| Anyscale（Ray） | 开源框架+云托管 | 计算资源+支持 |
| Mistral AI | 开源+商业模型 | API调用+企业合同 |

---

## 八、开源AI社区：贡献者生态与项目格局

### 8.1 GitHub星标前10AI相关开源项目（2024-2025）

| 排名 | 项目 | Stars（估算） | 核心功能 |
|------|------|-------------|---------|
| 1 | lang-ai/langchain | ~100k | LLM应用框架 |
| 2 | AUTOMATIC1111/stable-diffusion-webui | ~130k | AI图像生成WebUI |
| 3 | ggerganov/llama.cpp | ~60k+ | 本地LLM推理引擎 |
| 4 | ollama/ollama | ~80k | 本地模型运行管理 |
| 5 | vllm-project/vllm | ~40k+ | 高性能LLM推理 |
| 6 | huggingface/transformers | ~70k+ | 模型训练与推理库 |
| 7 | facebookresearch/llama | ~55k | LLaMA官方仓库 |
| 8 | QwenLM/Qwen | ~20k+ | 通义千问开源版 |
| 9 | THUDM/ChatGLM3 | ~15k+ | 智谱ChatGLM3 |
| 10 | deepseek-ai/DeepSeek-V3 | ~40k+ | DeepSeek开源系列 |

### 8.2 贡献者生态分析

开源AI社区的贡献者可分为三类：

**学术型**：以大学实验室和研究机构为主。EleutherAI（做GPT-NeoX等）、BigScience（做BLOOM）、LAION（做CLIP）是典型代表。学术开源的特点是追求方法论透明和数据可审计，但工程化程度相对有限。

**企业主导型**：Meta（LLaMA）、Google（Gemma）、Microsoft（Phi-3、DeepSpeed）、阿里（Qwen）是主力。企业开源通常有更完善的工程化、文档和社区运营，但背后有商业动机，许可证也更复杂。

**社区自发型**：LLaMA-Factory、LLaMA.cpp、Axolotl等工具项目，以个人开发者和小团队为主，迭代速度快，紧贴实际需求，但可持续性依赖核心维护者的热情。

### 8.3 开源社区的协作模式

开源AI社区已形成独特的协作模式：
- **上游优先**：大部分改进先在开源社区实验，再被上游模型厂商采纳
- **Hugging Face as Hub**：模型卡片、权重托管、评测基准的集中地，扮演了事实上的标准接口角色
- **微调民主化**：LoRA适配器的共享文化，使得"一个基座模型+数千个垂直LoRA"成为标准范式

---

## 九、开源AI基准评测：方法论与局限性

### 9.1 HuggingFace Open LLM Leaderboard

Open LLM Leaderboard由Hugging Face团队策划，是目前最具影响力的开源模型评测平台。2024年升级为**v2版本**，核心改进包括：

**评估框架升级**：与EleutherAI团队合作，更新了lm-evaluation-harness评测工具，加入对delta权重（LoRA微调后增量）的支持。

**四大核心指标**：
1. **MMLU**（Massive Multitask Language Understanding）：57个学科的多选题测试，考察知识广度
2. **MMLU-Pro**：MMLU的扩展版，题目更难、选项更多（10选1）
3. **GPQA Diamond**：研究生水平科学问题，考察深度推理
4. ** MATH**：数学奥赛级别问题，考察复杂推理能力

**评测方法论要点**：
- 标准化Prompt模板，排除Prompt工程的干扰
- 公开评测代码和数据，确保可复现性
- 区分"基座模型"和"指令微调模型"，避免混排

### 9.2 其他重要基准

| 基准 | 考察能力 | 特点 |
|------|---------|------|
| **HumanEval** | 代码生成 | Pass@1评估，LeetCode风格 |
| **MATH** | 数学推理 | 分步骤评分 |
| **BBH**（Big Bench Hard） | 复杂推理 | 23项挑战任务 |
| **IFEval** | 指令遵循 | 严格格式合规检测 |
| **AlpacaEval** | 通用对话质量 | 与GPT-4-Turbo对比胜率 |
| **MT-Bench** | 多轮对话 | 8类问题两轮对话 |

### 9.3 评测的局限性

**基准泄露风险**：模型在训练数据中见过评测数据，导致分数虚高（尤其是MMLU）。

**能力覆盖不全**：现有基准难以评估创造性思维、长期规划、具身智能等前沿能力。

**实用性与基准的脱节**：部分模型在基准上表现优异，但实际用户体验一般，反之亦然。

**微调与基座的混淆**：部分评测只测微调后的指令模型，掩盖了基座模型的能力差距。

---

## 十、开源AI趋势：从工具到AGI的路径探索

### 10.1 开放权重运动的深化

2024-2025年，开放权重运动进入新阶段：

**从语言到多模态**：LLaMA 3.1支持图像理解，DeepSeek-V3开源多模态版本，Qwen2.5-VL支持视觉理解，开源从纯文本扩展到全模态。

**推理能力的突破**：DeepSeek-R1证明开源模型可以在推理任务上达到与闭源顶尖模型相当的水平，改变了"闭源=推理强"的假设。

**Agent能力开源**：Mistral Devstral、Qwen-Coder等开源模型在软件工程Agent场景取得突破，开源AI正在从"聊天工具"进化为"行动工具"。

### 10.2 Together AI与Mistral的战略

**Together AI**定位为"开源模型云平台"，聚合了LLaMA、Mistral、Qwen等开源模型，提供优化的推理API。其策略是：
- 不训练自己的闭源模型，只做开源模型的最佳部署平台
- 开放模型权重下载，同时提供付费云推理服务
- 获得了超过3亿美元融资，估值超10亿美元

**Mistral AI**采用"开源+商业"的混合策略：
- 开源Mistral 7B、Mixtral等模型，建立开发者生态
- 商业版本Le Chat（面向企业）和La Plateforme（API服务）
- 欧洲市场定位：作为欧洲本土AI力量，减少对美国闭源模型的依赖
- 2025年12月发布Mistral 3，在多项基准上逼近GPT-4o

### 10.3 开源AGI的可能性

能否通过开源实现AGI（通用人工智能）？这是当前AI社区最深刻的争论之一：

**乐观派论点**：
- 开源模型的能力正在以更快速度追赶闭源模型
- 全球开发者社区的集体智慧可能超过任何单一公司
- LLaMA 3.1 405B等模型已在许多任务上达到GPT-4水平
- DeepSeek-R1证明创新可以在开源框架下发生

**悲观派论点**：
- GPT-5等最新闭源模型的能力边界仍在快速扩张
- AGI需要的安全对齐、长期记忆、持续学习等能力，开源尚未系统性解决
- 训练前沿模型所需的算力和数据资源，仍是开源社区难以复制的壁垒
- 闭源厂商的"模型合并"（Model Merging）策略可能形成新的不对称

**中间路线**："开放权重≠完全开放"。即使没有完全开源，前沿模型的权重逐步开放已是不可逆趋势，这本身就足以深刻改变AI权力格局。

### 10.4 未来十年的关键问题

1. **开源与闭源的能力差距会消失吗？** 可能性在上升，尤其在推理效率和经济性上开源可能反超。
2. **开源模型会成为企业主流选择吗？** 在合规要求高、数据敏感、成本敏感的场景，开源优势明显；但在对能力上限要求极高的场景，闭源仍将占主导。
3. **开源AI的治理框架会走向何方？** EU AI Act等监管框架可能对开源模型采取分级豁免，但强制水印、溯源追踪等机制可能会改变开源的"完全匿名"特性。
4. **中国开源AI能否引领全球？** DeepSeek-R1已经证明中国团队具备原创能力，未来在推理优化、多模态、中文理解等领域的领先可能扩大。

---

## 结语：开源AI的三重意义

开源AI生态的意义，远不止于技术层面。

**第一重：平权意义**。开源打破了AI能力被少数公司垄断的结构，让全球每一个开发者、研究者和用户都能平等地接触和使用最前沿的AI能力。这与互联网最初的开放精神一脉相承。

**第二重：安全意义**。开源模型可被完整审计，这为识别偏见、发现漏洞、研究对齐提供了闭源系统无法提供的透明度。AI安全的终极答案，可能藏在开源社区的集体智慧中。

**第三重：创新意义**。历史反复证明，开源社区在创新速度上往往超过商业公司。Linux、Git、Python、React——几乎所有改变了软件开发范式的技术，都来自开源社区。AI领域的类似故事，才刚刚开始。

开源AI正处于从"追赶者"到"引领者"的历史转折点。DeepSeek-R1的出现是这个转折点上最具标志性的事件，它证明开源AI不是闭源的"廉价替代品"，而是独立创新的一极。未来的AI格局，开源与闭源的竞争与融合，将决定整个行业的技术走向和权力分布。

---

**参考资料（部分）**

- Meta AI. "The Llama 3 Herd Models." arXiv:2407.21783, 2024.
- DeepSeek-AI. "DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning." 2025.
- Qwen Team. "Qwen Technical Report." arXiv:2309.16609, 2023.
- Touvron et al. "LLaMA: Open and Efficient Foundation Language Models." Meta AI, 2023.
- Jiang et al. "Mistral 7B." arXiv:2310.06825, 2023.
- Hugging Face. "Open LLM Leaderboard v2 Documentation." 2024.
- Ars Technica. "Hugging Face Surpasses 1 Million AI Models." September 2024.
- Nature. "How China Created DeepSeek and Shocked the AI World." January 2025.
- CNBC. "China's Open-Source Embrace Upends Conventional Wisdom." March 2025.

---

*报告生成日期：2026年4月 | 数据来源：GitHub官方数据、HuggingFace Hub官方数据、arXiv学术论文、TechCrunch、Ars Technica等权威科技媒体*
