# 14. Vector Database

> 向量数据库是专为存储和检索高维向量嵌入（Embedding）设计的数据库，是 RAG（检索增强生成）、语义搜索、推荐系统的核心基础设施。2024-2025 年，随着大模型应用的爆发，向量数据库赛道进入高速增长期，市场规模预计 2028 年超过 43 亿美元。

---

## 核心概念

### 什么是向量嵌入？

向量嵌入是深度学习模型（神经网络）的输出数据格式——将文本、图像、音频、视频等非结构化数据，映射为高维空间中的稠密向量（通常 768~1536 维），语义相似的内容在向量空间中距离更近。

### 向量数据库的核心能力

| 能力 | 说明 |
|------|------|
| **向量存储与索引** | 高效存储数十亿级向量，支持多种索引算法 |
| **ANN 搜索** | 近似最近邻搜索，毫秒级返回 Top-K 相似结果 |
| **元数据过滤** | 在向量检索基础上支持标量条件过滤 |
| **混合搜索** | 结合稀疏向量（BM25/TF-IDF）与稠密向量（Embedding） |
| **多模态检索** | 跨文本、图像、音频、视频等多模态统一检索 |
| **实时更新** | 支持向量插入、删除、更新 |

### 典型应用场景

- **RAG（检索增强生成）**：为 LLM 提供私有知识库检索
- **语义搜索**：超越关键词的语义理解搜索
- **推荐系统**：基于向量相似度的个性化推荐
- **图像/视频检索**：以图搜图、视频相似匹配
- **异常检测**：向量空间中离群点检测
- **代码搜索**：语义级代码检索

---

## 主流产品对比（2024-2025）

> 注：2024-2025 年，**Milvus 3.0 尚未正式发布**，当前最新稳定版为 **Milvus v2.6.x**。Zilliz 已预告 3.0 路线图，主打云原生重构与性能大幅提升，预计 2025 年下半年发布。

### 产品一览表

| 产品 | 类型 | 开源 | 2024-2025 关键版本 | 推荐场景 |
|------|------|:----:|------|------|
| **Pinecone** | 托管云服务 | ❌ | Serverless v2, 混合搜索增强 | 快速上线、无运维团队 |
| **Milvus** | 自托管 + 云 | ✅ | v2.6.x（3.0 路线图中） | 亿级向量企业级场景 |
| **Weaviate** | 自托管 + 云 | ✅ | v1.24（2024-2025） | 复杂过滤、GraphQL、混合搜索 |
| **Chroma** | 自托管 + 云 | ✅ | v1.0.0（2025.04）| AI 原生应用、轻量级 |
| **Qdrant** | 自托管 + 云 | ✅ | v1.7+（持续迭代）| 高性能、低延迟、自托管 |
| **FAISS** | 算法库（非数据库）| ✅ | Meta 持续维护 | 极致性能、嵌入式场景 |

### 1. Pinecone

**定位**：全托管向量数据库，主打零运维、快速上线。

| 指标 | 数据 |
|------|------|
| P95 延迟 | <50ms（100 万向量） |
| QPS | 5,000~10,000 |
| 内存占用 | ~4GB（100 万×768 维） |
| 定价 | 免费层 10 万向量；付费 $70~200/月（1000 万向量）|

**2024-2025 更新亮点：**
- **Serverless v2**：完全托管，弹性扩缩，按查询计费，降低中小团队成本
- **Sparse Vector 支持**：原生支持稀疏+稠密混合搜索，无需外部 BM25
- **Multi-index Namespaces**：逻辑隔离多租户场景
- **Metadata Filtering**：支持复杂标量过滤条件

**优势**：5 行代码即可上线，无需任何基础设施运维，延迟低，生态成熟。
**劣势**：闭源，供应商锁定，成本随规模增长显著（千万级 $200~400/月）。

---

### 2. Milvus

**定位**：开源企业级向量数据库，LF AI Foundation 项目，专注超大规模（十亿~万亿级）。

| 指标 | 数据 |
|------|------|
| P95 延迟 | 50~80ms（100 万向量） |
| QPS | 10,000~20,000 |
| 内存占用 | ~4GB（100 万×768 维） |
| 定价 | 自托管免费；Zilliz Cloud $100+/月 |

