# 12. Model Compression

> 调研时间：2026-04-08 | 研究范围：2024–2025 年 LLM 模型压缩技术进展

---

## 核心概念

### 什么是模型压缩

模型压缩（Model Compression）是指通过降低大语言模型（LLM）的参数量、精度或计算冗余，使其能在更少硬件资源下高效运行的一系列技术集合。LLM 压缩是解决"大模型参数规模爆炸与硬件算力/显存受限"这一核心矛盾的关键路径。

### 三大核心技术路线

| 技术 | 原理 | 压缩粒度 | 精度损失 | 部署收益 |
|------|------|----------|----------|----------|
| **量化（Quantization）** | 将 FP32/FP16 权重映射到低比特整数（INT4/INT8） | 权重级 | 1%~5% | 显存↓50-75%，速度↑2-4× |
| **剪枝（Pruning）** | 移除不重要的权重或结构组件 | 非结构化/结构化 | 0~10% | 参数量↓30-50% |
| **知识蒸馏（Distillation）** | 用大模型（Teacher）监督小模型（Student）学习 | 模型级 | 视方法而定 | 能力迁移至小模型 |

### 量化基础：INT4 vs INT8

- **INT8**：将权重从 FP16（2 字节）压缩到 1 字节，模型体积缩小 50%，精度损失通常 < 1%，是生产环境最常用选择
- **INT4**：将权重压缩到 0.5 字节，模型体积缩小 75%+，但精度损失约 2-5%；需借助 GPTQ/AWQ 等先进 PTQ 方法控制在 3% 以内
- **INT2/INT3**：极端压缩，理论上可行但精度损失显著，多为学术研究探索方向

量化分为两大范式：
- **训练后量化（PTQ）**：无需重新训练，用少量校准数据（通常 128-1024 条）校准量化参数，速度快（分钟级），工业界主流
- **量化感知训练（QAT）**：在训练过程中引入量化感知损失，精度更高但成本大（小时/天级），适合极端低比特（≤2bit）

---

## 2024-2025 技术进展

### 一、量化技术

#### PTQ 方法进化路线

1. **GPTQ（Generative Post-training Quantization）**
   - 核心：逐层量化，利用 Hessian 矩阵近似进行误差最小化
   - 支持 INT4/INT3/INT2
   - 优势：4bit 下精度优秀，适合追求精度的生产场景
   - 局限：量化时间较长（百亿参数模型需数小时）

2. **AWQ（Activation-aware Weight Quantization）**
   - 核心：观察激活值分布，识别并保护重要权重（weight quantization + activation scaling）
   - 优势：速度优先场景首选；对 13B 以上大模型效果好
   - 局限：需校准数据集，显存占用略高

3. **SpQR（Sparse Quantized Representation）**
   - 核心：结合稀疏性和量化，对敏感权重保持更高精度
   - 3.89bit 量化 LLaMA-30B，WikiText-2 perplexity 仅增 0.15

4. **VPTQ（Vector Post-Training Quantization）** ⭐ 2024 新突破
   - 核心：基于 Channel-Independent Second-Order Optimization 的向量量化方法
   - 在 2-bit 量化上显著优于 GPTQ/AWQ：LLaMA-3-7B perplexity 提升 4.41-7.34
   - 量化速度：仅需 SOTA 方法 10.4-18.6% 的时间
   - 推理吞吐：比 SOTA 提升 1.6-1.8×

5. **QuIP / QuIP#**
   - 基于非相干性（incoherence）处理的量化方法
   - 2-bit LLaMA2-70B perplexity 3.007（接近 FP16 水平）

6. **SmoothQuant & ZeroQuant**
   - 针对权重+激活同时量化（W8A8），解决激活值 outlier 问题
   - SmoothQuant：OPT-175B W8A8 perplexity 仅增 0.18

7. **LLM.int8()**
   - bitsandbytes 实现，OPT-13B W8A8 perplexity 无损（0.00 增加）

#### KV Cache 量化（2024 新方向）

长上下文场景下 KV Cache 显存占用急剧增长，成为新的瓶颈：

