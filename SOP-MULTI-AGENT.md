# 主-子 Agent 任务调度 SOP

> 版本：v1.0
> 创建：2026-04-08
> 适用场景：多任务并行研究/内容生产流水线

---

## 一、核心设计思想

**目标：** 最大化并发吞吐量 + 零空转等待 + 结果可追溯

**三条原则：**
1. **流水线不间断** — 5 个 slot 永远全满，1 个跑完立即补 1 个
2. **无批次概念** — 各任务独立，不等"整批完成"才推进
3. **交接有据** — 每个 agent 报告"完成了什么 / 准备做什么"，主 agent 依此决策

---

## 二、并发模型

```
┌─────────────────────────────────────────────────────┐
│                    主 Agent (Kiko)                   │
│  - 负责任务分配、监控、GitHub推送、飞书文档创建       │
│  - 不做具体研究，只调度                              │
└──────────────────┬──────────────────────────────────┘
                   │  最多 5 个并行
     ┌─────────────┼─────────────┬─────────────┬─────────────┐
     ▼             ▼             ▼             ▼             ▼
 ┌──────┐    ┌──────┐    ┌──────┐    ┌──────┐    ┌──────┐
 │Sub-1 │    │Sub-2 │    │Sub-3 │    │Sub-4 │    │Sub-5 │
 │Topic │    │Topic │    │Topic │    │Topic │    │Topic │
 │ 39   │    │ 40   │    │ 41   │    │ 42   │    │ 43   │
 └──┬───┘    └──┬───┘    └──┬───┘    └──┬───┘    └──┬───┘
    │完成补44    │完成补45    │完成补46    │完成补47    │完成补48
    └────────────┴────────────┴────────────┴────────────┘
         ←──────────── 滚动补位，永保 5 并行 ──────────────→
```

**并发数选择依据：**
- 5 是稳定平衡点（token 消耗 / 并发效率 / 系统稳定性）
- 遇到 loop 检测（excessive calls）立即停，等 5-8 秒再补

---

## 三、任务分发标准流程

### Step 1：初始化任务池

```
输入：N 个待执行任务（如 100 个 Topic）
输出：任务池队列（先进先出）
```

### Step 2：满载启动（第一波）

```
并行启动 min(5, 任务总数) 个 subagent
每个 agent 收到：任务编号 + 任务内容 + 完成后报告模板
```

### Step 3：监控与补位（滚动循环）

```
主 agent 监听 subagent 完成事件
    ↓
收到完成通知
    ↓
确认任务编号（防重复）
    ↓
从任务池取下一个任务
    ↓
立即 spawn 新 agent（补位）
    ↓
等待下一个完成通知
```

### Step 4：GitHub 批量推送

```
每完成 5 个任务（或每 20-30 分钟）
  → git add + commit + push
  → 主分支保持最新
```

### Step 5：飞书文档创建（按需）

```
Batch 01: 01-10 完成 → 创建飞书文档
Batch 02: 11-20 完成 → 创建飞书文档
...
每次创建后立即授权用户 full_access
```

---

## 四、Subagent 任务模板

每个 subagent 收到的任务 prompt 必须包含：

```
## 任务交接说明
上一个完成的任务：Topic XX [名称]
当前执行任务：Topic YY [名称]

---

[具体任务内容...]

## 完成后必须报告
```
[AGENT REPORT]
已完成：Topic YY [名称]
准备执行：Topic ZZ [名称]（等待主agent分配）
文件路径：/workspace/ai-reports/topics/YY-xxx.md
[/AGENT REPORT]
```
```

---

## 五、防重复机制（关键）

**问题：** spawn 时序差导致同一任务被两个 agent 重复执行（浪费资源）

**解法：主 agent 维护"正在执行"集合**

```
InProgress = {39, 40, 41, 42, 43}  ← 动态更新

当 agent N 完成 Topic X：
  1. 从 InProgress 删除 X
  2. 从任务池取 Y（Y ∉ InProgress）
  3. InProgress.add(Y)
  4. spawn(Y)
```

**重复检测逻辑：**
- 同一 Topic 只允许出现在 1 个 InProgress 集合中
- spawn 前再次检查任务池剩余量

---

## 六、Loop 检测应对

**触发条件：** exec / sessions_spawn 短时间内（~30s）重复调用失败