**2024-2025 更新亮点（v2.6.x）：**
- **Milvus Lite**：轻量级 Python 包，`pip install` 即可使用，无需 Docker
- **多索引类型**：原生支持 HNSW、IVF-PQ、DiskANN、ANNOY 等多种 ANN 索引
- **动态 Schema**：灵活的数据模型，支持动态字段
- **多模态 RAG**：集成 ColQwen2 等多模态模型，支持图像+文本混合检索
- **3.0 路线图预告**：云原生重构、分布式计算引擎升级、意图是挑战 Pinecone 的易用性

**优势**：水平扩展能力强，多种索引算法，适合超大规模数据，企业级特性完善。
**劣势**：部署复杂（需 Kubernetes），学习曲线陡峭，运维成本高。

---

### 3. Weaviate

**定位**：开源 AI 原生向量数据库，以 GraphQL API 和内置向量化著称。

| 指标 | 数据 |
|------|------|
| P95 延迟 | 50~70ms（100 万向量） |
| QPS | 3,000~8,000 |
| 内存占用 | ~3.5GB（100 万×768 维） |
| 定价 | 自托管免费；云端 $25/月起 |

**2024-2025 更新亮点（v1.24，2024）：**
- **Binary Quantization (BQ)**：二进制量化，大幅降低内存占用同时保持高召回率
- **导入/更新/删除性能提升**：批量处理效率显著提升
- **内置向量化模块**：集成 OpenAI、Cohere、HuggingFace 等向量化模型，开箱即用
- **混合搜索成熟**：向量 + BM25 混合搜索体验业界领先
- **GraphQL + REST 双 API**：灵活查询，开发者友好

**优势**：开箱即用的向量化、GraphQL 直观查询、混合搜索成熟、社区活跃。
**劣势**：GraphQL 对部分开发者有学习成本，自托管需 DevOps 经验。

---

### 4. Chroma

**定位**：专为 AI 应用设计的开源向量数据库，以极简 API 和开发者体验著称。

| 指标 | 数据 |
|------|------|
| P95 延迟 | 视部署方式 |
| 核心 API | 仅 4 个函数 |
| 定价 | 自托管免费；Chroma Cloud 按量计费 |

**2024-2025 更新亮点：**
- **v1.0.0（2025 年 4 月）**：首个正式稳定版，API 全面稳定
- **Sparse Vector Search（2025.10）**：新增稀疏向量搜索能力，挑战纯 Embedding 模式
- **Distributed Chroma: BYOC（2026.02）**：分布式部署支持 bring-your-own-cloud，企业级里程碑
- **WAL3（Write-Ahead Log，2025.09）**：可靠性大幅提升，支持崩溃恢复
- **Serverless 模式（2024.07）**：对象存储支持，降低冷存储成本
- **Context-1 模型研究（2026）**：发布自研 Embedding 模型 Chens-1
- **Embedding Adapters**：支持动态调整 Embedding 行为，无需重新索引

**优势**：API 极简（4 个函数），Python/JS 双语言，LangChain 官方集成，入门门槛最低。
**劣势**：分布式能力较新，生态比 Milvus/Qdrant 小，规模上限有限。

---

### 5. Qdrant

**定位**：Rust 编写的开源向量搜索引擎，以极致性能和精确控制著称。

| 指标 | 数据 |
|------|------|
| P95 延迟 | **30~40ms**（100 万向量，业界领先） |
| QPS | 8,000~15,000 |
| 内存占用 | ~3GB（含量化，内存降低 4 倍）|
| 定价 | 自托管免费；Cloud $30/月起 |

**2024-2025 更新亮点（v1.7+）：**
- **Quantization 原生支持**：int8/SQ8 量化，4 倍内存降低同时保持高召回
- **Payload Index**：高效的标量过滤索引，元数据过滤性能优异
- **动态 Segment**：写入和搜索并行，无全量重建
- **FastEmbed**：内置向量化库，无需依赖外部模型服务
- **Sparse Vector 支持**：支持 ColBERT 等多向量检索模型
- **Snapshot & Backup**：生产级备份恢复

**优势**：Rust 实现性能卓越，内存效率高，文档质量优秀，自托管成本极低。
**劣势**：生态系统比 Pinecone/Milvus 小，托管云服务相对较新。

---

### 6. FAISS（Bonus）

**定位**：Facebook AI Similarity Search，**不是数据库**，是 Meta 维护的 C++/Python 向量索引库。

| 指标 | 数据 |
|------|------|
| P95 延迟 | **10~20ms**（内存模式，极致） |
| QPS | 20,000~50,000 |
| 内存 | 纯内存， 受 RAM 限制 |

