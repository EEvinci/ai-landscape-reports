# 13. Inference Optimization

> 调研时间：2026-04-08 | 覆盖范围：2024–2025 年技术进展

---

## 核心概念

### LLM 推理的两大阶段

LLM 推理分为 **Prefill（预填充）** 和 **Decode（解码）** 两个阶段：

- **Prefill 阶段**：处理输入 prompt，计算首个 token，计算密集型，适合大批量并行
- **Decode 阶段**：自回归生成每个 token，显存访问密集，生成速度受限于显存带宽

### 三大核心瓶颈

| 瓶颈类型 | 具体表现 | 量化影响 |
|---------|---------|---------|
| **计算墙** | Attention 复杂度 O(n²)，128K 长文本指数爆炸 | 32K 输入时 Attention 耗时占比 > 80% |
| **内存墙** | KV Cache 显存占用达模型参数 4–8 倍 | LLaMA-70B FP16 下需 560GB 显存 |
| **调度墙** | 静态批处理长度差异导致 GPU 空闲 | 请求长度差异 10 倍时，利用率 < 25% |

### 三项核心技术原理

#### 1. Continuous Batching（连续批处理）

**解决的问题**：变长请求导致 GPU 空闲槽位。

传统静态批处理必须等所有请求完成才能开始新批次，而 Continuous Batching 在**每次迭代结束时**检查请求队列——当某请求完成生成，其 KV 缓存释放，新请求立即插入下一个步骤，实现真正的 iteration-level 调度。

- GPU 利用率从 30–40% 提升至 **75–85%**
- 在 OPT-30B 模型上，吞吐量从 450 → **9,800 tokens/s**（约 22 倍提升）

#### 2. PagedAttention（分页注意力）

**解决的问题**：KV 缓存的预分配碎片化和浪费。

将 OS 风格的虚拟内存分页引入 KV 缓存管理：
- 缓存分为固定大小的块（默认 16 tokens/块）
- 按需分配、实时释放，无需连续预分配
- 配合 BlockTable 逻辑→物理地址映射，实现灵活调度
- 显存利用率从 ~20% 提升至 **90%+**

**块大小计算（以 Llama 3.3 70B 为例）**：
- BF16 KV Cache：`80 × 8 × 128 × 16 × 2 × 2 = 5.0 MB/块`
- FP8 KV Cache：`80 × 8 × 128 × 16 × 1 × 2 = 2.5 MB/块`

#### 3. Speculative Decoding（投机解码）

**解决的问题**：自回归解码的顺序依赖限制了并行度。

核心思想：用一个小模型（Draft Model）快速生成多个候选 token，再由大模型（Target Model）并行验证，接受的 token 可一次吐出多个，大幅降低平均解码步数。

| 方案 | 代表工作 | 特点 |
|------|---------|------|
| 独立小模型 | DeepMind Speculative Decoding | 需额外训练小模型 |
| 自投机 | Medusa、Eagle | 复用主模型 hidden states，无需额外训练 |
| N-gram 统计 | vLLM NGram Speculative | 无需训练，支持 Chunked Prefill（v0.18.0+） |
| 批投机 | Batch Speculative Decoding | 2024 年新进展，需保证输出等价性 |

> **2024–2025 关键进展**：Batch Speculative Decoding 领域发现现有批实现存在输出不等价问题（OpenReview 2024），正确实现需要在调度层面严格对齐。

---

## 主流框架对比

### 框架概览

| 框架 | 开发方 | 首次发布 | 核心定位 | 开源协议 |
|------|--------|---------|---------|---------|
| **vLLM** | UC Berkeley Sky Computing Lab | 2023.06 | 高吞吐、易用、通用推理引擎 | Apache 2.0 |
| **SGLang** | UC Berkeley LMSYS.org | 2024.01 | 结构化生成语言、复杂推理任务 | Apache 2.0 |
| **TensorRT-LLM** | NVIDIA | 2023.09 | 极致性能、深度硬件优化 | NVIDIA Proprietary |

### 核心技术架构对比

