# LLM Architecture：大型语言模型底层架构综述

> 调研时间：2026-04-08 | 研究范围：Transformer核心 → 架构演进 → 位置编码 → MoE/新型架构 → 上下文扩展 → 推理优化

---

## 概述

2024-2025 年，大型语言模型（LLM）从"能用"走向"好用"，底层架构创新功不可没。从 2017 年 Vaswani 等人提出 Transformer 以来，其核心设计在十年间经历了无数次微调与重构：注意力机制的 IO 效率被 FlashAttention 彻底改造，位置编码从 Sinusoidal 演进到 RoPE，稀疏激活的 MoE 架构让千亿参数模型的训练成本大幅下降，而在 Transformer 之外，Mamba、RWKV 等新型架构也在特定场景中崭露头角。

理解这些底层架构设计，不仅是模型研发者的专利——对于 AI 产品经理、算法工程师乃至技术决策者而言，掌握 LLM 架构的核心逻辑，意味着能够更准确地判断模型能力边界、推理成本结构，以及未来技术演进的可能方向。

---

## 一、Transformer 核心组件

### 1.1 Self-Attention（自注意力机制）

Self-Attention 是 Transformer 的核心创新，也是其区别于 RNN 的根本所在。它允许输入序列中的每个位置同时"关注"所有其他位置，从而建模任意距离的依赖关系。与传统 RNN 的顺序计算相比，Transformer 的自注意力层可以通过矩阵运算完全并行化，极大释放 GPU/TPU 的计算潜力。

标准注意力公式为：

```
Attention(Q, K, V) = softmax(QK^T / √d_k) × V
```

其中 Q、K、V 由输入向量分别经三个线性变换得到，√d_k 用于缩放点积，防止梯度消失。现代 LLM 通常将 d_model 扩展到 4096～8192 维，同时使用 40～128 个注意力头并行计算，每个头的维度通常为 64～128。

**为何需要多头注意力？** 单一注意力的加权求和只能捕捉一种关联模式，而多头注意力允许模型在不同的表示子空间同时学习不同类型的依赖关系——语法、语义、位置、指代等——类似于 CNN 中多个卷积核同时提取不同特征的做法。

自注意力的核心瓶颈在于**O(N²) 的显存占用**：对于长度为 N 的序列，注意力矩阵 QK^T 的维度为 N×N，N=131K 时单个矩阵就超过 68GB（FP16），这直接推动了 FlashAttention 等一系列内存优化技术的诞生。

### 1.2 FFN（Feed-Forward Network）

FFN 通常占据 Transformer 总参数的 2/3 左右（以 7B 模型为例，约 4.5B 参数来自 FFN），是 Transformer 中参数量最大的模块。其标准结构为两层线性变换，中间插入非线性激活函数：

```
FFN(x) = activation(xW₁ + b₁) × W₂ + b₂
```

其中中间层扩展比（expansion ratio）通常为 4，即 d_ffn = 4 × d_model（d_model=4096 → d_ffn=16384）。激活函数从原始的 ReLU 逐步演进到 GeGLU、SwiGLU 等更平滑的变体，后者在 LLaMA、Qwen 等主流模型中已成为标准配置。

FFN 在 Transformer 中扮演的角色远超简单的非线性变换。从信息流动的角度看，注意力层负责"路由"——决定信息在哪些 token 之间传递；而 FFN 负责"存储"——每个 token 独立经过一个两层的非线性映射，等效于在局部对信息进行非线性编码。从实验观察来看，FFN 的贡献在不同层级存在显著差异：浅层 FFN 更多处理词汇级特征，深层 FFN 则倾向于编码抽象知识和复合语义。

### 1.3 LayerNorm 与 Pre-LN 范式

LayerNorm（Layer Normalization）对每一层的激活值进行归一化，稳定训练梯度，防止数值溢出。其公式为：

```
LN(x) = γ × (x - μ) / σ + β
```

其中 μ 和 σ 是 x 的均值和标准差，γ 和 β 是可学习的缩放和偏移参数。与 BatchNorm 不同，LayerNorm 不依赖 batch 维度，使其在序列建模任务中更为稳定。