**适用场景**：数据量 < 1000 万向量，追求极致性能，且有能力自行封装 CRUD/API 层的团队。FAISS 本身无持久化、无高可用，需配合外部存储使用。

---

### 横向综合对比

| 维度 | Pinecone | Milvus | Weaviate | Chroma | Qdrant | FAISS |
|------|:--------:|:------:|:--------:|:------:|:------:|:-----:|
| **托管云服务** | ✅ 优 | ✅ Zilliz | ✅ | ✅ | ✅ | ❌ |
| **自托管** | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **开源协议** | 闭源 | Apache 2.0 | BSD-3-Clause | Apache 2.0 | Apache 2.0 | Apache 2.0 |
| **延迟（1M向量）** | <50ms | 50~80ms | 50~70ms | 视部署 | **30~40ms** | **10~20ms** |
| **规模上限** | ~10亿 | **万亿级** | ~10亿 | ~1亿 | ~10亿 | ~1000万 |
| **混合搜索** | ✅ | ✅ | ✅ | ✅（新增）| ✅ | ❌ |
| **元数据过滤** | 基础 | 高级 | 高级 | 基础 | **高级** | ❌ |
| **多模态** | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| **学习曲线** | ★ | ★★★ | ★★ | ★ | ★★ | ★★ |
| **10M向量/月成本** | $200~400 | $300~600 | $150~300 | $50~150 | $120~250 | $50~100 |

---

## 核心算法进展

### ANN 算法全景图

近似最近邻（ANN）搜索的核心目标：在召回率与速度之间取得最优平衡。主流算法分为四大类别：

```
ANN 算法
├── 图方法        → HNSW, DiskANN (Vamana)
├── 聚类方法      → IVF, IVF-PQ
├── 量化方法      → PQ, SQ, BQ
├── 哈希方法      → LSH
└── 混合方法      → IVF-HNSW, IVF-PQ
```

---

### 1. HNSW（Hierarchical Navigable Small World）

**原理**：构建多层跳跃式小世界图，搜索时从最上层粗略定位，逐步下沉到底层精细搜索。

**核心参数**：
- `M`：每个节点最大连接边数（默认 16，影响召回与内存）
- `efConstruction`：构建时候选列表大小（默认 200，影响索引质量）
- `efSearch`：搜索时候选列表大小（默认 100，影响延迟与召回）

**2024-2025 进展**：
- **参数自动调优工具**：Milvus、Qdrant 均推出自动参数推荐，减少人工调参成本
- **HNSW + Binary Quantization**：Weaviate v1.24 将 BQ 与 HNSW 结合，内存降低 32 倍，召回率损失 <2%
- **HNSW 内存优化**：Float16/bfloat16 存储支持，内存降低 50% 而精度几乎不变
- **增量索引改进**：支持更多实时插入场景，tombstone 清理策略优化

**优势**：查询速度极快（亚毫秒），召回率高，实现成熟，生态广泛。
**局限**：全量内存加载，内存成本高；索引构建慢；大规模下参数敏感。

---

### 2. DiskANN（Disk-Accelerated Nearest Neighbor）

**原理**：微软研究院提出的图算法（Vamana 图），核心创新是将压缩向量存于内存、全量向量存于 SSD，通过顺序读盘实现大规模向量检索。

**核心参数**：
- `R`：每个节点最大出边数
- `L`：剪枝阈值
- `Alpha`：图构建时的放大系数

**2024-2025 进展**：
- **FreshDiskANN**：支持实时插入/删除/更新，不再是静态索引（Milvus 原生集成）
- **Azure AI Search 集成**：微软将 DiskANN 集成到 Azure Database for PostgreSQL，降低企业使用门槛
- **批处理 I/O 优化**：现代 NVMe SSD 顺序读优化，延迟接近内存水平
- **与 HNSW 融合**：IVF-HNSW 混合索引结合聚类剪枝 + 图搜索，在大规模下性能更稳定

**优势**：内存成本大幅降低（SSD 比 DRAM 便宜 10 倍），天然支持超大规模（百亿~万亿级），过滤搜索原生友好。
**局限**：查询延迟高于纯内存 HNSW，索引构建慢，SSD 带宽成为新瓶颈。

---

### 3. IVF-PQ（Inverted File Index + Product Quantization）

**原理**：先通过 K-Means 聚类将向量分桶，搜索时只扫描最近的 N 个聚类中心及其桶内向量；PQ 将高维向量压缩为低维码书表示。