| 维度 | vLLM | SGLang | TensorRT-LLM |
|------|------|--------|-------------|
| 内存管理 | PagedAttention（分页式） | RadixAttention（基数树） | Paged KV Cache |
| KV 缓存共享 | 不支持前缀共享 | 支持自动前缀缓存共享 | 部分支持 |
| 调度器 | 异步迭代级调度 | 高效调度 + 前缀复用 | In-flight Batching |
| Attention 内核 | 优化 CUDA 内核 | FlashInfer 融合算子 | 手写 CUDA + cuBLAS/cuDNN |
| 结构化输出 | 基础支持（后处理） | **原生正则约束解码** | 不支持 |
| 量化方案 | AWQ、GPTQ、SqueezeLLM | AWQ、GPTQ | **FP8、INT8、INT4、AWQ、GPTQ、SmoothQuant** |
| 并行策略 | TP、PP | TP、DP | TP、PP、Expert 并行、序列并行 |
| 多模态 | 实验性 | **原生支持** | 有限支持 |
| API 兼容性 | OpenAI 兼容 | OpenAI 兼容 + 自定义 DSL | OpenAI 兼容 + trtllm-serve |

### H100 基准测试数据（Llama 3.3 70B FP8, 2026 实测）

| 指标 | vLLM v0.18 | TensorRT-LLM v1.2 | SGLang v0.5 |
|------|-----------|------------------|-------------|
| **50 并发吞吐量** | 1,850 tok/s | 2,100 tok/s | 1,920 tok/s |
| **TTFT p50（10 并发）** | 120 ms | 105 ms | 112 ms |
| **TTFT p95（100 并发）** | 1,450 ms | 1,280 ms | 1,380 ms |
| **冷启动时间** | ~62 秒 | **~28 分钟**（引擎编译） | ~58 秒 |
| **峰值显存（80GB H100）** | 78 GB | 79 GB | 78 GB |
| 模型支持广度 | **最广** | 较少 | 中等 |

### 各框架核心优势与局限

**vLLM**
- ✅ 生态最成熟、社区最活跃（GitHub Stars 超 50k）
- ✅ 硬件支持最广（NVIDIA + AMD ROCm）
- ✅ OpenAI API 兼容，上手最快
- ✅ PagedAttention 显存效率 90%+
- ✅ 更新迭代快（每周多次）
- ⚠️ 无原生前缀缓存，多轮对话不如 SGLang
- ⚠️ 不支持 FP8 量化
- ⚠️ 结构化输出需后处理

**SGLang**
- ✅ 多轮对话性能领先：RadixAttention 带来 **5×** 吞吐量提升
- ✅ 原生结构化输出（正则约束解码），直接生成 JSON/XML
- ✅ DSL 提供强大控制流，适合 Agent 复杂工作流
- ✅ 多模态原生支持（VLM/LLaVA）
- ✅ 100 轮对话场景下延迟降低 **40%**，显存节省 **50%**
- ⚠️ 社区相对小，生态较新
- ⚠️ DSL 有学习曲线
- ⚠️ 文档不如 vLLM 完善

**TensorRT-LLM**
- ✅ 极致性能：吞吐量和延迟均为最优
- ✅ 量化支持最全面，FP8（需 H100/L40S）性能领先
- ✅ 多 GPU 并行效率最高（~85–90%）
- ✅ 官方 NVIDIA 支持，企业级稳定性
- ✅ 模型可编译为加密引擎（IP 保护）
- ⚠️ 仅支持 NVIDIA GPU
- ⚠️ 冷启动需约 28 分钟引擎编译（重启仅 90 秒）
- ⚠️ 学习曲线陡，TensorRT 底层优化门槛高

### 选型决策矩阵

| 需求场景 | 推荐框架 |
|---------|---------|
| 快速上手、通用场景 | **vLLM** |
| 多轮对话、RAG、共享前缀工作负载 | **SGLang** |
| 极致性能、长期单一模型生产环境 | **TensorRT-LLM** |
| 结构化输出（JSON/XML）、Agent 应用 | **SGLang** |
| 需要 2 分钟内快速启动 | **vLLM / SGLang** |
| 多硬件支持（AMD + NVIDIA） | **vLLM** |
| 成本敏感 + 追求性能 | **vLLM + 量化** |
| 已有 NVIDIA 生态 + 追求极致性能 | **TensorRT-LLM** |
| 研究与实验 | **vLLM 或 SGLang** |

---

## 2024–2025 技术进展

### 1. FlashAttention-3（2024）

- 引入 5D Tensors 分块（batch×head×seq×d×block）
- 支持交错式流水线，进一步提升 HBM 访问效率
- A100 上 32K 序列：30.4s → **3.8s**（8 倍加速）

### 2. Chunked Prefill（分块预填充）