在 Transformer 中 LayerNorm 的**放置位置**（Pre-LN vs Post-LN）对训练稳定性有显著影响。原始 Transformer 采用 Post-LN（Norm 在 Add 之后），但深层网络的残差路径上梯度方差迅速增大，导致训练不稳定。**Pre-LN**（Norm 在子层输入处）通过在每个残差分支的起始处做归一化，使梯度路径更加平滑，已成为事实标准。DeepNorm（微软）、RMSNorm 等变体进一步优化了深层 Transformer 的训练稳定性。

### 1.4 位置编码（Positional Encoding）

Transformer 的自注意力机制是位置无关的——对输入序列做任何排列，输出都是相同的。位置编码是打破这一对称性的唯一机制。2024-2025 年，业界已从早期的绝对位置编码（Sinusoidal / Learnable）全面转向相对位置编码，其中 **RoPE** 与 **ALiBi** 两条技术路线最具代表性。

---

## 二、2024-2025 Transformer 架构演进

### 2.1 FlashAttention：IO感知的注意力计算

FlashAttention（FA，由 Stanford & Meta 提出）通过**分块计算（Tiling）+ 重计算（Recomputation）**，将注意力机制的显存占用从 O(N²) 降至 **O(N)**，同时保持数值等价性：

- **核心思路**：不将完整的 N×N 注意力矩阵物化为 HBM，而是分块加载到 SRAM 中计算，避免高带宽显存读写
- **重计算梯度**：用 O(N) 显存换 O(N) 计算，通过前向传播的小块保存必要信息，反向时重新计算注意力矩阵
- **FlashAttention-2**（2023）：进一步优化 CUDA 核函数，吞吐提升约 1.5～2 倍
- **FlashAttention-3**（2024）：引入 TensorFloat-32（TF32）加速、WGMMA 指令支持，理论速度再提升 1.5～2 倍；在 H100 GPU 上可达 740 TFLOPS
- 几乎所有 2024-2025 年发布的大模型（GPT-4o、LLaMA 3、Qwen2、DeepSeek-V3）底层均使用 FlashAttention

### 2.2 分组查询注意力（GQA）

传统 MHA 的 KV Cache 随头数线性增长，在长序列推理中成为严重瓶颈。**Grouped Query Attention**（GQA，Google 2023年提出，2024-2025 年全面普及）提供了 MHA 与 MQA 之间的优雅插值。

GQA 的核心思想是：将 Q 头分组，每组共享一套 K/V 投影。例如在 Llama 3 中，32 个 Q 头被分为 8 组，每组共享同一套 Key 和 Value 向量。数学上，第 j 组的输出为该组内所有 Q 头注意力的拼接：

```
O_j = concat(Z_j,0, Z_j,1, ..., Z_j,m-1) × W_O
```

实验数据（Llama 2 70B）表明，GQA 在长序列场景（2048 tokens）下推理速度快于 MHA，同时验证损失（43.60）显著优于 MQA（45.52），接近 MHA（42.62）。

| 机制 | KV Cache 规模 | 推理速度 | 质量 |
|------|-------------|---------|------|
| MHA（多头注意力）| N_heads × layers | 最慢 | 最高 |
| GQA（分组查询注意力）| N_groups × layers | 中等 | 接近 MHA |
| MQA（多查询注意力）| 1 × layers | 最快 | 质量下降 |

**采用 GQA 的主流模型**：LLaMA 3.1/3.2/3.3、GPT-4o、Claude 3.5、Qwen2、Mistral。值得注意的是，DeepSeek-V2 在 GQA 基础上进一步引入 **MLA（Multi-head Latent Attention）**，通过低秩压缩 Key/Value，进一步降低 KV Cache 显存占用，是 2024 年注意力机制优化的重要创新。

### 2.3 Sliding Window Attention（滑动窗口注意力）

SWin Attention 通过限制每个 token 只关注局部窗口（如 4096 token）来降低计算复杂度，结合全局 attention 层处理远距离依赖：