**核心参数**：
- `nlist`：聚类中心数量（影响召回与构建速度）
- `nprobe`：搜索时扫描的聚类数量（召回与速度权衡）
- `m`：PQ 子空间数（压缩率控制）

**2024-2025 进展**：
- **IVF-PQ + HNSW 混合**：先用 IVF 聚类缩小范围，再用 HNSW 精细搜索，兼顾召回与速度（Milvus 官方推荐）
- **Additive Quantization (AQ)**：相比 PQ 更新的量化方法，召回率更高但实现更复杂
- **OPQ 优化**：Optimized Product Quantization 通过旋转对齐向量主成分方向，进一步降低量化误差
- **混合精度量化**：不同维度使用不同精度（int8/float16），精度损失更小

**优势**：量化压缩率高（4x~32x），内存效率极高，适合超大规模数据。
**局限**：召回率天然低于 HNSW，精度受量化误差影响，参数调优复杂。

---

### 三大 ANN 算法横向对比

| 维度 | HNSW | DiskANN | IVF-PQ |
|------|------|---------|--------|
| **存储位置** | 全内存 | SSD + 压缩内存 | 可量化至内存 |
| **查询延迟** | **亚毫秒级** | 1~10ms（SSD I/O） | 1~5ms |
| **召回率** | 最高（95~99%） | 高（90~97%） | 中高（85~95%）|
| **内存效率** | 低（全量内存）| **高**（SSD 扩展）| 很高（量化压缩）|
| **索引构建** | 快（增量）| 慢（全量构建）| 中等 |
| **规模适用性** | <10亿向量 | **百亿~万亿** | <10亿 |
| **实时更新** | 支持（tombstone）| FreshDiskANN 支持 | 需重建 |
| **过滤支持** | 需后置过滤 | 原生分区友好 | 需限制扫描范围 |

**2024-2025 趋势**：不再是单一算法打天下，而是**混合架构**——IVF-PQ 建图 + HNSW 搜索 + DiskANN 大规模存储，三者融合是学术界和工业界的主要方向。

---

## 向量数据库的标量化与元数据过滤

### 核心挑战

向量检索（Top-K 最近邻）与标量过滤（WHERE 条件）天然存在张力：
- **前置过滤（Pre-filtering）**：先过滤再向量搜索，可能因过滤后候选集太小而召回率暴跌
- **后置过滤（Post-filtering）**：先向量搜索再过滤，可能返回不足 K 个结果
- **混合过滤（Hybrid filtering）**：向量搜索与过滤交替进行，复杂度最高

### 各产品元数据过滤能力对比

| 产品 | 过滤机制 | 索引类型 | 过滤性能 |
|------|------|------|------|
| **Pinecone** | Metadata index | B-Tree 类索引 | 中等 |
| **Milvus** | Field index（标量索引）| BitSet Filter | 好（HNSW 分区）|
| **Weaviate** | bm25 + vector 混合 | 倒排索引 | **优秀** |
| **Chroma** | Where 子句 | 简单过滤 | 基础 |
| **Qdrant** | Payload index | 分区 + 条件推送 | **优秀** |

### 2024-2025 技术进展

**1. 分区索引（Partitioned Indexing）**
将向量按过滤条件分区（Partition），搜索时只扫描相关分区，从根本上减少候选集。DiskANN 的设计原生支持此模式。

**2. BitSet Filter（Milvus 方案）**
在向量搜索层叠加 BitSet，用位运算高效标记符合过滤条件的向量集合，再与 ANN 搜索结果取交集，延迟增加控制在 5~15% 以内。

**3. 条件推送（Condition Push-down，Qdrant 方案）**
在图遍历或聚类扫描时，将过滤条件提前推送到数据节点，避免扫描无效向量，显著降低大规模过滤场景的延迟。

**4. 混合搜索（Hybrid Search）**
最流行的方案：稀疏向量（BM25/TF-IDF）提供精确关键词匹配，稠密向量（Embedding）提供语义理解，RRF（Reciprocal Rank Fusion）融合两者评分。

```
混合搜索评分 = RRF(sparse_score) + α × RRF(dense_score)
```

- **Pinecone**：内置 sparse-dense hybrid，无需外部 BM25 服务
- **Weaviate**：BM25 + 语义向量深度融合，体验最成熟
- **Qdrant**：Sparse vector + dense vector + payload filter 统一查询
- **Milvus**：通过混合搜索 API 支持，需自行集成 BM25

