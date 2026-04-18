# Harness Engineering：给 AI Agent 装上"护栏"的工程学

## 【开篇场景】

2025 年 11 月，Anthropic 的一篇工程博客描述了一个真实痛点：Claude Code 在处理一个耗时 8 小时的代码库重构任务时，前 4 小时表现完美，后 4 小时开始"跑偏"——AI 开始修改从未要求改动的文件，最终把一个正常运行的系统改出了 47 个 bug。这不是模型能力问题，而是**工程架构**问题。Anthropic 把这个领域命名为 **Harness Engineering**。

## 【核心是什么】

Harness（护栏/线束）是 Agent 与真实世界之间的中间层：它定义 Agent 能做什么、看到什么、在什么条件下停止。一个好的 Harness 能让普通模型做出超预期表现，一个差的 Harness 能让最强模型彻底失控。

Harness 不是单一工具，而是一套工程体系，包括沙箱隔离、状态追踪、错误恢复、权限边界和任务分解机制。Claude Code 的核心架构本质上就是一个精心设计的 Harness。

## 【关键原理】

把 AI Agent 想象成一辆没有刹车的自动驾驶汽车。Harness 就是给它装上刹车系统、车道偏离警告、和紧急停车按钮。

Anthropic 在 2025 年 11 月发表的《Effective harnesses for long-running agents》中揭示了几个核心原则：

**1. 脑手分离（Brain-Hand Decoupling）**：2026 年 2 月 Anthropic 发布的《Scaling Managed Agents》明确提出这个架构——用专门的 Agent 负责"思考规划"（Brain），用另一个 Agent 负责"执行操作"（Hand）。Brain Agent 拥有完整的任务上下文，可以做全局规划；Hand Agent 只负责在受限环境中执行具体指令。这样即使 Hand Agent 跑偏，Brain Agent 可以及时纠正。

**2. 状态护栏（State Rails）**：Harness 必须追踪 Agent 运行过程中的关键状态节点，并在状态异常时强制触发检查点（checkpoint）。就像火车必须沿着轨道行驶，Agent 也必须有状态边界。

**3. 权限最小化原则**：Claude Code 在 2025 年 10 月的更新中引入了沙箱权限分级——默认情况下，Agent 只能操作当前工作目录，无法访问 SSH keys、数据库密码等高敏感资源。只有在用户明确批准后才临时扩展权限。

## 【真实案例】

**案例 1：Claude Code 的"Auto Mode"**
2026 年 3 月 25 日，Anthropic 发布了 Claude Code auto mode。这是 Harness Engineering 的典型产物：AI 可以自主执行多步骤任务，但每完成一个阶段会自动暂停，提交 diff 并等待人工确认。这不是限制 AI 能力，而是把人工审批点嵌入到 Agent 工作流中，将"失控风险"压在可接受范围内。

**案例 2：Anthropic 并行编译 C 编译器的多 Agent 实验**
2026 年 2 月，Anthropic 用多个 Claude 实例并行开发一个 C 编译器，每个 Agent 负责不同模块（词法分析、语法树、代码生成）。Harness 在这里的作用是协调模块间的接口契约——每个 Agent 的输出必须严格符合预定接口规范，否则主协调 Agent 会拒绝合并。实验结果显示，团队协作开发速度比单体 Agent 提升了约 4 倍，而 bug 率降低了 60%。

**案例 3：SWE-bench 的"噪声"问题**
Anthropic 在 2025 年底的基准测试研究中发现，基础设施配置差异会导致同一个 Agent 在不同测试环境下产生截然不同的结果。这种"评测噪声"本质上是 Harness 设计不当导致的。解决方法是构建标准化、可复现的 Harness 环境，让 Agent 行为真正可测量。

## 【趋势与机会】

**1–3 年判断：值得深度关注。**

Harness Engineering 是 AI Agent 从"玩具"走向"工业级"产品的必经之路。目前这个领域几乎是空白——行业内有大量关于"Prompt Engineering"的讨论，却很少有人讨论"Prompt 的运行环境如何设计"。

**机会点：**
- **Harness-as-a-Service**：为企业提供标准化 Agent 运行框架，类似 CI/CD 工具链的定位
- **安全沙箱工具**：专用于限制 Agent 操作边界的沙箱产品（已有 Docker/容器方案，但专用产品稀缺）
- **合规审计工具**：当 Agent 替代人执行操作时，需要记录"Agent 做了什么、为什么做"——这是金融、医疗行业的合规刚需

**风险提示：** Harness 设计过度会严重削弱 Agent 能力，形成"又贵又慢又没用"的局面。好的 Harness 需要在"安全约束"和"自主空间"之间找到精确的平衡点，这是工程能力的真正考验。

一句话总结：模型能力决定了 Agent 的上限，Harness 设计决定了 Agent 的下限。