| 方法 | 核心思想 | 效果 |
|------|----------|------|
| **KIVI** | Key 按通道量化，Value 按 token 量化 | 2-bit，无需微调 |
| **KVQuant** | 10M 上下文长度下实现 2-bit，perplexity 增 0.19，1.4× 加速 | — |
| **WKVQuant** | 过去 token 量化 + 跨块重建 | 支持权重+KV联合量化 |

#### Meta 原生量化模型（2024.10）

Meta 在 2024 年 10 月发布了首批官方量化 Llama 模型（1B/3B）：
- 精度/安全对齐与原始模型持平
- 推理速度提升 2-4×
- 代表 llama.cpp GGUF 生态与官方模型的深度整合趋势

#### 基准精度数据（WikiText-2 Perplexity Diff）

| 方法 | 模型 | 比特数 | Perplexity 增量 | 加速比 |
|------|------|--------|-----------------|--------|
| LUT-GEMM | LLaMA-65B | 3 | +0.14 | 2.04× |
| SqueezeLLM | LLaMA-13B | 3 | +0.51 | 2.4× |
| GPTQ | OPT-175B | 3 | +0.34 | 3.24× |
| AWQ | LLaMA2-70B | 3 | +0.42 | 3.2× |
| SpQR | LLaMA-30B | 3.89 | +0.15 | 2.0× |
| QuIP | LLaMA2-70B | 2 | +3.007 | — |

### 二、剪枝技术

#### Unstructured Pruning（非结构化剪枝）

逐权重剪枝，最灵活但硬件加速有限：

| 方法 | 模型 | 稀疏度 | Perplexity 变化 |
|------|------|--------|-----------------|
| **SparseGPT** | OPT-175B | 50% | -0.14（更优） |
| **Wanda** | LLaMA-65B | 50% | +1.01 |
| **SAMSP** | LLaMA2-13B | 50% | +0.63 |

#### Structured Pruning（结构化剪枝）

按通道/注意力头/层为单位剪枝，可直接获得硬件加速：

| 方法 | 模型 | 压缩比 | Perplexity 增量 | 加速比 |
|------|------|--------|-----------------|--------|
| **SliceGPT** | LLaMA2-70B | 30% | +1.73 | 1.87× |
| **LLM-Pruner** | LLaMA-13B | 20% | +3.6 | — |
| **FLAP** | LLaMA-65B | 50% | +7.09 | — |

#### Semi-Structured / N:M Pruning（2024 热点）

利用 NVIDIA A100 等硬件的 2:4 结构化稀疏特性（每 4 个元素中保留 2 个），兼顾精度与加速：

| 方法 | 模型 | 模式 | Perplexity 增量 | 加速比 |
|------|------|------|-----------------|--------|
| SparseGPT (2:4) | OPT-175B | 2:4 | +0.39 | 2× |
| Wanda (2:4) | LLaMA-65B | 2:4 | +2.69 | 1.24× |
| E-Sparse (2:4) | LLaMA-65B | 2:4 | +2.13 | 1.53× |

> **结论**：N:M 稀疏性在实际部署中实用性最强，在 NVIDIA GPU 上可获得 2× 加速且 perplexity 损失可控。

### 三、知识蒸馏

#### 2024-2025 重要进展

**Black-box KD（以闭源 LLM 作为 Teacher）**：
- **Chain-of-Thought 蒸馏**：将推理能力（CoT）从小模型迁移到更小模型，是 MiniLM 等方法的核心
- **In-Context Learning 蒸馏**：Meta-ICT 通过上下文引导适应新任务
- **指令遵循蒸馏**：SELF-INSTRUCT 让 Student 模型自己生成指令数据集

**White-box KD（以开源 LLM 作为 Teacher）**：
- **MiniLLM**：引入反向 KL 散度（Reverse KL Divergence），防止学生模型低估低概率区域的概率分布，在生成质量上显著优于标准 KL
- **GKD**：用自生成输出训练自回归模型，解决 Teacher-Student 分布不匹配问题
- **TED（Task-aware layer-wise Distillation）**：逐层对齐隐藏表示，结合任务感知策略