---

## 多模态向量检索

### 什么是多模态向量检索？

指在**同一向量空间**中跨模态检索：给定图像查询，返回相关文本；给定文本，返回相关图像；甚至支持音频、视频等混合模态。

核心技术路径：
1. **统一嵌入模型**：使用 CLIP、ColBERT、UniViLM 等多模态模型生成跨模态统一向量
2. **多向量检索**：每个文档/图像生成多个向量（如 ColBERT 的 token-level 向量），查询时逐token匹配再重排
3. **多索引融合**：各模态独立建索引，查询时融合结果

### 2024-2025 关键进展

**1. MUVERA（NeurIPS 2024）**
Google 提出的多向量检索新框架，将多向量相似度搜索问题降维为单向量问题，通过固定维度编码（FDE）大幅降低计算成本，同时保持高召回率。

**2. MegaPairs（ACL 2025）**
利用视觉-语言模型（VLM）和开放域图像合成异构 KNN 三元组，支持通用多模态检索训练，性能超越专用监督方法。

**3. ColBERTv2 / PLAID**
多向量检索的工业成熟方案：
- **ColBERT**：每个 token 生成一个向量，查询时快速最大相似度匹配（MaxSIM）
- **ColBERTv2**：压缩向量表示，内存降低 6 倍
- **PLAID**：分区索引加速，支持多阶段管道

**4. OneSparse（ACM SIGIR）**
统一多索引向量搜索系统，通过索引间交叉推送优化和多索引查询格式，高效支持稀疏+稠密混合的多模态场景。

**5. 多模态 RAG 落地**
- **Milvus + ColQwen2**：PDF 页面图像直接作为检索单元，结合文本+图像双模态生成答案
- **Chroma 多模态**：支持图像 Embedding + 文本混合存储
- **Qdrant Sparse Vector**：原生支持 ColBERT 风格的多向量检索

### 多模态向量数据库能力对比

| 产品 | 多模态检索 | Embedding 集成 | 跨模态搜索 | 多向量支持 |
|------|:------:|------|:------:|:------:|
| **Pinecone** | ✅ | ✅（内置）| ✅ | 基础 |
| **Milvus** | ✅ | 需集成 | ✅ | ✅（ColBERT）|
| **Weaviate** | ✅ | ✅（内置）| ✅ | 基础 |
| **Chroma** | ✅ | ✅ | ✅ | 基础 |
| **Qdrant** | ✅ | ✅（FastEmbed）| ✅ | ✅（Sparse）|

---

## 选型指南

### 按场景选型决策树

```
数据规模 < 1000万向量？
│
├── 是 → 团队无运维能力？
│         ├── 是 → 【Pinecone】5行代码上线，Serverless 按量计费
│         └── 否 → 需要极简 API？
│                   ├── 是 → 【Chroma】4个函数，Python/JS生态完整
│                   └── 否 → 需要最高性能？
│                             ├── 是 → 【FAISS】+ 自行封装（内存模式10~20ms）
│                             └── 否 → 【Qdrant】30~40ms，性价比最高
│
└── 否（> 1000万向量）→ 需要自托管？
                        ├── 否 → 【Pinecone】Serverless，弹性扩展
                        └── 是 → 复杂过滤 + GraphQL？
                                  ├── 是 → 【Weaviate】混合搜索 + GraphQL 最成熟
                                  └── 否 → 超大规模（> 1亿）？
                                            ├── 是 → 【Milvus】水平扩展，支持 DiskANN/HNSW/IVF-PQ
                                            └── 否 → 【Qdrant】高性能 + 成本控制
```

### 企业级选型关键评估维度

**1. 数据规模与增长预期**
- <1000 万向量：Chroma / Qdrant / Pinecone 均能满足
- 1000 万~10 亿：Qdrant / Weaviate / Milvus
- >10 亿：Milvus + Zilliz Cloud（唯一经过大规模验证的开源方案）

**2. 运维能力**
- **零运维**：Pinecone（Serverless）> Chroma Cloud > Zilliz Cloud
- **轻量运维**：Chroma（单进程）/ Qdrant（Docker 单节点）
- **全量运维**：Milvus（Kubernetes）/ Weaviate（Docker Compose 或 K8s）

