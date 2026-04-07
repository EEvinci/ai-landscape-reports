# AI Infrastructure & MLOps 研究报告

> 研究日期：2026-04-08 | 覆盖范围：AI基础设施、MLOps全流程、实验追踪、模型部署、观测性、开源工具生态

---

## 一、AI 基础设施：算力供给格局

### 1.1 GPU 云服务格局

AI 计算的核心基础设施仍是 GPU 云服务，2025-2026年呈现"云厂商主导 + 专业化玩家细分"的竞争格局。

**头部云厂商：**

- **AWS**：通过 SageMaker 提供 P4d（P4d.24xlarge 配备 4×A100 40GB）、P5 实例（H100），并推出 SageMaker HyperPod 面向大规模训练。EC2 UltraClusters 支持跨多可用区的高速 RDMA 互联。SageMaker 本身提供端到端 MLOps 能力，从数据标注到模型部署全流程覆盖。
- **Google Cloud**：TPU 是其差异化优势，最新 TPU v5e 提供 256-chip pods，v5p 面向大模型训练峰值算力，配合 Vertex AI 平台形成完整的端到端 MLOps 生态。Vertex AI 支持定制化模型训练、AutoML 和模型部署，并内置实验追踪功能。
- **Microsoft Azure**：ND H100 v5 实例支持大规模训练，与 OpenAI 深度整合；Azure ML 提供企业级 MLOps 工作流，支持 Designer 可视化建模和 Code-based 两种开发模式。
- **Lambda Labs / CoreWeave**：专业 AI 云服务商，提供 H100、B100 集群，配置灵活、定价透明，成为中小型 AI 公司的首选算力来源。Lambda 提供预配置 Jupyter Notebook 环境，降低实验启动门槛。

**算力租赁平台（灵活定价模式）：**

- **Vast.ai**：以竞价拍卖模式提供低成本 GPU 租赁，A100 80GB 可低至 $1.5-2.0/hr，面向独立研究者和小团队。提供 Docker 容器一键部署，支持 RTX 4090、A100、H100 等多种配置。
- **GPUs.io**：聚合 17+ 云厂商的实时价格比较工具，帮助用户找到最优性价比配置，覆盖 Vast.ai、Paperspace、Lambda 等主流平台。

### 1.2 TPU 与 NPU 生态

- **TPU（Tensor Processing Unit）**：Google 自研，专为矩阵运算优化，v5e 性价比突出，主要在 Google Cloud 独占使用，配套 JAX/Flax 框架。适合大规模 Transformer 训练，但锁定 Google 生态，换用成本高。
- **NPU（Neural Processing Unit）**：华为昇腾 NPU（Ascend 910B/910C）在中国市场快速崛起，PyTorch 已原生支持 NPU 适配（vLLM-Ascend 分支），MindIE 推理引擎逐步成熟。美国出口管制加速了国产 NPU 的研发与部署节奏，寒武纪、摩尔线程等国产芯片配套软件栈日趋完善。
- **边缘 AI**：Intel NPU（集成于 Meteor Lake / Lunar Lake 处理器）、Apple Neural Engine (ANE)、Qualcomm AI Hub 推动推理向端侧迁移。AWS Inferentia2、Google Edge TPU 提供边缘推理节点，适用于低延迟、隐私敏感场景。

### 1.3 算力趋势与前沿动态

多芯异构（CPU + GPU + NPU 混合调度）成为大模型训练事实标配；HBM 带宽优化是推理芯片竞争焦点，H200 提供 4.8TB/s 带宽，比 H100 高 2.4 倍。算力租赁市场年增速超 40%，去中心化算力（如 Fluence）开始探索共享经济模式。

**2025年值得关注的格局变化：**
- 英伟达 H200/B100 供货逐步改善，B200（Blackwell Ultra）进入早期部署，单卡算力持续攀升
- AMD Instinct MI300X 在部分超算场景形成竞争，成本优势开始显现，适合对性价比敏感的场景
- 国产 GPU（摩尔线程、天数智芯、寒武纪）性能持续追赶，适配 PyTorch、vLLM 等主流开源框架
- Kubernetes 上的 GPU 调度（via Volcano / Gang Scheduling）成熟度提升，多任务共享 GPU 集群成为可能