**2024 NeurIPS 新成果 - Compact Language Models via Pruning and Distillation**：
- NVIDIA 将剪枝+蒸馏结合，压缩 Nemotron-4 系列模型 2-4×，在语言建模任务上对标同规模最佳模型

### 四、低秩分解（Low-Rank Factorization）

| 方法 | 核心思想 | 效果 |
|------|----------|------|
| **LPLR** | 随机草图 + 量化压缩权重矩阵 | 参数压缩 |
| **ASVD** | 激活感知的 SVD，融入激活分布的缩放因子 | — |
| **LASER** | 层选择性秩减少，提升稀有训练数据处理能力和改写抵抗性 | — |

---

## 主流工具生态

### 量化工具全景

| 工具 | 开发者 | 定位 | 特色 |
|------|--------|------|------|
| **llama.cpp / GGUF** | Georgi Gerganov | CPU/边缘推理 | 纯 C/C++ 实现，支持 Q2-Q8 多种 GGUF 格式，门槛最低 |
| **GPTQ-for-LLaMa** | QwopQwop200 | GPU 4bit 量化 | 最成熟的 GPTQ 量化方案，HuggingFace 广泛支持 |
| **AutoAWQ** | MIT TriDAO | GPU 速度优先 | AWQ 自动化实现，2024 年主流选择 |
| **ExLlama / ExLlamaV2** | turboderp | GPU 高效推理 | GPTQ 模型推理速度最优，v2 达 64 tok/s |
| **bitsandbytes** | Tim Dettmers | PyTorch 集成 | NF4/INT8，QLoRA 基础库 |
| **AutoFP8** | Neural Magic | FP8 量化 | 新兴 8-bit 浮点量化方案 |
| **BitBLAS** | Microsoft | 混合精度矩阵运算 | 生产级混合精度推理支持 |
| **VPTQ** | — | 极端低比特 | 2-bit SOTA，2024 新开源 |

### GGUF 量化格式详解（llama.cpp）

| 格式 | 有效位数 | 7B 模型大小 | 质量 | 推荐场景 |
|------|----------|------------|------|----------|
| Q2_K | ~2.5bit | ~2.5GB | 较低 | 极致内存受限 |
| Q3_K_M | ~3.5bit | ~3.3GB | 中等 | 内存紧张 |
| **Q4_K_M** | **~4.5bit** | **~4.1GB** | **良好** | **性价比最优，推荐首选** |
| Q5_K_M | ~5.5bit | ~4.8GB | 优秀 | 高精度需求 |
| Q6_K | ~6.5bit | ~5.5GB | 接近无损 | 精度优先 |
| Q8_0 | 8bit | ~7.2GB | 几乎无损 | 等同原始 |

### 推理引擎对比

| 引擎 | 厂商/团队 | GPU 支持 | 特色 | 2024-2025 更新 |
|------|-----------|----------|------|----------------|
| **vLLM** | UC Berkeley LMSYS | NVIDIA 为主 | PagedAttention，吞吐最高，开源主流 | FP8 支持，AWQ 集成 |
| **TensorRT-LLM** | NVIDIA | NVIDIA 全系列 | 极致推理性能，企业生产 | 新增 INT4/FP8 支持 |
| **MLC-LLM** | TVM Unity / 陈天奇 | CPU+GPU+移动端 | 跨平台统一引擎，支持 iOS/Android/Web | 2024 升级 MLCEngine 统一架构 |
| **gpt-fast** | PyTorch Labs | PyTorch 生态 | 简洁 PyTorch 原生实现 | 活跃开发中 |
| **llama.cpp** | Georgi Gerganov | 全部 | CPU 推理事实标准，低显存 | 持续更新新量化格式 |
| **llama2.c** | Andrej Karpathy | 嵌入式 C | 单文件极简实现，教学/嵌入式 | — |

### Benchmark 实测对比（LLaMA-2-13B, RTX 3090）

> 数据来源：oobabooga benchmark（2024）

