# 02. RAG 系统（检索增强生成）

> 研究时间：2026-04-08 | 研究者：Kiko Sub-Agent #02 | 运行时长：2分34秒

---

## 技术演进（2024-2025）

### 基础 RAG 的局限 → 推动大量 RAG 变体

#### 1. Self-RAG（自我反思 RAG）
- **来源**：华盛顿大学 + 艾伦 AI 研究所 + IBM（2023 ICLR）
- **核心创新**：让 LLM 自己决定何时检索、检索什么、如何评价生成质量
- **机制**：引入特殊 token（Retrieve / Critique），模型输出时自行判断是否需要外部知识，并做自我评估
- **优势**：减少不必要的检索开销，提高答案可溯源性；幻觉率显著降低
- **升级方向**：与长上下文结合，做自适应路由（SELF-ROUTE）

#### 2. Corrective-RAG（自纠错 RAG）
- **核心思想**：检索结果质量不可信时，主动触发纠错循环
- **工作流**：检索 → 评估相关性 → 质量差则重检/扩展查询/降级使用模型知识
- **典型实现**：多轮 Query Rewriting + 置信度门控，低于阈值则触发二次检索

#### 3. Hybrid-RAG（混合检索 RAG）
- **双路检索**：BM25（词匹配）+ Dense Vector（语义相似度），Reciprocal Rank Fusion 融合
- **为何必要**：纯向量检索对精确 ID 查询、缩写词、专业术语召回差
- **实践结论**：BM25 不可省略，混合检索在所有 benchmark 上持续优于单路

#### 4. Agentic RAG（智能体 RAG）
- **2025 年主流方向**：将 RAG 管道嵌入多智能体协作框架
- **代表模式**：
  - Router Agent：理解用户意图，分发到不同检索管道
  - Multi-Document Agent：跨多个文档集合同时推理
  - Plan-and-Execute Agent：规划检索步骤，迭代执行
- **适用场景**：复杂问答、企业知识库、多数据源联合检索

#### 5. GraphRAG（知识图谱 RAG）
- **来源**：微软研究院 2024 年开源项目
- **核心思路**：从知识图谱中检索相关实体和关系，而非纯文本块
- **Pipeline**：文本 → 知识图谱抽取 → 社区层次构建 → 图 ML 输出 → LLM 生成
- **优势**：在多跳推理问题上远超朴素语义搜索
- **2025 演进**：GraphRAG 2.0 支持本地模型（Ollama），大幅降低部署门槛

#### 6. 其他前沿方向

| 技术 | 核心思想 |
|------|---------|
| Late Chunking | 先 embedding 全句再做滑动窗口切分，保留句子级语义 |
| Contextual Retrieval | chunk 前加文档级上下文描述，显著提升召回 |
| Agentic Chunking | LLM 驱动的语义感知切分，准确率提升 40% |
| Multi-modal RAG | 支持图像+视频+音频检索（2025 正在成熟） |
| TableRAG | 专注文本+表格混合检索 |
| LegoRAG | 模块化 RAG 管道，灵活组合 |

---

## 主流框架对比

| 维度 | **LlamaIndex** | **LangChain / LangGraph** | **Haystack** | **RAGFlow** |
|------|--------------|--------------------------|-------------|------------|
| **定位** | 专注检索优化 | 通用编排框架 | 企业级生产 | 无代码可视化 |
| **Token 开销** | ~1.6K | ~2.4K | 中等 | 较高 |
| **RAG 代码量** | 30-40% 更少 | 较多 | 中等 | 拖拽生成 |
| **集成数量** | 300+ | 500+ | 200+ | 主流 |
| **学习曲线** | 较平缓 | 较陡 | 中等 | 平缓 |
| **最佳场景** | 检索密集型 | 多步 Agent | 企业生产 | 非技术人员 |

**推荐生产模式**：LlamaIndex（检索层）+ LangGraph（编排层）— 业界最常见组合。

---

## 架构设计要点

### 生产级 RAG Pipeline
```
用户查询 → Query Rewrite → Hybrid Retrieval → Reranker → Context Assembler → LLM Generator → 响应+引用
```

### 核心要点
- **Chunking**：300-800 token，语义切分优于固定大小，表格提取为结构化 JSON，元数据必须
- **检索**：BM25 + vector 混合是标配，检索 50-200 条候选，rerank 精选 top 5-10
- **Reranker**：Cross-Encoder，`bge-reranker-large` 质量优先
- **安全护栏**：Prompt Injection 检测（`ignore previous instructions` 等模式）

---

## 与长上下文的结合趋势

### 核心结论
> **2025 年最优解几乎都不是纯 RAG 或纯长上下文，而是混合架构。**

### 定量对比

| 维度 | RAG | 长上下文 LLM |
|------|-----|------------|
| 延迟 | ~1s（固定） | 随上下文线性增长，200K 可达 12s+ |
| 成本 | 节省 10-50x | 每次付费全部上下文 |
| 事实准确性 | 94%（可溯源） | 89%（存在中间遗忘） |
| 幻觉率 | 低 | 高 |

### 何时选谁
- 知识库 > 1M tokens / 频繁更新 → **RAG**
- 法律合同全文关系分析 / 多会话客服 → **长上下文**
- **大多数企业场景** → **混合路由（SELF-ROUTE）**

---

## 实战建议

| 阶段 | 核心动作 |
|------|---------|
| 起步 | LlamaIndex + 语义切分 + 混合检索 |
| 进阶 | Query Rewrite + Reranker + 层级 Chunking |
| 生产 | LlamaIndex + LangGraph 组合 + 可观测性 + 安全护栏 + Redis 缓存 |

**常见踩坑**：幻觉率高（chunk丢上下文）→ 层级切分；检索不相关（向量模型不适配）→ 微调 embedding；延迟高 → k值调小 + Rerank 前置

---

## 推荐资源

### 必读
- [microsoft/graphrag](https://microsoft.github.io/graphrag/) — 微软 GraphRAG
- [run-llama/llama_index](https://github.com/run-llama/llama_index) — LlamaIndex
- [langchain-ai/langchain](https://github.com/langchain-ai/langchain) — LangChain + LangGraph
- Self-RAG（arXiv:2310.11511）
- RAG vs Long-Context LLMs（EMNLP 2024, arXiv:2407.16833）

### Benchmark
- MS MARCO（通用问答）、BEIR（17 个检索任务）、HotpotQA（多跳推理）