解决长预填充请求导致的**队头阻塞（Head-of-Line Blocking）**：

- 将预填充拆分为 N-token 块，每块之间交错执行其他活跃序列的解码步骤
- 长上下文请求分片处理，其他请求持续生成
- 实测效果（50 并发，10% 为 32K-token 输入）：
  - p50 TTFT 几乎不变（680ms → 720ms）
  - **p95 TTFT 降低 68%**（2,800ms → 890ms）

> vLLM v0.18.0 中，基于 draft model 的 speculative decoding 与 `--enable-chunked-prefill` **不兼容**，但 NGram GPU speculative decoding **已支持**共存。

### 3. RadixAttention（SGLang 核心创新）

- 在 KV 缓存层引入基数树（Radix Tree）数据结构
- 自动识别和复用相同前缀的请求（如系统提示词）
- 多轮对话场景：100 轮对话吞吐量提升 **5×**，显存节省 **50%**
- 部署规模：超过 **30 万 GPU**，日处理数万亿 tokens

### 4. 批处理与投机的融合冲突（2024）

Continuous Batching 与 Speculative Decoding 在调度层面存在直接冲突：
- Batching 追求批量处理最大吞吐量
- Speculative 追求单序列快速完成验证

2024 年 OpenReview 论文 "Batch Speculative Decoding Done Right" 指出多个现有批实现**违反输出等价性**——投机解码必须产生与逐条执行完全一致的 token 序列。正确实现需要调度层面的严格对齐保证。

### 5. FP8 量化全面落地

- **H100/L40S** 原生支持 FP8 算术指令
- TensorRT-LLM 在 FP8 模式下，70B 模型吞吐量达 **7,000 tok/s**（H100）
- vLLM 在 v0.18.0 中仍不支持 FP8，主要支持权重量化（AWQ/GPTQ）

### 6. 多步推理（Multi-Step Decoding）

- SGLang 在 v0.3.0+ 引入多步推理，每轮迭代生成多个 token
- vLLM v0.6.0 基准对比：SGLang 中位 TTFT **比 vLLM 快 3 倍**（在在线场景下）

### 7. 模型支持持续扩展

- vLLM：HuggingFace Transformers 全兼容，GPT、LLaMA、Mistral、Qwen 等
- SGLang：LLM + VLM 原生支持（LLaVA 等）
- TensorRT-LLM：需要特定模型支持列表，生态相对封闭

---

## 成本优化策略

### 策略一：量化压缩

| 量化方案 | 精度损失 | 显存节省 | 吞吐提升 | 代表框架 |
|---------|---------|---------|---------|---------|
| FP16 → FP8 | 极小 | ~50% | 1.5–2× | TensorRT-LLM |
| FP16 → INT8 | 较小 | ~60% | 1.5–2× | 全框架 |
| FP16 → INT4 (AWQ) | 可接受 | ~75% | 2–2.5× | vLLM/SGLang/TRT-LLM |
| FP16 → INT4 (GPTQ) | 可接受 | ~75% | 1.8–2.2× | vLLM/SGLang/TRT-LLM |

**实测数据（LLaMA-2-7B on A100）**：

| 量化方案 | vLLM | SGLang | TensorRT-LLM |
|---------|------|--------|-------------|
| FP16 | 2,500 tok/s | 3,800 tok/s | 4,200 tok/s |
| AWQ INT4 | 4,200 tok/s | 5,800 tok/s | 6,500 tok/s |

### 策略二：KV 缓存优化

- **PagedAttention**（vLLM）：显存利用率 90%+，支持 2–4× 并发请求数
- **RadixAttention**（SGLang）：前缀复用场景节省 30–50% 显存
- **前缀缓存**：相同系统提示词在多请求间共享 KV 缓存

### 策略三：批处理与调度优化

- **Continuous Batching**：GPU 利用率 30–40% → **75–85%**，吞吐量 2–3× 提升
- **Chunked Prefill**：p95 TTFT 降低 50–70%，消除队头阻塞
- **动态调整** `--max-num-seqs`（默认 1024）和 `--max-num-batched-tokens`（默认 8192–32768）

### 策略四：并行推理架构

| 并行策略 | 适用场景 | 通信开销 | 效率 |
|---------|---------|---------|------|
| **Tensor Parallelism (TP)** | 单节点多卡，巨大模型 | NVLink 高带宽 | ~85–90% |
| **Pipeline Parallelism (PP)** | 多节点，深度模型 | 较低 | ~70–80% |
| **Expert Parallelism (EP)** | MoE 模型（如 Mixtral） | 中等 | ~80–85% |
| **Context Parallelism (CP)** | 超长序列（如 128K+） | 中等 | ~75–80% |