- **Mistral 7B**（2023）是首个大规模采用此方案的公开模型
- 可与 FlashAttention 深度结合，在保持局部建模能力的同时享受 O(N) 显存优化
- 主要缺陷是信息流通需要多层堆叠才能触达远距离 token，理论上有信息损失

---

## 三、RoPE vs ALiBi：位置编码技术路线对比

### 3.1 RoPE（旋转位置编码）

RoPE 由苏剑林（苏神）提出，首先在 RoFormer 中应用，目前是**最主流的位置编码方案**，被 LLaMA、Qwen、Baichuan、ChatGLM、DeepSeek 等绝大多数主流中文/英文模型采用。

**核心原理**：对 Q 和 K 向量进行旋转，使它们的内积只与相对位置有关：

```
f(q_m, m) = q_m × R(m, d)
f(k_n, n) = k_n × R(n, d)
⟨f(q,m), f(k,n)⟩ = q^T k × e^(i(m-n)θ)
```

**旋转角度公式**：θ_j = b^(-2j/d)，其中 b 通常为 10000

**关键特性**：
- ✅ 天然编码相对位置（内积只依赖 m-n）
- ✅ 支持长度外推（配合 NTK-aware 等方法）
- ✅ 高频维度保持旋转（外推），低频维度缩放位置（插值）
- ⚠️ 实现相对复杂，需要修改 Q/K 的生成逻辑
- ⚠️ 对超长上下文的外推能力仍需配合微调

### 3.2 ALiBi（线性偏置注意力）

ALiBi（Attention with Linear Biases）由 Presser 等人提出，方案更为简洁：**在注意力分数上直接加一个与相对距离相关的偏置项**，无需修改 Q/K/V 向量：

```
Attention(q_i, k_j) = softmax(q_i · k_j / √d + m·|i-j|)
```

其中 m 是每个注意力头特有的斜率参数：m_n = 1/2^(8n/h)，h 为头数。

**关键特性**：
- ✅ 实现极简，无需修改模型架构
- ✅ 长度外推能力好，无需微调即可处理更长序列
- ✅ 计算开销极小
- ⚠️ 线性偏置的表达能力有限，无法建模复杂的相对位置关系
- ⚠️ 在需要精确位置信息的任务上通常弱于 RoPE

### 3.3 技术路线对比与选择

| 维度 | RoPE | ALiBi |
|------|------|-------|
| 代表模型 | LLaMA、Qwen、DeepSeek、ChatGLM | Llama 2（早期）、MPT、Falcon |
| 外推能力 | 强（需配合 NTK/YaRN 方案）| 强（无需微调）|
| 表达能力 | 强 | 中等 |
| 计算开销 | 略高 | 几乎无 |
| 主流程度 | **绝对主流** | 相对小众 |

**趋势**：2024-2025 年新发布的大模型几乎清一色采用 RoPE 或其增强变体，ALiBi 主要在特定场景（如完全不需要微调的外推任务）中使用。

---

## 四、混合专家模型（MoE）与新型架构探索

### 4.1 MoE（Mixture of Experts）

MoE 是 2024-2025 年大模型 Scaling 的关键技术路径，其核心思想是**稀疏激活**：每层包含多个 FFN"专家"，但每次只激活其中 1～2 个，实现"参数量大但计算量小"。

**架构特点**：
- **专家数量**：通常 8～256 个 FFN 专家
- **路由器（Router）**：小型 MLP，输出每个专家的激活权重，通常配合 Top-K 稀疏选择
- **共享专家**：DeepSeek-V2/V3 引入"共享专家"机制，降低路由崩溃风险
- **负载均衡**：加入辅助损失防止少数专家被过度使用

**代表模型与效果**：

| 模型 | 专家数 | 激活专家数 | 参数量 | 特点 |
|------|--------|-----------|--------|------|
| Mixtral 8×7B | 8 | 2 | 46.7B | 开源 MoE 先驱 |
| DeepSeek-V2 | 128 | 8 | 236B | 共享专家 + MLA 注意力 |
| GPT-4（推测）| 约 16 | 约 2 | 约 1.8T | 闭源，架构未公开 |
| LLaMA 4（推测）| — | — | — | 预计采用 MoE |