| 量化方案 | Perplexity | 显存(GB) | 推理速度(tok/s) | 特点 |
|----------|-----------|----------|----------------|------|
| EXL2-4.65bit | 4.321 | 9.0 | **56.5** | 速度最优 |
| AWQ-4bit-32g | 4.325 | 10.6 | 39.5 | 精度/速度平衡 |
| Q4_K_M (GGUF) | 4.333 | 9.0 | 30.8 | CPU友好，便捷 |
| GPTQ-4bit-act | 4.338 | 8.7 | 42.4 | 显存最低 |
| load_in_4bit (bnb) | 4.364 | 8.2 | 23.1 | 精度损失较大 |

> **结论**：EXL2 在推理速度上领先 147% vs bitsandbytes；Q4_K_M 在 CPU 场景下是事实标准；AWQ 速度与精度平衡最好。

---

## 部署实践

### 场景化部署策略

```
部署场景判断树：
│
├─ GPU 服务器 → 生产推理
│   ├─ 追求速度 → TensorRT-LLM + INT4 AWQ/GPTQ
│   └─ 追求精度 → vLLM + FP16/INT8
│
├─ 消费级 GPU（RTX 3090/4090）→ 个人/研究
│   ├─ 个人助手 → llama.cpp GGUF Q4_K_M（纯 CPU 或 +GPU offload）
│   └─ 微调实验 → QLoRA + bitsandbytes INT4 NF4
│
├─ 手机/移动端 → 端侧部署
│   └─ MLC-LLM + Q4_K_M（iOS/Android）
│
└─ CPU only → 低成本/服务器less
    └─ llama.cpp GGUF Q4_K_M/Q5_K_M
```

### 最佳实践：llama.cpp 量化 + 推理

```bash
# 1. 安装
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp && make -j && make LLAMA_CUBLAS=1 -j

# 2. HuggingFace → GGUF 转换
python convert_hf_to_gguf.py /path/to/llama-2-7b \
  --outfile llama-2-7b-f16.gguf --outtype f16

# 3. 量化
./llama-quantize llama-2-7b-f16.gguf \
  llama-2-7b-q4_k_m.gguf q4_k_m

# 4. 推理（CLI）
./llama-cli -m llama-2-7b-q4_k_m.gguf \
  -p "What is machine learning?" -n 256 --temp 0.7

# 5. 推理（Python）
pip install llama-cpp-python
```

```python
from llama_cpp import Llama

llm = Llama(
    model_path="./llama-2-7b-q4_k_m.gguf",
    n_ctx=2048,
    n_gpu_layers=35,  # GPU offload 层数
)
output = llm("Q: What is model quantization?\nA:", max_tokens=256)
```

### 最佳实践：GPTQ 量化 + vLLM 推理

```python
# GPTQ 量化
from transformers import AutoModelForCausalLM, GPTQConfig

gptq_config = GPTQConfig(
    bits=4,
    dataset="c4",
    group_size=128,
    desc_act=True,  # activation order，更高精度
)
model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-2-7b-hf",
    quantization_config=gptq_config,
    device_map="auto",
)

# vLLM 部署
# vllm serve meta-llama/Llama-2-7b-hf --quantization gptq
```

### QLoRA：量化 + 高效微调

> QLoRA = Quantized LLMs with Low-Rank Adapters，是 2024-2025 最流行的微调范式

```python
from transformers import AutoModelForCausalLM
from peft import LoraConfig
from bitsandbytes import BitsAndBytesConfig

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype="float16",
    bnb_4bit_use_double_quant=True,
)

model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-2-7b",
    quantization_config=bnb_config,
)

lora_config = LoraConfig(r=64, lora_alpha=16, target_modules=["q_proj", "v_proj"])
# 结合 LoRA 微调，仅训练 ~0.5% 参数
```

### 显存估算公式

```
推理显存需求 ≈ (参数量 × 量化字节数) + KV Cache + 激活值

示例：7B 模型 INT4 推理
= (7B × 0.5 字节) + 上下文 4K 的 KV Cache + 激活值
≈ 4GB + ~1GB + ~0.5GB ≈ 5.5GB（RTX 3060 即可运行）
```