**3. 查询性能要求**
- 亚毫秒极致延迟：HNSW 全内存方案（FAISS > Qdrant > Pinecone）
- 毫秒级均衡：Qdrant / Weaviate / Milvus
- SSD 成本优先：DiskANN 方案（Milvus / Azure AI Search）

**4. 过滤复杂度**
- 简单过滤（< 3 个条件）：Pinecone / Chroma
- 复杂过滤（多字段、多条件）：Qdrant Payload Index / Weaviate / Milvus

**5. 预算约束（10M 向量 × 768 维，月成本）**
| 预算 | 推荐方案 |
|------|------|
| < $100/月 | 自托管 Qdrant / Weaviate / Chroma |
| $100~300/月 | Qdrant Cloud / Weaviate Cloud |
| $300~500/月 | Zilliz Cloud / 2-3 节点 Pinecone |
| > $500/月 | 考虑自托管 Milvus 集群，长期成本更低 |

**6. 生态与集成**
- **LangChain / LlamaIndex**：Chroma 官方深度集成，Pinecone/Weaviate/Qdrant/Milvus 均支持
- **大模型厂商集成**：OpenAI、Cohere、Mistral 均推荐 Pinecone/Qdrant
- **云平台偏好**：AWS 用户推荐 Pinecone/Aurora + pgvector；Azure 用户推荐 Azure AI Search（DiskANN）；GCP 用户推荐 Vertex AI Vector Search

### 各场景最优推荐总结

| 场景 | 首选 | 备选 |
|------|------|------|
| **快速原型/POC** | Pinecone | Chroma |
| **RAG 生产应用（中小规模）** | Qdrant | Weaviate |
| **企业 RAG（大规模）** | Milvus | Weaviate |
| **多租户 SaaS 应用** | Weaviate | Qdrant |
| **推荐系统/实时搜索** | Qdrant | Pinecone |
| **图像/视频检索** | Milvus | Pinecone |
| **嵌入式/边缘设备** | FAISS | Chroma（Lite）|
| **成本敏感（自托管）** | Qdrant | Weaviate |
| **完全不想运维** | **Pinecone** | Zilliz Cloud |

---

## 推荐资源

### 官方文档

| 资源 | 链接 |
|------|------|
| Milvus 官方文档 | https://milvus.io/docs |
| Pinecone 官方文档 | https://docs.pinecone.io |
| Weaviate 文档 | https://docs.weaviate.io |
| Qdrant 文档 | https://qdrant.tech/documentation/ |
| Chroma 文档 | https://docs.trychroma.com/ |
| FAISS GitHub | https://github.com/facebookresearch/faiss |

### 学术论文

| 论文 | 会议/年份 | 关键贡献 |
|------|---------|--------|
| MUVERA: Multi-Vector Retrieval via Fixed Dimensional Encoding | NeurIPS 2024 | 多向量检索降维方法 |
| MegaPairs: Massive Data Synthesis for Universal Multimodal Retrieval | ACL 2025 | 多模态检索数据合成 |
| OneSparse: Unified Multi-index Vector Search | ACM SIGIR | 多索引融合引擎 |
| DiskANN paper (Vamana Graph) | Microsoft Research | SSD 大规模向量搜索基础 |

### 技术文章与报告

- **TensorBlue: Vector Database Comparison 2025** — 综合横向评测，涵盖五大主流产品
- **Vectroid: HNSW vs DiskANN Comparison** — ANN 算法深度对比
- **PingCAP: ANN Search Explained (IVF vs HNSW vs PQ)** — ANN 算法入门必读
- **Milvus Blog: DiskANN Explained** — DiskANN 原理解析

### 行业数据

- 向量数据库市场规模预计 2028 年超过 **43 亿美元**（CAGR ~35%）
- LF AI Foundation：Milvus 是当前最活跃的开源向量数据库项目之一
- GitHub 星标趋势（2024-2025）：Qdrant 增速最快，Milvus 稳居第一

### 开源项目与工具

| 工具 | 用途 |
|------|------|
| **FastEmbed**（Qdrant 出品）| 轻量级 Embedding 库，支持 100+ 模型 |
| **LlamaIndex / LangChain** | 向量数据库集成层，几乎支持所有主流产品 |
| **DB-GPT** | 国产 RAG 框架，内置向量数据库集成 |
| **Milvus Insight** | Milvus 可视化管理和监控工具 |

---

*本报告基于 2024-2025 年公开信息整理，数据和功能动态更新，建议在选型前查阅各产品最新官方文档。*