MoE 的核心挑战：路由器负载不均、通讯开销（分布式训练）、推理时专家并行调度的工程复杂度。

### 4.2 新型序列建模架构

**Transformer 的 O(N²) 复杂度**催生了大量替代架构研究，2024-2025 年形成了几条成熟的技术路线：

#### Mamba（选择性状态空间模型）

Mamba 由卡内基梅隆大学 Albert Gu 团队提出，是 S4→HiPPO→Mamba 演进路线的集大成者：

- **选择性机制（Selection Scan）**：让 SSM 参数成为输入依赖的函数，实现类似注意力的内容感知建模
- **O(N) 线性复杂度**：序列越长，相比 Transformer 的效率优势越显著
- **硬件感知算法**：通过并行扫描（Parallel Scan）实现高效的 GPU 计算
- **主要局限**：在超大规模（70B+）语言建模任务上与 Transformer 仍有差距；状态大小固定，长程依赖的表达能力受限于状态维度

2025 年提出的 **Mamba-2** 的核心贡献是提出了 **State Space Duality（SSM-Attention 对偶性）** 理论框架，证明了 SSM 和注意力机制在数学上可以相互转化，从而在接口层面实现了统一。具体而言，Mamba-2 将 SSM 的选择性扫描表示为一种特殊的线性注意力形式，使得 SSM 层和 Attention 层可以在同一模型中混合使用，互为补充。这一理论突破为 Transformer-Mamba 混合架构奠定了基础：模型的浅层使用 Transformer 块捕捉复杂的全局依赖，深层使用 SSM 块进行高效的长程压缩，在保持能力的同时显著降低计算成本。实验表明，Mamba-2 在多项基准任务上优于原始 Mamba，部分场景已接近同规模 Transformer 的表现，标志着 SSM 路线从"替代品"向"补充模块"的定位转变。

#### RWKV（线性注意力 + RNN 融合）

RWKV 由 PENG Bo 等人提出，定位为**用 RNN 机制重新实现 Transformer 效果**：

- **核心创新**：将 Transformer 的注意力机制转化为等价的线性形式（Linear Attention），再用 RNN 风格的状态传递实现生成
- **STATE 矩阵机制**：用固定大小的状态矩阵 S 替代 KV Cache，通过 k·S^T 建立键到值的映射
- **RWKV-7（2025）**：引入动态演化状态矩阵，进一步逼近 Transformer 的表达力
- **最大优势**：推理时无需存储完整历史 KV，内存占用恒定；支持高效的 Prefix Caching
- **实际应用**：ChatRWKV 开源生态、微软 Windows 集成（通过 rwkv.cpp）、Jina AI 推动商业落地

#### RetNet 与 Hyena

- **RetNet**（Microsoft，2023）：提出 Retention 机制替代 Attention，保持 O(N) 训练复杂度的同时支持并行和递归两种推理模式；与 FlashAttention 同期工作，定位互补
- **Hyena**（H3 团队，2023）：用长卷积（Long Convolution）+ 数据控制门控替代注意力，在部分任务上接近 Transformer，但总体成熟度不如 Mamba/RWKV

**趋势对比**：

| 架构 | 理论复杂度 | 实际优势场景 | 成熟度 |
|------|---------|-----------|-------|
| Transformer | O(N²) | 全场景基准，Scaling Law 验证充分 | ⭐⭐⭐⭐⭐ |
| Mamba | O(N) | 长序列、硬件受限环境 | ⭐⭐⭐ |
| RWKV | O(N) | 边缘部署、低内存设备 | ⭐⭐⭐ |
| RetNet | O(N) | 学术研究，工业落地有限 | ⭐⭐ |

Transformer 仍是 2025 年大模型训练的绝对主流，但 Mamba/RWKV 在**推理效率**、**边缘部署**、**超长上下文**等特定场景已展现明确优势，预计将作为补充架构长期共存。

