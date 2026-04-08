# Multi-Agent 任务调度 SOP（架构版）

> 版本：v2.0
> 更新：2026-04-08
> 核心变化：主 Agent 轻量化 + 子 Agent 全自主

---

## 一、设计思想

**旧问题：**
- 主 Agent 既要分配任务、又要监听完成、还要判断推送时机
- 主 Agent 超时 / session 断开 → 整个调度链断裂
- 子 Agent 写完文件后依赖主 Agent 补位 → 单点依赖

**新架构原则：**
> 主 Agent 只做一件事：**把任务交给子 Agent**
> 子 Agent 自主完成：**执行 → 推送 → 消失**
> 两者互不依赖，各司其职

---

## 二、架构模型

```
┌─────────────────────────────────────────────────────┐
│                  主 Agent（极简）                     │
│  职责：初始化任务池 → 分配第一批任务 → 监听补位        │
│  超时 / 断开不影响子Agent运行                         │
└────────────────────────┬────────────────────────────┘
                         │ 分配任务（一次性，不参与执行）
     ┌──────────────────┼──────────────────┐
     ▼                  ▼                  ▼
 ┌────────┐        ┌────────┐        ┌────────┐
 │子Agent-1│        │子Agent-2│        │子Agent-3│
 │Task#01 │        │Task#02 │        │Task#03 │
 │写+推送  │        │写+推送  │        │写+推送  │
 │完成后   │        │完成后   │        │完成后   │
 │消失     │        │消失     │        │消失     │
 └───┬────┘        └───┬────┘        └───┬────┘
     │告知主Agent       │告知主Agent       │告知主Agent
     │"我完成了"        │"我完成了"        │"我完成了"
     ▼                  ▼                  ▼
  主Agent分配下一个  主Agent分配下一个  主Agent分配下一个
  Task#04            Task#05            Task#06
```

**关键变化：子 Agent 推送不再等主 Agent 判断数量**

---

## 三、主 Agent 工作流（v2）

### Step 1：初始化

```python
# 输入：N 个待执行任务
# 输出：任务池文件（task_pool.json）
# 每次分配记录已分配 topic_id

{
  "total": 100,
  "assigned": [1, 2, 3, 4, 5],
  "completed": [],
  "pending": [6, 7, 8, ... 100]
}
```

### Step 2：启动第一批

```
并行启动 5 个子 Agent，分别携带：
  子Agent-1：Task #01
  子Agent-2：Task #02
  子Agent-3：Task #03
  子Agent-4：Task #04
  子Agent-5：Task #05
```

### Step 3：监听补位（唯一职责）

```
主 Agent 持续监听 subagent 完成事件
  ↓
收到 Topic X 完成通知
  ↓
从 task_pool.json 取 pending 第一个：Task #Y
  ↓
spawn 子Agent-Y（携带 Task #Y）
  ↓
更新 task_pool.json（assigned.add(Y)）
  ↓
继续监听
```

**主 Agent 超时退出后：**
- 已在运行的子 Agent 不受影响，继续执行+推送
- 下次主 Agent 重新上线时，读取 task_pool.json 对账
- 发现缺失任务 → 继续补位

---

## 四、子 Agent 工作流（v2 全自主）

### 任务 prompt 结构

```
## 任务
Topic：## [具体任务内容]

## 执行要求
1. 撰写完整报告（≥4000字）
2. 写入文件：/workspace/ai-reports/topics/XX-name.md
3. 推送 GitHub：
   cd /workspace/ai-reports
   git add topics/XX-name.md
   git commit -m "feat: add topic XX-name"
   git push origin master
4. 完成报告后，按以下格式回复：
[AGENT DONE]
topic: XX
next_topic: <读取下方任务池，取pending第一个>
file: topics/XX-name.md
pushed: true
[/AGENT DONE]
```

### 关键行为

| 行为 | 说明 |
|---|---|
| 执行 | 撰写完整报告，≥4000字 |
| 写入文件 | 直接覆盖写入 |
| 推送 | 每个任务完成后单独推送（不等数量门槛） |
| 回报 | `[AGENT DONE]` 格式，告知完成状态 |
| 消失 | 推送完成后自行终止，不等待任何确认 |

**每个子 Agent 完成后：独立 git push，不依赖主 Agent 判断**

---

## 五、GitHub 推送规则（v2）

**旧规则：** 每 5 个 topic 推送一次（主 Agent 判断）
**新规则：** 每个子 Agent 推送自己的 topic（自主触发）

```bash
# 子 Agent 每次完成后执行
TOPIC_FILE="topics/$(printf '%02d' $TOPIC_ID)-ai-*.md"
git add "$TOPIC_FILE"
git commit -m "feat: add topic $TOPIC_ID"
git push origin master
```