### 量化方法选择决策树

```
Q: 部署在哪里？
├─ GPU 生产服务器
│   └─ Q: 追求速度还是精度？
│       ├─ 速度优先 → AWQ INT4
│       └─ 精度优先 → GPTQ INT4 或 vLLM FP16
│
├─ 消费级 GPU
│   ├─ 纯推理 → GGUF Q4_K_M（llama.cpp）
│   └─ 推理+微调 → QLoRA（bitsandbytes）
│
├─ 手机/移动端 → MLC-LLM + Q4_K_M
│
└─ CPU only → GGUF Q4_K_M（llama.cpp CPU 模式）
```

---

## 推荐资源

### 📚 综述论文

| 论文 | 来源 | 年份 | 备注 |
|------|------|------|------|
| A Survey on Model Compression for Large Language Models | MIT TACL | 2024 | ⭐ 最全面的 LLM 压缩综述 |
| LLM Compression for Enhanced Performance: A Comparative Study | IEEE | 2025 | 结构化 vs 非结构化剪枝对比 |
| Compact Language Models via Pruning and Knowledge Distillation | NeurIPS | 2024 | NVIDIA 工业实践 |

### 🛠️ 工具仓库

| 工具 | GitHub | 用途 |
|------|--------|------|
| llama.cpp | `ggerganov/llama.cpp` | CPU/边缘推理，GGUF 格式 |
| vLLM | `vllm-project/vllm` | 高吞吐 GPU 推理服务 |
| TensorRT-LLM | `NVIDIA/TensorRT-LLM` | NVIDIA GPU 极致性能 |
| MLC-LLM | `mlc-ai/mlc-llm` | 跨平台统一部署 |
| AutoAWQ | `casperhansen/autoawq` | AWQ 自动化量化 |
| GPTQ-for-LLaMa | `qwopqwop200/GPTQ-for-LLaMa` | GPTQ 量化 |
| bitsandbytes | `TimDettmers/bitsandbytes` | NF4/INT8 量化 + QLoRA |
| gpt-fast | `pytorch-labs/gpt-fast` | PyTorch 原生高效推理 |
| LLaMA-Factory | `hiyouga/LLaMA-Factory` | 统一微调框架（支持 QLoRA）|
| Awesome-LLM-Compression | `HuangOwen/Awesome-LLM-Compression` | ⭐ 压缩资源合集（1.8k ⭐）|
| Awesome-LLM-Prune | `pprp/Awesome-LLM-Prune` | 剪枝专项资源 |

### 📊 基准测试

| 基准 | 覆盖范围 |
|------|----------|
| WikiText-2/C4/PTB | 语言建模困惑度 |
| LAMBADA / PIQA / OpenBookQA | Zero-shot 能力 |
| GSM8K / CommonsenseQA | 推理能力 |
| BIG-Bench (200+ 任务) | 综合能力 |
| EleutherAI LM Harness (60+ 任务) | 标准评测套件 |
| LLM-KICK | 压缩模型专项评测 |

### 🔗 学习资料

- [oobabooga 量化对比基准](https://oobabooga.github.io/blog/posts/gptq-awq-exl2-llamacpp/)：EXL2/AWQ/GPTQ/GGUF 详细性能对比
- [QubitTool 模型量化完全指南](https://qubittool.com/zh/blog/model-quantization-complete-guide)：从原理到实战
- [LLM 量化方法深度对比](https://cloud.tencent.com/developer/article/2361027)：GPTQ/GGUF/AWQ 中文详解
- [MIT TACL 综述 arXiv](https://arxiv.org/abs/2308.07633)：全面梳理量化/剪枝/蒸馏方法
- [VPTQ arXiv](https://arxiv.org/abs/2409.17066)：2024 低比特量化 SOTA

---

*报告基于 2024-2025 年公开研究成果与实测数据综合整理。技术快速发展，部分数据可能因版本迭代有所变化，建议参考对应工具的最新 Release 说明。*