---

## 五、上下文长度扩展技术

大模型训练时的上下文窗口是有限的（如 2048/4096 token），但推理时需要处理更长的文本。核心矛盾在于：**直接外推会导致注意力分数异常，线性插值会丢失高频位置信息**。

### 5.1 发展脉络

```
直接外推（失败）→ 线性插值 PI（Meta, 2023）→ NTK-aware（2023）→ NTK-by-parts → YaRN（2023）→ LongRoPE（2024）
```

### 5.2 主要方法

**Position Interpolation（PI）**：Meta 2023 年提出，将位置索引直接缩放（÷scaling factor），仅需 1000 步微调即可扩展到 32K。缺陷是对所有维度一刀切，丢失局部位置信息。

**NTK-aware Scaled RoPE（RoPE ABF）**：Reddit 网友 bloc97 提出，核心思路是**高频外推 + 低频插值**，通过修改旋转基数 b 而非缩放位置索引，实现无需微调的外推。Code Llama 将 base 提升至 100 万。Meta 后续将其命名为"RoPE ABF"。

**NTK-by-parts**：在 NTK-aware 基础上引入波长（wavelength）概念，按 λ=2πb^(2i/d) 对每个维度分类：波长 > 上下文长度 → 插值；波长 << 上下文长度 → 外推；中间 → 混合。

**YaRN（Yet another RoPE extensioN）**：在 NTK-by-parts 基础上增加 **Pre-softmax Scaling（温度缩放）** 机制，只需 400 步微调即可将 LLaMA 7B 扩展至 128K 上下文，困惑度达到 2.37（Proof-pile 数据集）。已用于 DeepSeek-V2（128K）、Qwen2-128K 等模型。

**LongRoPE / LongRoPE v2**：微软 2024-2025 年提出，通过**非均匀位置插值**（不同维度不同缩放比例）和**混合上下文窗口训练**，将 LLaMA 等模型扩展至 200 万 token，同时尽量保持短上下文性能不下降。LongRoPE v2 引入了专门解决短上下文性能退化问题的机制。

### 5.3 主流模型上下文能力对比（2024-2025）

| 模型 | 最大上下文 | 扩展方法 |
|------|----------|---------|
| GPT-4o | 128K | 闭源，未公开 |
| Claude 3.5 | 200K | 闭源，未公开 |
| LLaMA 3.1 | 128K | YaRN |
| Qwen2 | 128K | YaRN |
| DeepSeek-V3 | 128K | YaRN + NTK |
| Kimi 1.5 | 200K | 闭源 |
| GLM-4 | 128K | 闭源 |

---

## 六、大模型推理效率优化

### 6.1 KV Cache

自回归生成中，每个新 token 都需要与所有历史 token 计算注意力。KV Cache 通过**存储已计算的 Key 和 Value 向量**，避免每步重新计算：

```
新 token 的注意力 = Attention(Q_new, [K_历史 + K_new], [V_历史 + V_new])
```

**瓶颈**：KV Cache 显存占用 = 2 × batch × seq_len × layers × heads × head_dim × dtype_size。在 70B 模型、131K 上下文下，仅 KV Cache 就需要数百 GB 显存。

**为什么没有 Q Cache**：Query 只用于当前步骤生成，无需复用；Key/Value 在每步都需要与新 Query 计算注意力，必须保留。

### 6.2 PagedAttention（vLLM 核心）

vLLM（UC Berkeley，2023年提出）借鉴操作系统内存管理思想，将 KV Cache 分块（Paged）管理：

- **传统方案**：为每个请求预分配连续的显存空间，碎片化严重（实际利用率仅 20-30%）
- **PagedAttention**：将 KV Cache 切分为固定大小的块（Block），按需分配，无内部碎片，**显存利用率提升至 60-80%**
- **Prefix Caching**：多个请求共享相同 system prompt / prefix，只需存储一份 KV，大幅降低重复计算
- vLLM 2024-2025 版本持续引入：**Speculative Decoding**（投机解码）、**Flash Attention 3 集成**、**Tensor Parallelism** 优化