---

## 二、MLOps 全生命周期

MLOps（Machine Learning Operations）是一套将 DevOps 原则应用于机器学习系统的工程实践，核心目标是实现 ML 系统的可靠、可重复、可扩展的持续交付。

```
数据管理 → 实验追踪 → 模型注册 → 部署监控 → 持续迭代
```

### 2.1 数据管理

数据是 ML 系统的地基。MLOps 要求：数据版本控制（DVC / Delta Lake / Quilt）、数据质量验证（Great Expectations / TFX Data Validation）、数据血缘追踪（Apache Atlas / OpenMetadata）、特征存储（Feast / Tecton）。数据漂移是生产失效的首要原因之一，需在数据入口即建立监控。

### 2.2 实验追踪

将每次训练运行（Run）的参数、指标、产物（模型文件、图表）完整记录，实现可复现性（Reproducibility）。核心价值：横向对比不同实验版本、找到最优超参组合、沉淀可复用知识。好的实验追踪工具能将一个研究 idea 到生产模型的转化效率提升数倍。

### 2.3 模型注册

模型注册表（Model Registry）管理模型版本生命周期：None → Staging → Production，支持模型元数据记录（训练数据集、评估指标、所属团队），并与 CI/CD 管道打通，实现自动化晋升（Promotion）流程。配合 GitHub Actions 或 Jenkins，可实现 Push-to-Deploy 的自动化模型发布。

### 2.4 部署监控

包括推理延迟、吞吐量、GPU 利用率、错误率等基础设施指标，以及业务层指标（预测准确率、转化率等）。发现问题后触发自动化重训练管道，形成 MLOps 闭环的最后一环。

---

## 三、实验追踪工具深度对比

### 3.1 MLflow

Databricks 出品的开源实验追踪平台，Apache 2.0 许可，是事实上的行业标准，被全球数万个 ML 团队使用。

**优势：**
- 完全开源，可自托管，无厂商锁定，生态最成熟
- 覆盖全生命周期四大组件：追踪（Tracking）+ 单元（Projects）+ 模型管理（Registry）+ Serving
- 与 Spark 深度集成，企业级数据栈友好，适合大数据场景
- Model Registry 支持 Staged Rollout（None→Staging→Production）三阶段发布模型
- 可配合 GitHub Actions 实现自动模型晋升，结合 CI/CD 管道实现全自动化

**局限：**
- UI 功能相对基础，团队协作体验弱于商业竞品
- 大规模实验过滤和搜索效率一般，缺少高级可视化功能

### 3.2 Weights & Biases (W&B)

专注研究团队的商业化实验追踪工具，以体验流畅和可视化强大著称，在学术和 AI 研究社区有极高渗透率。

**优势：**
- 极简集成（`wandb.init` 一行代码），自动捕获 GPU/CPU 使用情况、GPU 显存、温度等硬件指标
- 强大的可视化面板：参数重要性、热力图、超参空间可视化（Parallel Coordinates）
- 团队协作体验优秀，支持 Report（类 Notebook 分享），方便研究员向非技术干系人展示结果
- Sweeps（超参搜索）与 Report 联动，适合学术/研究场景
- 2025年强化了 LLM 训练追踪（生成轨迹、Token 使用量追踪、Chat Eval）

**局限：**
- 免费版功能受限，商业版按成员计费（$20/人/月起）
- 数据存储在 W&B 云端，存在数据主权顾虑（支持自托管 W&B Server，但配置复杂）

### 3.3 Neptune.ai

主打"统一 Metadata 存储"的实验追踪平台，定位介于开源与商业之间，2025年持续强化 LLM 场景支持。

**优势：**
- 支持与 25+ MLOps 工具集成（MLflow、Feast、Kubeflow、Seldon 等），实现 metadata 统一汇聚
- Runs 之外还存储模型元数据、数据集版本、监控指标，一站式管理 ML 资产
- 轻量 API，灵活查询，团队权限管理成熟，支持细粒度 RBAC
- 适合需要跨工具整合的中大型 ML 团队，提供 SaaS 与私有化部署两种模式