**飞书文档创建：** 仍由主 Agent 批量处理（每 10 个 topic 一篇）

---

## 六、断线恢复机制

```
主 Agent 重新上线
    ↓
读取 task_pool.json
    ↓
对比 GitHub 远程分支（git fetch + git log）
    ↓
识别缺失任务（assigned 但未 pushed）
    ↓
继续补位，或自己直接写缺失任务
```

**核心：task_pool.json 是恢复点，GitHub 是最终状态证明**

---

## 七、当前实现（OpenClaw 平台）

> OpenClaw 平台限制：subagent 只能通过 `sessions_spawn` 启动，无法预先注入任务池读取能力。
> 因此当前采用**主 Agent 托管分配 + 子 Agent 自主推送**的混合模式。

**已实现（v2 简化版）：**

- ✅ 主 Agent 分配任务（不参与执行）
- ✅ 子 Agent 自主 git push（不等数量门槛）
- ✅ 推送完成即消失，无单点依赖
- ✅ 主 Agent 超时不影响子 Agent 执行
- ✅ 飞书文档由主 Agent 批量处理
- ⚠️ 补位仍依赖主 Agent 监听（平台限制）

---

## 八、飞书文档创建规则

```
每完成 10 个 topic → 主 Agent 创建一篇飞书文档
    ↓
写入目录 + 各报告摘要 + GitHub 链接
    ↓
立即授权用户 full_access：
  - member_id: ou_36f398d4a7e8bc323453cda6d2c90a65
  - permission: full_access
  - type: docx / bitable
    ↓
推送飞书通知
```

---

## 九、任务池状态文件

路径：`/workspace/ai-reports/task_pool.json`

```json
{
  "created_at": "2026-04-08T00:00:00+08:00",
  "total": 100,
  "assigned": [39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 53, 54, 55, 57, 58, 59],
  "completed_pushed": [39, 40, 41, 42, 43, 44, 48, 49, 50, 53, 54, 55, 57, 58, 59],
  "pending": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 51, 52, 56, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 100],
  "completed_unpushed": [47],
  "note": "47 education 待推送（git push进行中）"
}
```

---

## 十、完整任务列表（Topic 01-100）

```
Batch A（01-10）：Prompt/RAG/Agent/Fine-tuning/RLHF/Eval/Safety/多模态/Embedding/长上下文
Batch B（11-20）：推理/压缩/推理优化/向量数据库/Agent记忆/合成数据/分布式训练/MoE/推理模型/Function Calling
Batch C（21-28）：AI Coding/搜索/视频/语音/医疗/金融/教育/科学
Batch D（29-38）：机器人/治理/架构/开源LLM/MLOps/Context工程/伦理/Benchmark/企业AI/创业
Batch E（39-48）：云计算/数据工程/语音AI/营销/广告/媒体/产品设计/安全/生活服务/政策法规
Batch F（49-56）：投资/生活方式/未来/芯片/哲学/游戏/法律/教育
Batch G（57-64）：科研/制造/开源/未来/生物学/物理学/化学/天文学
Batch H（65-72）：能源/材料/气候/社交媒体/公共服务/交通/语言/认知科学
Batch I（73-80）：神经科学/哲学/人机协作/自由意志/教育未来/开放科学/跨学科/医疗未来
Batch J（81-100）：基因组学/纳米技术/量子AI/超级智能/存在风险/历史/哲学经济学/社会学/人类学/语言学/政治学/国际关系/未来学/应用伦理/智慧/总结展望
```

---

## 十一、关键参数

| 参数 | 值 | 说明 |
|---|---|---|
| 并发数 | 5 | 子 Agent 最大并行 |
| 主 Agent 职责 | 分配 + 补位 | 不参与执行 |
| 子 Agent 行为 | 执行 → 推送 → 消失 | 完全自治 |
| GitHub 推送 | 每个任务完成后即推送 | 不等数量门槛 |
| 飞书文档节奏 | 每 10 个 topic | 主 Agent 批量处理 |
| 断线恢复 | task_pool.json + GitHub 对账 | 主 Agent 重新上线时执行 |

---

## 十二、v1 vs v2 对比

| 维度 | v1 | v2 |
|---|---|---|
| 主 Agent 职责 | 分配 + 监听 + 判断推送 + 创建飞书 | 仅分配 + 补位 |
| GitHub 推送 | 主 Agent 判断（每5个） | 子 Agent 自主推送 |
| 主 Agent 超时 | 调度链断裂 | 子 Agent 继续运行 |
| 推送失败处理 | 主 Agent 统一处理 | 子 Agent 各自重试 |
| 飞书文档 | 主 Agent 批量处理 | 主 Agent 批量处理 |
| 断线恢复 | 依赖主 Agent 重新上线 | task_pool.json + GitHub 对账 |