### 6.3 其他重要优化方向

- **Continuous Batching（动态批处理）**：不等所有序列完成生成，新序列随时加入批次，GPU 利用率从 ~30% 提升至 ~60-80%
- **Speculative Decoding**：用小模型快速生成若干 token，大模型并行验证，减少自回归步数；实测可提升 2-3 倍首 token 速度
- **Tensor Parallelism（张量并行）**：将大矩阵乘法拆分到多 GPU，支撑 100B+ 参数模型的单机多卡推理
- **FlashInfer / LightLLM 等新推理框架**：针对不同硬件（Hopper/Blackwell）做专门优化
- **Prefix Caching 的工程实现**：共享 system prompt（如"你是一个有帮助的助手"）在多用户并发场景下效果显著，通常能减少 30-60% 的首 token 延迟

---

## 七、总结与趋势展望

2024-2025 年 LLM 架构演进呈现几个核心趋势：

**1. Transformer 仍是基础，但每个模块都在被精细化改造**
- FlashAttention-3 将注意力计算推向硬件理论峰值，IO 优化成为标准工程实践
- Pre-LN + GQA 的组合使训练稳定性和推理效率不再是对立目标
- 位置编码从"能用就行"演进到"精确适配"：RoPE 系方案结合 YaRN 已能稳定处理 128K+ 上下文

**2. Scaling 路径分化：宽模型 vs 稀疏模型**
- **稠密模型（Dense）**：LLaMA 3.2、Qwen2.5 等，架构简单，训练/推理流程标准化，是当前开源社区的主流选择
- **MoE 模型**：DeepSeek-V3、Mixtral 系列等，用稀疏激活换取训练成本优势；DeepSeek-V2 引入的 MLA（Multi-head Latent Attention）与共享专家机制进一步降低了通讯开销，代表了 MoE 工程化的前沿方向
- **核心工程挑战**：MoE 的路由负载均衡、多专家并行调度仍是分布式训练中尚未完全解决的问题

**3. 推理效率成为新的竞争维度**
- 推理成本（Token/$）正在成为与模型能力同等重要的竞争维度
- RWKV、Mamba 等线性复杂度架构在手机端、浏览器端、嵌入式设备上的部署优势明显
- vLLM、TensorRT-LLM、SGLang 等推理框架的成熟，使 70B 模型的单卡部署成为可能
- Speculative Decoding 与 Prefix Caching 的组合已在生产环境中实现 2-3 倍的首 token 速度提升

**4. 上下文长度成为厂商差异化竞争焦点**
- 上下文窗口从 8K 扩展到 128K 再到 200K+，背后是 YaRN、LongRoPE 等位置编码创新的持续突破
- "有效上下文"问题值得关注：部分研究指出许多模型在长上下文的后半部分出现"lost in the middle"现象，实际有效信息检索能力未必与标称窗口成正比
- LongRoPE v2 的混合上下文窗口训练方法试图同时解决长、短上下文性能问题，代表了下一阶段的技术方向

**5. 融合架构是长期演进方向**
- **Mamba-2** 尝试将 SSM 的线性复杂度和 Transformer 的表达能力做更深层次的融合
- **RWKV-7** 的动态状态矩阵是 RWKV 路线迄今最激进的改进，试图缩小与 Transformer 在任务表现上的差距
- 2025-2026 年的趋势可能是：Transformer 作为预训练的默认架构，而 Mamba/RWKV 在推理侧做效率优化，两者通过混合部署实现"训练用 Transformer，推理用 SSM"的分工

> **关键参考**：FlashAttention（2022-2024）、GQA（2023）、RoPE（2021）、YaRN（2023）、LongRoPE（2024）、Mixtral（2023）、DeepSeek-V2/V3（2024）、Mamba（2023）、RWKV-7（2025）、vLLM（2023）

---

*本报告基于 2024-2025 年公开论文、技术博客与开源项目信息整理，数据截至 2026-04-08。*