**局限：**
- 品牌认知度低于 W&B，社区资源和第三方教程相对少
- UI 自定义能力有限，不支持高度定制化 Dashboard

### 3.4 Google Vertex AI Experiments

Google 云上的企业级实验追踪服务，适合深度绑定 GCP 生态的团队，在 Vertex AI 平台上统一管理。

**优势：**
- 与 Vertex AI Training、Prediction、Model Registry 深度集成，从实验到部署一气呵成
- 支持大规模并行实验（数百个并行 Trials），适合超参搜索和神经网络架构搜索（NAS）
- AutoML 能力内置，从实验追踪无缝衔接 AutoML，非 ML 专家也能训练高质量模型
- 基于 Vizier（Google 内部超参优化系统，Bayesian Optimization 实现）的超参调优

**局限：**
- 强绑定 GCP，换云成本极高
- 使用门槛较高，界面偏向工程师而非研究员

### 对比小结

| 维度 | MLflow | W&B | Neptune | Vertex AI |
|------|--------|-----|---------|-----------|
| 部署方式 | 开源自托管 | SaaS/自托管 | SaaS/自托管 | 仅 GCP 云服务 |
| 协作体验 | 一般 | 优秀 | 良好 | 良好 |
| LLM 追踪 | 基础 | 优秀 | 良好 | 良好 |
| 生态整合 | 最广 | 一般 | 最广 | 仅 GCP |
| 适合场景 | 企业/开源 | 研究团队 | 混合团队 | GCP 深度用户 |

---

## 四、模型部署：推理服务框架

### 4.1 TorchServe

PyTorch 官方模型服务框架，是将 PyTorch 模型部署到生产环境的首选方案，2025年全面拥抱 LLM 场景。

**核心能力：**
- RESTful / gRPC API，兼容标准 ML 推理场景，提供健康检查和批量推理端点
- 多模型并行加载、自动批处理（A/B/C 轮询调度 Worker），支持模型版本管理
- 2024-2025版本新增 LLM 推理支持：通过集成 vLLM 后端，可一键部署 Llama、Mistral 等大语言模型
- 与 Kubernetes / KServe 原生集成，支持 Istio 流量治理，可实现金丝雀发布
- 自定义 Handler 机制允许开发者实现预处理/后处理逻辑，无需修改模型本身

**适用场景：** PyTorch 模型（尤其非 Transformer 类：推荐系统、目标检测、传统 ML）的标准化部署；LLM 场景下作为 vLLM 的补充封装层。

### 4.2 TensorFlow Serving (TF-Serving)

Google 维护的 TF 官方推理服务器，成熟稳定但在 LLM 时代增长趋缓。

**核心能力：**
- 高性能 C++ 引擎，支持模型热更新（无需重启服务），Batching Scheduler 优化吞吐量
- gRPC + REST 双协议，监控指标内置（Prometheus 端点），OpenTelemetry 追踪集成
- 支持模型版本管理、多模型同时服务，基于 SavedModel 格式
- 与 TensorFlow Extended (TFX) 生态无缝集成，适合已有 TF 资产的团队

**局限：** 2025年后 TF 生态向 JAX/Flax 迁移，TF-Serving 更新频率明显下降，新增 LLM 支持极其有限。

### 4.3 vLLM — LLM 推理的事实标准

2023年由 UC Berkeley 推出，以 PagedAttention 彻底改变了 LLM 推理效率，2025年已迭代至 v1 架构并实现 PD 分离（Prefill/Decode 分离）支持。

**核心技术：**
- **PagedAttention**：借鉴操作系统虚拟内存分页管理，将 KV Cache 非连续存储，将 GPU 显存利用率提升至 90%+（传统方法仅 30-40%）
- **连续批处理（Continuous Batching）**：动态将不同请求打包执行，消除空泡（GPU Bubble），吞吐量提升 2-4 倍
- **FP8 量化**：在 H100 上原生支持，近 INT8 效率但精度损失更小
- **多节点支持**：通过 Ray 可扩展至多节点集群，支持最大 405B 参数模型