**症状：** 返回 `error: [Loop detected: excessive-calls]`

**应对流程：**
```
1. 停止继续 spawn
2. 等待 5-8 秒（exec: sleep）
3. 重试 spawn
4. 若仍失败，等待更长时间（15-20 秒）
```

**预防：** 两次 spawn 之间保持合理间隔（≥3s），不密集连续调用

---

## 七、GitHub 推送时机

| 触发条件 | 操作 |
|---|---|
| 每完成 5 个任务 | git add + commit + push |
| 长时间（>30min）无完成 | 主动检查文件状态，必要时推送 |
| 整批（Batch）完成 | 飞书文档 + GitHub 双推送 |

**推送命令：**
```bash
cd /workspace/ai-reports
git add topics/
git commit -m "feat: add topics XX-YY"
git push origin master
```

---

## 八、飞书文档创建时机

```
每个 Batch（10 个 Topic）完成后：
1. 汇总 10 个报告的核心摘要
2. 创建飞书文档（feishu_doc create）
3. 写入目录 + 摘要 + 各报告链接
4. 立即授权 full_access（feishu_perm add）
   - member_type: openid
   - member_id: ou_36f398d4a7e8bc323453cda6d2c90a65
5. 推送飞书通知用户
```

---

## 九、任务池管理

### 剩余任务队列（截至 2026-04-08）

```
49 AI-Life
50 AI-Investment
51 AI-Philosophy
52 AI-Future
53 AI-Chips
54 AI-Research-Method
55 AI-Gaming
56 AI-Agriculture
57 AI-Manufacturing
58 AI-Legal
59 AI-Psychology
60 AI-Creative-Writing
61 AI-Design
62 AI-Architecture
63 AI-Math
64 AI-Biology
65 AI-Physics
66 AI-Chemistry
67 AI-Astronomy
68 AI-Energy
69 AI-Materials
70 AI-Climate
71 AI-Social-Media
72 AI-Public-Services
73 AI-Transportation
74 AI-Language
75 AI-Cognitive-Science
76 AI-Neuroscience
77 AI-Philosopher
78 AI-Human-AI-Collaboration
79 AI-Freedom
80 AI-Education-Future
81 AI-Open-Science
82 AI-Interdisciplinary
83 AI-Healthcare-Future
84 AI-Genomics
85 AI-Nanotechnology
86 AI-Quantum-AI
87 AI-Superintelligence
88 AI-Existential-Risk
89 AI-History
90 AI-Philosopher-2
91 AI-Economics
92 AI-Sociology
93 AI-Anthropology
94 AI-Linguistics
95 AI-Political-Science
96 AI-International-Relations
97 AI-Futures-Studies
98 AI-Ethics-Applied
99 AI-Wisdom
100 AI-Summary-Outlook
```

---

## 十、关键配置参数

| 参数 | 值 | 说明 |
|---|---|---|
| max_concurrency | 5 | 最大并发 subagent 数 |
| spawn_timeout | 600s | 单个任务超时时间 |
| git_push_interval | 5 topics | 每多少个任务推一次 GitHub |
| feishu_batch_size | 10 topics | 每多少个任务创建一篇飞书文档 |
| loop_retry_delay | 5-8s | 遇到 loop 检测后等待时间 |
| report_template | [AGENT REPORT] 格式 | subagent 完成汇报模板 |

---

## 十一、常见问题处理

| 情况 | 处理方式 |
|---|---|
| 同一 Topic 被两个 agent 领取 | 后到的 agent 立即终止，主 agent 重新分配 |
| subagent 长时间（>10min）无响应 | 杀掉重启，重新分配同一任务 |
| GitHub push 失败 | 保留本地，等下一个推送周期重试 |
| 飞书文档创建失败 | 记录状态，后续手动补救或重试 |
| 所有任务跑完 | 创建汇总索引文档，推送最终 GitHub，通知用户 |

---

## 十二、效果评估

```
目标：100 个 Topic
模型：5 并行滚动
预计总时间：100/5 × 平均单任务时间(4min) ≈ 80 分钟

vs 串行：100 × 4min = 400 分钟（6.7小时）
vs 5并行整批：ceil(100/5) × 4min = 80 分钟（相同，但零空转滚动更快）
```
