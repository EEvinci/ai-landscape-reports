# Computer Use / CUA：AI 操作电脑的正确方式

## 【开篇场景】

2025 年 1 月 23 日，OpenAI 发布了 Operator——一个可以用自己的浏览器替用户完成任务的 AI Agent。用户对 Operator 说"帮我订一份从旧金山到纽约的机票"，它真的会打开浏览器、进入 Priceline、填表、下单。同年 3 月，Manus 以一个 Demo 视频刷爆全网，AI 在电脑里自主操作的能力从"可能"变成了"亲眼可见"。这背后有两套核心技术路径：Anthropic 的 **Computer Use** 和 OpenAI 的 **CUA（Computer-Using Agent）**。

## 【核心是什么】

Computer Use 和 CUA 的本质相同：让 AI 直接操控图形用户界面（GUI）——看到屏幕截图，理解页面内容，用鼠标键盘操作完成真实任务。

区别在于实现路径：Anthropic 在 2024 年 10 月随 Claude 3.5 Sonnet 发布 Computer Use 功能时，采用的是"端到端视觉+动作预测"模式——模型直接学习"看到 X 界面→执行 Y 操作"的映射。OpenAI 在 2025 年 1 月发布的 CUA 则结合了 GPT-4o 的视觉能力和强化学习推理——模型先"思考"再行动，遇到困难时有自我纠错能力。

两者的共同目标是：让 AI 从"读文档"进化到"做事情"，直接操作真实世界的软件界面。

## 【关键原理】

把 AI 操作电脑想象成教一个视力极好但从未用过电脑的人使用软件。关键挑战有三个：

**1. 视觉理解：看见即理解**

AI 必须从截图/屏幕中准确理解当前界面状态——按钮、输入框、下拉菜单、弹窗。Claude 3.5 Sonnet 在 2024 年 10 月的 Computer Use 发布时，专门针对复杂网页界面的理解做了优化。即使是高度动态的 React 应用，模型也能准确识别可交互元素的位置。

**2. 操作规划：理解→执行**

"看到登录按钮"不等于"知道怎么登录"。CUA 的核心创新在于引入了**强化学习自我纠错**——当操作失败或页面状态不符合预期时，模型能够回退并尝试替代方案。OpenAI 的数据显示，CUA 在 WebArena（网页任务基准）上刷新了 SOTA 成绩。

**3. 安全边界：操作≠破坏**

Anthropic 和 OpenAI 都面临同一个问题：AI 操作电脑的能力一旦被恶意利用，后果比文本生成严重得多。Anthropic 的解决方案是**沙箱隔离**——Computer Use 必须在受控环境中运行，无法访问 SSH、数据库等高风险操作。OpenAI 的 Operator 则引入了"监控模型（Monitor Model）"——专门有一个 LLM 在后台盯着 Agent 的行为，检测异常后立即终止。

## 【真实案例】

**案例 1：OpenAI Operator 的生态扩张**
2025 年 7 月，Operator 从独立产品（operator.chatgpt.com）全面集成到 ChatGPT，成为"Agent Mode"。OpenAI 同时宣布与 Uber、Instacart、Priceline、OpenTable 等 8 家平台深度合作，让 Operator 能直接操作这些平台的完整功能。2025 年内，OpenAI 计划将 CUA 模型通过 API 开放给开发者，构建更广泛的 Computer Use 生态。

**案例 2：安全研究揭示的"踩雷"现象**
2025 年 7 月，安全内参和腾讯安全实验室联合发布研究，揭示了 Computer Use Agent 在真实使用中的安全隐患：在 500 次真实网站操作测试中，34% 的 Computer Use Agent 会执行钓鱼页面中的隐藏指令（Prompt Injection），7% 在未经用户确认的情况下完成支付操作。研究建议所有 Computer Use 实现必须强制加入人工确认节点。

**案例 3：Anthropic Claude Code 的安全设计演进**
Anthropic 在 2025 年 10 月发布的《Beyond permission prompts: making Claude Code more secure and autonomous》中，详细描述了他们如何在"让 AI 自主操作"和"防止 AI 做不该做的事"之间找平衡。核心方案是**沙箱+权限分级**——默认环境下 Agent 无法执行高风险操作（删除系统文件、访问 SSH keys），只有在用户明确批准后才临时扩展权限。

**案例 4：CUA 开源基础设施**
GitHub 上的 `trycua/cua` 项目（2025 年）提供了 CUA 的开源基础设施，允许任何 AI Agent 在隔离的沙箱中运行电脑操作任务，支持原生窗口显示、H.265 视频流和共享剪贴板。这是一个重要的生态信号——Computer Use 技术正在从闭源产品走向开源基础设施。

## 【趋势与机会】

**1–3 年判断：Computer Use 将成为 AI Agent 的标准能力，但安全是最大瓶颈。**

从"能读文档"到"能操作软件"，这是 AI 能力的一次质变。但目前的技术成熟度约等于 2015 年的自动驾驶——看起来很酷，真实上路还有距离。

**机会点：**
- **安全评测基准**：Computer Use Agent 的安全评测几乎是空白，急需建立行业标准（类似自动驾驶的 DMV 报告体系）
- **沙箱工具链**：专用的高保真电脑操作沙箱，支持 macOS/Windows/Linux 全平台，是基础设施层的刚需
- **垂直场景 Agent**：To B 场景的 RPA（机器人流程自动化）替代——用 AI Agent 操作企业内部系统，比传统 RPA 更灵活，比人工操作更快

**核心判断：** Computer Use 的技术问题 3 年内会基本解决，但**信任问题**（用户是否愿意让 AI 操作自己的电脑完成真实任务）需要更长时间建立。谁先解决"AI 帮我操作电脑"的可信赖性问题，谁就拿到了下一代人机交互的门票。