**性能表现（对比参考）：**
- 吞吐量比 naive 方案高 2-4 倍
- 混合长度负载场景比 HuggingFace TGI 高 10-30%
- 推理延迟 Python 开销仅 1-5ms（对 LLM 整体响应时间可忽略）
- vLLM v1 支持 PD 分离架构，Prefill 和 Decode 分别在独立 GPU 集合执行，优化资源利用率

**部署体验：** `pip install vllm` 后一行命令启动，OpenAI 兼容 API，开源社区最活跃，新模型架构支持最快，支持 SGLang Runtime 等多种后端。

### 4.4 Triton Inference Server

NVIDIA 出品的通用推理服务器，2025年以 TensorRT-LLM 作为 LLM 推理核心，性能领先。

**核心能力：**
- **TensorRT-LLM**：对 Transformer 结构进行算子融合、内核优化，在 H100 上比 vLLM 高 20-40% 吞吐量
- **通用性**：同时支持 LLM、CV 模型、Embedding 模型、多模态模型的统一服务
- **多节点支持**：最成熟的分布式推理方案（Tensor + Pipeline 并行），支持多模型并发服务
- **动态批处理 + 预热**：推理前自动 Warm-up 编译，支持 FP8 原生量化在 Hopper 架构 GPU

**局限：** TensorRT 引擎编译耗时（30分钟至数小时），模型更新流程重；学习曲线陡峭，需要 CUDA/C++ 背景。

### 部署选型建议

| 场景 | 推荐方案 |
|------|---------|
| LLM 通用推理，首选易用性 | **vLLM** |
| HuggingFace 生态优先 | **TGI（Text Generation Inference）** |
| H100 集群，最大化吞吐 | **Triton + TensorRT-LLM** |
| PyTorch 传统 ML 模型 | **TorchServe** |
| TF 模型 + 成熟运维体系 | **TF-Serving** |

---

## 五、观测性：模型监控与质量保障

### 5.1 模型漂移检测（Model Drift Detection）

模型漂移是生产 ML 系统最核心的风险，分为三种类型：

- **数据漂移（Data Drift）**：输入数据分布发生变化，如用户行为模式改变导致特征值漂移
- **概念漂移（Concept Drift）**：特征与标签的映射关系发生变化，如季节性因素影响推荐模型
- **预测漂移（Prediction Drift）**：模型输出分布本身发生变化，可作为前两者的先导指标

**主流工具对比：**

| 工具 | 类型 | 核心能力 | 2025 更新 |
|------|------|---------|---------|
| **Evidently AI** | 开源 | Data/Concept Drift、回归/分类性能报告，Prometheus 集成 | 新增实时告警与 Evidently Cloud |
| **Fiddler AI** | 商业 SaaS | 生产漂移追踪、根因分析、合规审计 | 支持 GenAI 监控和 LLM 输出评估 |
| **SageMaker Model Monitor** | 云服务 | 自动统计检验（KS/CVT），无需编码 | 支持数据质量基准 |
| **Arize AI** | 商业 SaaS | 多框架支持、性能追踪、集成追踪 | 支持 LLM 幻觉追踪和 RAG 评估 |

**推荐方法论：** 采用 KL 散度（连续特征）+ Population Stability Index（PSI，综合评分）+ 每请求实时监控的多层策略，金融场景阈值建议 KL < 0.05，通用场景可放宽至 0.2。

### 5.2 数据质量监控

数据质量是模型质量的先决条件，核心监控维度：

- **完整性**：缺失值比例、空值检测、特征覆盖度
- **一致性**：Schema 校验、类型一致性、参照完整性
- **时效性**：数据延迟监控、特征新鲜度（Feature Freshness）、数据管道 SLA
- **统计稳定性**：分布检验（PSI、KL Divergence、Wasserstein Distance）

**推荐架构：Great Expectations（数据验证）+ Evidently AI（漂移检测）+ Prometheus（指标收集）+ Grafana（可视化告警）**，四者协同实现从数据入口到业务指标的端到端可观测性。

### 5.3 A/B 测试框架

