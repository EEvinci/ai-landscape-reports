# 04. Fine-tuning & PEFT（微调与参数高效微调）

> 研究时间：2026-04-08 | 研究者：Kiko Sub-Agent #04 | 运行时长：3分52秒

---

## PEFT 技术演进

### LoRA 家族演进

| 方法 | 年份 | 核心创新 | 关键优势 |
|------|------|---------|---------|
| **LoRA** | 2021 | 低秩分解 ΔW=BA，参数量减少99%+ | 推理零开销，生态成熟 |
| **QLoRA** | 2023 | 4-bit NormalFloat + 双重量化 + 分页优化器 | 单卡可微调65B模型 |
| **AdaLoRA** | 2023 | SVD动态分配秩预算 | 自适应重要层多调，普通层少调 |
| **DoRA** | 2024 ICML Oral | 权重分解为 magnitude + direction 独立调节 | Llama 7B 提升+3.7%，推理零开销 |
| **LoRA+** | 2024 | A/B 矩阵使用不同学习率 | 收敛更稳定 |
| **LoftQ** | 2023 | 交替量化与 SVD 分解 | 量化与低秩适配协同 |
| **QDoRA** | 2024 | DoRA + 量化压缩 | 压缩模型上超越 FT 和 QLoRA |
| **BitDelta** | 2024 | 1-bit 权重差分量化 | 极低成本微调 |

**2025 年主流格局**：LoRA 是底座；QLoRA 打开消费级大门；**DoRA 是免费升级**，推荐优先使用。

---

## 微调 vs RAG vs Prompt 选择框架

### 效果与成本量化对比

| 方案 | 准确率区间 | 月均成本（估算） |
|------|-----------|----------------|
| Prompt Engineering | 75-85% | $0-50 |
| RAG | 88-94% | $200-2000 |
| PEFT (LoRA) | 90-96% | $500-5000 |
| 全量微调 | 92-97% | $5000-50000+ |

**最优实践**：Fine-tuned 模型 + RAG + Prompt Engineering 混合架构

### 何时选谁

| 条件 | 推荐 |
|------|------|
| 知识需要实时更新 | RAG |
| 需要改变模型行为/风格 | Fine-tuning |
| 基础模型能力足够 | Prompt Engineering |
| 消费级显卡微调大模型 | QLoRA |
| 追求最佳性能 | DoRA / QDoRA |

---

## 训练数据构建

### 核心原则
> **数据质量 > 数据数量**（业界公认）

### 流程
1. 数据采集（领域文档/内部知识库/公开数据集）
2. 清洗（去重 + 质量过滤 + 毒性过滤）
3. 标注（人工/LLM生成/蒸馏/众包）
4. 配比（通用30-40% + 领域40-50% + edge case 10-20%）
5. 评估（Reward Model打分 + KL散度检验 + 人工抽检）

### SFT 数据格式
```json
{"messages": [
  {"role": "system", "content": "你是一个专业的法律顾问..."},
  {"role": "user", "content": "合同违约金上限是多少？"},
  {"role": "assistant", "content": "根据《民法典》第五百八十五条..."}
]}
```

---

## 主流平台对比

| 平台 | 特点 | 适用场景 |
|------|------|---------|
| **OpenAI Fine-tuning** | 全托管，REST API，最少50条数据 | 快速 API 集成 |
| **Google Vertex AI** | VPC隔离，IAM权限，多模态支持 | 企业级 |
| **AWS SageMaker** | JumpStart一键部署，分布式训练 | 已有 AWS 基础设施 |
| **Modal** | Serverless GPU，按秒计费 | 灵活开发，成本敏感 |
| **Unsloth** | 加速2-5x，节省50%显存 | LoRA 训练加速 |

---

## 实战建议

### LoRA/DoRA 超参经验值

| 模型规模 | 推荐 Rank | Alpha | Target Modules |
|------|---------|-------|---------------|
| 7B | 16-32 | 32-64 | q_proj, v_proj |
| 13B | 32-64 | 64 | q_proj, v_proj |
| 70B | 64-128 | 128 | q/v/k_proj |

### 避坑清单
- ❌ 数据量不是越大越好，质量差会拉低模型
- ❌ 不要全量微调能用 LoRA/QLoRA 的场景
- ❌ LoRA rank 不是越大越好，超过一定值收益递减
- ✅ 永远保留验证集，从 eval loss 判断停止时机
- ✅ 微调后必须评估：目标任务提升 vs 非目标任务退化

---

## 推荐资源

### 开源工具
- [Hugging Face PEFT](https://github.com/huggingface/peft) — 20+ PEFT 方法统一接口
- [trl](https://github.com/huggingface/trl) — SFT/DPO/PPO 训练 Pipeline
- [Axolotl](https://github.com/OpenAccess-AI-Collective/axolotl) — 支持 LoRA/DoRA/QLoRA/全量
- [LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory) — 中文友好，80+模型支持
- [Unsloth](https://unsloth.ai) — LoRA 训练加速 2-5x

### 核心论文
- LoRA 原论文（arXiv:2106.09685）⭐ 必读
- QLoRA（arXiv:2305.14314）⭐ 必读
- DoRA（ICML 2024, arXiv:2402.09353）⭐ 推荐
- PEFT Survey（AI Review 2025）— 覆盖100+论文，全面综述