**分布式推理组合**（以 LLaMA-2-70B 为例）：

| 配置 | 硬件 | 吞吐量 | 成本（相对单卡） |
|------|------|--------|----------------|
| 单卡 A100 | 1× A100 80GB | ~300 tok/s | 1× |
| 4× A100 | 4× A100 80GB | 500–800 tok/s | 4× |
| 8× A100 | 8× A100 80GB | 1,000–1,500 tok/s | 8× |

### 策略五：流式输出降低感知延迟

- **SSE（Server-Sent Events）**：每个 token 生成后立即推送，首 Token 延迟降至 **300ms 以内**
- vLLM / SGLang / TensorRT-LLM 均支持 OpenAI 兼容的流式 API
- 配合 Chunked Prefill，在长输入场景下用户体验显著改善

### 策略六：模型路由与智能调度

- **LeanLM 等平台**：自动检测过度参数化的 LLM 调用，路由至更小模型
- **Prefix caching**：共享系统提示词避免重复计算
- 综合优化可实现 **3–10× 成本降低**

### 成本量化参考

| 优化组合 | 效果 | 成本降幅 |
|---------|------|---------|
| vLLM + PagedAttention | 吞吐量 3–4× | ~60% |
| + Continuous Batching | 吞吐量累计 3–4× | ~70% |
| + INT4 量化 | 吞吐量 2–2.5× | ~75% |
| + SGLang 前缀缓存（多轮） | 吞吐量 5× | ~80% |
| **综合最优**（SGLang + 量化） | 吞吐量 ~10× | **~90%** |

---

## 推荐资源

### 官方文档与代码

| 资源 | 链接 | 说明 |
|------|------|------|
| vLLM 官网 | https://vllm.ai | 文档、博客、GitHub |
| vLLM GitHub | https://github.com/vllm-project/vllm | Apache 2.0 开源 |
| SGLang GitHub | https://github.com/sgl-project/sglang | Apache 2.0 开源 |
| SGLang 文档 | https://sgl-project.github.io | 包含博客与论文 |
| TensorRT-LLM | https://github.com/NVIDIA/TensorRT-LLM | NVIDIA 官方 |
| TensorRT-LLM 文档 | https://nvidia.github.io/TensorRT-LLM | 性能基准与使用指南 |

### 关键论文

| 论文 | 来源 | 核心贡献 |
|------|------|---------|
| *Efficient Memory Management for LLMs with PagedAttention* | SOSP 2023 | PagedAttention 原理 |
| *Orca: A Distributed Serving System for Transformer-Based Generative Models* | OSDI 2022 | Continuous Batching 起源 |
| *FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness* | NIPS 2022 | FlashAttention 基础 |
| *SGLang: Efficient Execution of Structured Language Model Programs* | arXiv 2023 | SGLang 与 RadixAttention |
| *Batch Speculative Decoding Done Right* | OpenReview 2024 | 批投机解码正确性 |

### 技术博客与对比报告

| 文章 | 来源 | 亮点 |
|------|------|------|
| vLLM vs TensorRT-LLM vs SGLang: H100 Benchmarks (2026) | Spheron | 最新 H100 基准测试数据 |
| LLM Serving Optimization: Continuous Batching & PagedAttention | Spheron | 技术原理与 H100 实测 |
| 大模型推理优化三剑客（知乎） | 知乎专栏 | 技术数学推导与成本分析 |
| vLLM、SGLang 与 TensorRT-LLM 综合对比 | 阿里云开发者社区 | 全面功能对比表 |
| LLM Inference Optimization: Speed, Cost & Scalability | DeepSense.ai | 成本优化全景指南 |

### 开源工具与平台

| 工具 | 用途 |
|------|------|
| LeanLM | LLM 成本优化平台（路由/缓存/蒸馏/量化） |
| LeanDojo | 自动化 LLM 服务分析 |
| mini-infer | 从零手写推理引擎（含 CB/PA/SD 等核心模块） |

---

*本报告基于 2024–2025 年公开技术资料整理，数据来自各框架官方基准测试与第三方测评，实际性能因硬件、模型、工作负载不同可能有所差异。建议在目标场景下进行实测验证。*