模型上线的最后一道质量门，A/B 测试框架需要关注：

- **流量分层**：确保实验组与对照组流量隔离，避免 Traffic Leak
- **统计显著性**：样本量计算、p-value 监控（避免假阳性），使用 Sequential Testing 减少实验等待时间
- **渐进式 Rollout**：金丝雀发布（Canary）+ 特征开关（Feature Flags），从 1% → 5% → 20% → 100% 分阶段放量

**工具选型：**
- **LaunchDarkly / Unleash**：Feature Flag 管理，支持渐进式灰度、AB 实验、Dark Launch
- **Optimizely**：传统 AB 测试平台，2025年强化了 ML 场景支持（模型性能 AB 测）
- **内部灰度平台**：大厂通常自建（如 Uber 的 Experiment Platform），中小团队可用 Flagsmith

---

## 六、开源 MLOps 工具生态全景对比

### 6.1 Kubeflow

CNCF 毕业项目，Kubernetes 上的 ML 编排标准，功能最全但复杂度也最高，是大规模生产 ML 的主流选择。

**核心组件：**
- **Kubeflow Pipelines (KFP v2)**：DAG 编排，支持 Steps 间数据传递（InputPath/OutputPath），可视化 Pipeline 编辑器，基于 Argo Workflows
- **Katib**：超参数调优（基于 Optuna/TensorFlow Estimator），支持随机搜索、贝叶斯优化和 Grid Search
- **KServe**：模型推理服务（支持 vLLM、Triton、TorchServe 多后端），Serverless 自动扩缩容（KEDA 集成）
- **Training Operators**：分布式训练（PyTorchJob、TFJob、MPIJob），支持多节点 GPU 集群

**优势：** 功能最完整，Kubernetes 原生，企业级可扩展，与 Istio/Prometheus/Grafana 生态无缝集成。

**挑战：** 安装复杂（10+ 组件），资源占用高，KFP DSL 学习曲线陡，新版本 API 变化频繁，需要专职 SRE 维护。

**适用场景：** 有 Kubernetes 团队支撑的大规模生产 ML 平台，中小团队慎入。

### 6.2 Seldon

专注模型部署与服务化的 MLOps 平台，基于 Kubernetes，但比 Kubeflow 轻量，专注推理价值。

**核心能力：**
- **Seldon Core**：将任意 ML 模型（sklearn/xgb/tensorflow/pytorch）封装为 REST/gRPC API，3行 YAML 配置即可完成部署
- **Seldon Deploy**：企业级模型部署管理平台（含 UI、监控、Rollback/Roll-forward 能力）
- **Alibi**：模型可解释性库（SHAP、LIME、CFE），满足欧盟 AI Act 合规要求
- **Ambassador**（Envoy-based）：流量管理，支持 Shadow Mode、A/B Testing、Canary Release

**优势：** 比 Kubeflow 轻量，专注推理，集成 Istio/Envoy 流量治理，企业版提供 SLA 支持。

**挑战：** 开源版功能有限，高级功能（多团队、审计日志）需企业版订阅。

### 6.3 Tempo

ZenML 团队出品的 MLOps 框架，专注"多框架统一推理"部署，核心理念是降低推理部署门槛。

**核心哲学：** 一次编写，Tempo 负责推理部署到任意后端（Seldon、KServe、BentoML、Ray Serve），让数据科学家专注建模而非 DevOps。

**优势：**
- 声明式推理配置，`@pipeline` 装饰器统一建模，开箱即用的生产级推理
- 与 MLflow、W&B、Evidently 无缝集成，metadata 自动汇聚
- 支持 Shadow Deployment、A/B Testing、Multi-Armed Bandit 等高级流量策略
- 自动生成 API 文档和模型卡（Model Card），降低交接成本

**局限：** 项目较新（2021年起步），社区规模有限，文档质量有待提升，生产案例积累不足。

### 6.4 ZenML

Python-first 的 MLOps 框架，以"框架无关 + 云原生"为核心定位，2025年进入快速增长期，在 GitHub 上获得大量关注。

**核心能力：**
- **Stacks 概念**：将 Orchestrator（任务编排）、Step Operator（步骤执行器）、Model Deployer（模型部署）解耦组合，一套代码切换不同后端（Kubeflow、Airflow、Vertex AI、AWS SageMaker 等 40+ 后端）
- 内置 MLflow、W&B、Evidently、Great Expectations 集成，开源免费（Apache 2.0）
- 专注 ML Pipeline 编排，不强制绑定特定推理框架或云厂商，团队可逐步演进技术栈
- 提供 `zenml model deployer` 插件，自动化将模型部署到 KServe/Seldon/Triton

**优势：** 上手门槛低（纯 Python API），灵活切换后端，云厂商中立，2025年获大量种子用户。

**挑战：** 企业级功能（RBAC、审计日志）仍在完善，超大规模使用案例较少。

### 综合对比

| 维度 | Kubeflow | Seldon | Tempo | ZenML |
|------|----------|--------|-------|-------|
| 定位 | 全栈平台 | 推理服务 | 多框架推理 | Pipeline 编排 |
| 复杂度 | 高 | 中 | 低 | 低 |
| Kubernetes 依赖 | 必须 | 必须 | 可选 | 可选 |
| 主要优势 | 功能最全 | 推理生态 | 推理抽象 | 灵活性与中立性 |
| 主要挑战 | 运维负担 | 商业化锁定 | 社区小 | 企业功能弱 |
| 适合规模 | 大型 | 中大型 | 中型 | 小中大型 |
| 2025 趋势 | 成熟稳定 | 与 KServe 整合 | 生态扩展 | 快速成长 |

---

## 七、趋势与展望

**2025-2026年 MLOps 核心趋势：**

1. **LLM 原生 MLOps**：MLOps 工具链全面拥抱 LLM 场景，实验追踪增加 Token 追踪，推理服务以 vLLM/TGI 为主流后端，Kubeflow KServe 支持 LLM 部署。
2. **PD 分离（Prefill/Decode 分离）**：大模型推理架构演进，Prefill 节点与 Decode 节点独立扩缩容，vLLM v1 原生支持该架构，实现算力成本优化。
3. **MLOps 平台整合**：Dataiku、Databricks、AWS SageMaker 等大平台持续整合 MLOps 能力，独立工具面临整合或专业化两条出路。
4. **可观测性左移**：监控从"部署后"前移到"训练中"，Evidently AI + MLflow 联动实现训练期漂移预检，降低生产风险。
5. **国产化替代加速**：昇腾 NPU + MindIE + 飞桨生态在中国企业加速落地，形成独立于 CUDA 的推理栈；同时开源推理框架 vLLM 已完成昇腾适配。
6. **自动化机器学习（AutoML）深化**：HPO 从实验工具演变为 MLOps Pipeline 内置能力，Vertex AI、Katib、Optuna 在生产环境中承担更核心角色。

**选型决策树：**

```
团队规模 / 需求
    │
    ├── 小团队 / 研究原型
    │       → MLflow（追踪）+ vLLM（推理）+ ZenML（Pipeline）
    │
    ├── 中型团队 / 多框架协作
    │       → W&B（追踪）+ vLLM/TGI（LLM推理）+ Seldon（服务）+ Evidently（监控）
    │
    └── 大型企业 / 平台级建设
            → Kubeflow（编排）+ Triton+TensorRT-LLM（推理）+ Vertex AI（平台）+ Evidently+Prometheus/Grafana（可观测性）
```

**一句话总结：** MLOps 工具链在 2025-2026 年已进入"LLM 优先"阶段，选择工具时优先考虑 LLM 支持成熟度、团队技术栈匹配度以及长期运维成本，而非功能数量堆砌。vLLM 已成为 LLM 推理的共识选择，MLflow 仍是企业级开源实验追踪的首选，ZenML 是中小团队快速构建 MLOps 能力的最佳入口。

---

*本报告基于 2025-2026 年公开信息整理，数据来源包括 MLflow 官方文档、vLLM 项目主页、NVIDIA Triton 文档、Evidently AI 技术博客、ZenML/Tempo 官方 GitHub 及各云厂商官网。*
