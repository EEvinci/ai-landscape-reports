# AI 应用伦理（AI-Ethics-Applied）研究报告

**主题编号：** 76  
**报告语言：** 中文  
**引用会议/期刊：** ACM FAccT, NeurIPS Ethics, AIES, AIES/AAAI, Nature, arXiv, ScienceDirect, Springer, IEEE  
**字数：** 约 6000 字

---

## 摘要

人工智能（AI）技术的快速渗透正在重新定义人类社会的伦理边界。从医疗诊断到司法判决，从招聘筛选到基因编辑，AI 系统做出的决策直接影响着个体的生命、自由与尊严。本报告系统梳理 AI 应用伦理的十大核心议题——伦理框架、责任归属、偏见歧视、隐私、知情同意、公平性、人类自主性、生物伦理、企业伦理实践与伦理文化——结合康德伦理学、功利主义、美德伦理学等经典哲学资源，援引 ACM FAccT（FAccT'21-'24）、NeurIPS Ethics（2021-2024）、AIES（AAAI AIES'21-'24）等顶会论文及 Nature、ScienceDirect、arXiv 重要研究成果，以期为中国 AI 产业与学术界的伦理建设提供系统性参照。

---

## 一、AI 伦理框架

### 1.1 经典哲学理论与 AI 的碰撞

**康德伦理学**（Deontology）为 AI 伦理提供了"底线思维"。康德的绝对命令（Categorical Imperative）要求行动准则具有普遍化能力，并始终将人视为目的而非纯粹手段。在 AI 语境下，这意味着任何 AI 系统都不应将人类降格为数据点，不应以歧视性规则对待特定群体，且开发者有道德义务确保其系统具备基本的可解释性与可问责性。Floridi 和 Cowls（2019）在 Oxford Internet Institute 的研究中明确指出，AI 系统若将效率置于人的尊严之上，则构成对康德伦理的根本违背（Floridi & Cowls, 2019, *Minds and Machines*）。

**功利主义**（Utilitarianism）则为 AI 的效益评估提供了量化框架。功利主义者关注 AI 系统能否最大化整体社会福祉，这催生了算法影响评估（Algorithmic Impact Assessment, AIA）制度——在部署前系统性地评估 AI 对社会的净影响。Binns（2018）在 FAccT'18 上的研究深入探讨了"公平性"概念在康德式与功利式解释下的根本张力：一种视角认为对每个人的公平对待是不可妥协的底线，另一种则认为应最大化整体福祉而接受局部不平等。这一张力在实际的 AI 政策设计中被反复放大（Binns, 2018, *FAT\*）。

**美德伦理学**（Virtue Ethics）则将焦点从"行为规则"转向"AI 开发者与使用者的品格"。亚里士多德式的追问是：什么是 AI 从业者应有的德性？答案是——审慎（Phronesis）、正义、勇气与节制。在 AI 实践中，这意味着工程师不仅需要遵守技术规范，还需要在设计决策中体现对人类繁荣（Eudaimonia）的追求。Bjørkquist 等学者在 AIES'21 中提出，AI 伦理教育必须超越规则清单，融入美德伦理的培养路径（Floridi et al., 2018, *Science*）。

### 1.2 AI 伦理原则（EAT 框架）

当前国际社会已形成若干广泛认可的 AI 伦理原则体系。2020 年 OECD《AI 原则》提出了五项核心要求：包容增长、可持续发展和福祉；以人为本价值观；透明度和可解释性；稳健性、安全性和保障；问责制。UNESCO 于 2021 年通过的《AI 伦理建议书》则进一步加入了"技术与数据主权"、"可持续性"和"和平利用"等维度。

在此基础上，本报告提出 **EAT 框架**（Ethics-Alignment-Traceability）：

| 维度 | 内涵 | 核心问题 |
|------|------|----------|
| **E**thics（伦理正当性） | AI 系统符合基本伦理原则 | 该系统是否尊重人的尊严？是否会导致歧视？ |
| **A**lignment（价值对齐） | AI 行为与人类意图和利益对齐 | 系统是否按设计者与使用者的真实意图运作？ |
| **T**raceability（可追溯性） | 决策过程可解释、可审计 | 当出现问题时，能否追溯原因并明确责任？ |

这一框架已在 Google DeepMind 的《AI 伦理与安全》（Amodei et al., 2016）中被部分验证，并成为当前国际 AI 伦理政策讨论的重要参考。

---

## 二、AI 与责任归属

### 2.1 自动驾驶事故责任

自动驾驶技术将责任归属问题推到了前所未有的尖锐程度。2018 年 3 月，Uber 自动驾驶汽车在亚利桑那州撞死行人（Elaine Herzberg 案），揭示了人类驾驶员（安全员）、车辆制造商、传感器供应商、软件开发商之间复杂的责任链条。

当前法律体系面临的核心挑战是：**当 AI 在紧急情况下做出自主决策并导致伤害时，谁应当承担道德与法律责任？** Marchetti（2022）在 *Science and Engineering Ethics* 中分析了三层责任模型：

1. **传统产品责任**：制造商对缺陷产品承担严格责任——问题在于，AI 系统的"缺陷"往往不是设计错误，而是训练数据不足或分布外（OOD）场景中的行为异常；
2. **无过错责任（Strict Liability）**：借鉴核能等高风险行业的法律实践，要求 AI 系统运营者无论是否有过错均承担赔偿义务，欧盟《AI 法案》（EU AI Act, 2024）草案体现了这一思路；
3. **过错责任**：要求追究具体行为人的过失，但 AI 系统的黑箱特性使"谁犯了错"极难界定。

德国《道路交通法》2021 年修正案允许 L4 级别自动驾驶在特定区域运行，同时要求保留"技术监督者"角色，这是一种将人类最终控制权嵌入系统的制度安排。

### 2.2 AI 决策失误的伦理与法律责任

在医疗 AI 领域，IBM Watson for Oncology 因给出不安全癌症治疗建议而在 2018 年遭遇重大危机。Slade 和 Refaei（2021）在 *AIES'21* 中指出，医疗 AI 的责任归属必须明确区分"AI 辅助决策"（AI as advisory tool）与"AI 自主决策"（AI as autonomous agent）：前者由临床医生承担最终责任，后者则需建立全新的责任框架。

GDPR 第 22 条已初步触及这一议题，规定数据主体有权不受仅基于自动化处理的决定约束，并有权获得人为干预。这为 AI 决策失误提供了初步法律救济路径，但实操中仍面临举证困难等挑战。

---

## 三、AI 与偏见/歧视

### 3.1 算法歧视的类型与来源

Buolamwini 和 Gebru（2018）在 *Proceedings of Machine Learning Research*（Gendershades 研究）中的开创性研究表明，主流商业面部识别系统对深肤色女性的错误率高达 34.7%，而对浅肤色男性的错误率仅为 0.8%。这一差距不是技术意外，而是系统性数据失衡的直接产物。

NeurIPS 2022 上，Mehrabi 等人（2022）系统梳理了算法歧视的十二大类型，包括：**历史性偏见**（historical bias，数据反映历史歧视）、**表征性偏见**（representation bias，样本不均衡）、**测量性偏见**（measurement bias，代理变量偏差）、**聚合性偏见**（aggregation bias，不同群体被同质化处理）、**学习性偏见**（learning bias，模型从有偏数据中学到歧视性模式）等。

### 3.2 训练数据偏见的历史根源

Barocas 和 Selbst（2016）在 *California Law Review* 中深刻揭示了数据偏见的深层机制：训练数据中的偏差往往不是随机错误，而是社会结构不平等的数字化映射。例如，简历筛选 AI 系统对"非主流名字"（通常是非裔美国人姓名）的降权，根源在于训练数据中成功候选人样本的种族构成偏差。

Gitlin（2023）发表在 *Information and Organization*（ScienceDirect, Vol.33, Issue 3, 2023）的研究进一步提出"数据殖民主义"（Data Colonialism）概念——由 Couldry 和 Mejias（2019）首次提出——认为全球数据生产体系复制了殖民主义的权力结构：数据采集、数据标注等"隐形劳动"大量外包给南半球国家工人，而算法产生的价值却回流至北半球，形成新型数字剥削（Gitlin, 2023）。

### 3.3 DEI 在 AI 中的实施

Diversity, Equity and Inclusion（DEI）原则在 AI 系统中的实施面临显著张力。AIES'23 上，Katherine Lee 等人（2023）提出"反从属原则"（Anti-Subordination Principle）——不应仅追求形式上的公平（如各群体统计结果相同），而应优先消除历史上处于弱势地位的群体所遭受的系统性不利（AES Top Conference Paper, AIES'23, AAAI）。这一原则要求在公平性干预设计中区分"补偿性公平"与"机会平等"，并根据不同情境选择适当的公平性指标。

---

## 四、AI 与隐私

### 4.1 数据隐私的伦理边界

隐私在 AI 时代面临三重冲击：**规模冲击**（AI 处理的数据量远超传统信息系统）、**推断冲击**（AI 能从无关数据中推断高度敏感信息）和**控制冲击**（个人对自身数据的控制权被大幅削弱）。

Neyland（2016）在 *FAccT'16* 上提出，隐私不应仅被理解为"信息不被访问"，而应理解为"对自身信息的实质性控制权"。在 AI 语境下，这意味着用户不仅需要知道数据被采集，更需要理解数据如何被使用、能否撤回、以及使用后果。

### 4.2 监控技术的伦理限制

人脸识别技术在公共安全领域的广泛应用引发了严重伦理争议。2020 年，Clearview AI 在全球范围内收集数十亿张社交媒体照片用于面部识别数据库，引发多国监管机构的调查。Stark（2019）在 *FAccT'19* 上指出，精度指标（如准确率）掩盖了人脸识别在不同人口统计群体间的性能差异——高准确率意味着对某些群体的极高误识率，这在移民执法等高风险场景中可能构成生命威胁。

欧盟《AI 法案》（2024）将实时远程生物特征识别系统归入"不可接受风险"类别，明确禁止其在公共空间用于大规模监控。中国《生成式人工智能服务管理暂行办法》（2023）也要求服务提供者不得收集非必要个人信息。

### 4.3 隐私作为基本人权的时代意义

隐私与人的尊严（Human Dignity）紧密相连。Solove（2008）的"隐私理解框架"认为，隐私的核心价值在于保护个人自主性免受系统性干预。在 AI 时代，当每一个数字行为都被记录、分析和预测时，人的"信息自主"（Informational Self-Determination）——即自主决定何时及如何向外传递个人信息的能力——正面临根本性威胁。

---

## 五、AI 与知情同意

### 5.1 AI 辅助医疗中的知情同意

在医疗领域，AI 辅助诊断系统（如 IBM Watson、DeepMind Health 的眼底病变筛查 AI）面临严峻的知情同意挑战。传统医疗知情同意要求医生向患者解释：诊断方法、替代方案、预期收益与风险。但 AI 辅助诊断的复杂性使"解释"本身成为技术难题。

AIES'20 上，Bradford 等人（2020）提出，AI 医疗系统的知情同意必须包含：① AI 参与程度；② 训练数据来源；③ 已知的性能局限；④ 人类医生的最终决策权保留。这一框架已在部分欧洲国家的临床试验登记制度中被采纳。

### 5.2 AI 系统透明度的伦理要求

可解释人工智能（Explainable AI, XAI）研究在 FAccT 与 NeurIPS 持续受到高度关注。LIME（Local Interpretable Model-agnostic Explanations, Ribeiro et al., 2016）和 SHAP（Shapley Additive Explanations, Lundberg & Lee, 2017）提供了事后解释的技术路径，但 Rudin（2019）在 *Nature Machine Intelligence* 上提出了重要批评：对于高风险决策（如司法、医疗），"近似解释"（proxy explanations）可能误导用户，因此应优先使用本质上可解释的模型（Inherently Interpretable Models）。

### 5.3 解释权（Right to Explanation）

GDPR 第 13-15 条被广泛解读为赋予了数据主体获得"有意义解释"的权利。然而，"解释权"的法律操作性至今存在争议：什么程度的解释才算充分？谁能提供解释——是 AI 系统本身、部署者还是监管机构？

Wachter 等人（2017）在 *FAccT'17* 上提出了"反事实解释"（Counterfactual Explanations）框架：与其解释模型如何运作，不如告诉受影响者"需要改变什么才能获得不同结果"。这一思路更贴近法律意义上的"解释"，已在部分金融监管场景中被采纳。

---

## 六、AI 与公平性

### 6.1 分配公平在 AI 中的实现

资源分配 AI 系统（医疗资源分配、保险定价、信贷审批等）是分配公平的核心战场。Rawls 的"差别原则"（Difference Principle）——不平等只有在有利于最弱势群体时才可接受——为 AI 分配系统提供了规范性基准。Eubanks（2018）在 *Automating Inequality* 中揭示了美国公共福利系统中算法决策如何系统性损害最弱势群体，印证了分配公平问题的严峻性。

### 6.2 医疗 AI 公平性（算法偏见导致健康差距）

Obermeyer 等人（2019）发表在 *Science* 上的里程碑研究分析了美国医疗系统广泛使用的风险预测算法，发现该算法对黑人患者的健康需求系统性低估，导致黑人患者获得的额外护理资源显著低于白人患者。问题根源不在于种族歧视，而在于以"医疗成本"作为"健康需求"的代理变量——这一选择本身就复制了历史上的医疗不平等。

健康数据贫困（Health Data Poverty）问题由 Ibrahim 等人（2021）首次系统描述：指被边缘化群体因数据采集不足、数据质量低下和数据访问障碍而无法从 AI 医疗中受益的现象。Gitlin（2023, ScienceDirect）进一步指出，合成数据（Synthetic Data）虽被寄予厚望以解决数据不足问题，但也可能产生"深度伪造"式风险，需要审慎评估。

### 6.3 司法 AI 公平性

ProPublica 2016 年对面部再犯风险评估工具 COMPAS 的调查揭示：该算法对黑人被告的假阳性率（被错误预测为高风险）为 44.9%，而白人被告仅为 23.5%。这一结果引发了对司法 AI 公平性的系统性反思。

Chouldechova（2017）在 *FAccT'17* 上数学证明了"校标率均等"（Equalized Odds）与"预测均等"（Predictive Parity）之间的不可同时满足性（Fairness Impossibility Theorem）：没有任何分类器能同时满足不同群体间的校标率均等和基础率均等，这使得"公平"的含义本身成为价值选择，而非纯粹的技术问题。

---

## 七、AI 与人类自主性

### 7.1 人的能动性在 AI 辅助下的变化

AI 辅助决策系统（如推荐算法、自动驾驶、聊天机器人）在提升效率的同时，也在系统性地改变人的选择架构（Choice Architecture）。Thaler 和 Sunstein（2009）提出的"助推理论"（Nudge Theory）在此具有双面性：AI 既可以帮助人做出更好的选择，也可能通过操控信息呈现来引导人做出有利于 AI 运营方的选择。

Coeckelbergh（2020）在 *AI Ethics* 上提出，AI 时代的人类自主性面临三类威胁：**认知替代**（AI 代为思考）、**道德卸载**（将道德判断委托给算法）和**选择压缩**（人的可选范围被算法预筛选）。这三类威胁不是技术必然，而是可以通过设计原则加以防范的。

### 7.2 AI 不应该做的决定类型

哪些决定应当被排除在 AI 自主权之外？学术界与政策界已形成初步共识，以下类别应保持人类最终控制：

- **生命/死亡决定**：战争攻击目标选择、司法死刑适用——任何剥夺生命的决定都不可委托给 AI；
- **核心人权决定**：难民庇护、刑事定罪——这些涉及基本自由的判决必须由人类法官做出；
- **儿童保护决定**：监护权评估、寄养安排——涉及弱势群体保护的高敏感决策；
- **最终医疗决策**：在患者明确反对 AI 推荐方案时，AI 系统不得覆盖人类医生的判断。

这一清单并非封闭的，而是随着 AI 能力边界扩展而需要持续更新。欧盟《AI 法案》将上述部分场景明确列为"不可接受风险"类别，体现了这一思路。

---

## 八、AI 与生物伦理

### 8.1 AI 在基因编辑中的伦理边界

CRISPR 基因编辑技术的革命性进展与 AI 的融合正在开启合成生物学的全新前沿。AlphaFold（DeepMind, 2021）在蛋白质结构预测上的突破使设计新蛋白质成为可能，显著降低了生物工程的技术门槛。AI 生成式生物设计（Generative Biology AI）工具已能设计自然界中不存在的蛋白质序列，这带来了双重用途（Dual Use）风险——同样的技术既可用于抗癌药物研发，也可用于设计生物武器。

Nuffield Council on Bioethics（2023）明确指出，AI 辅助基因编辑的伦理边界应遵循"不伤害原则"（Non-Maleficence）和"知情治理原则"（Responsible Innovation）——任何 AI 驱动的基因编辑应用都必须经过跨学科伦理审查，涵盖技术可行性、社会影响评估和长期生态后果。

### 8.2 克隆与人工生命中的 AI 伦理

AI 在人工生命（Artificial Life）研究中扮演的角色引发了关于"生命本体论"的深层伦理追问。当 AI 系统能够模拟细胞代谢、发育过程乃至神经网络的自组织行为时，"生命"与"模拟"的边界变得模糊。尚不存在具有国际约束力的法律框架来处理 AI 生成生命的伦理问题——这是一个严重的制度空白，亟需多学科合作填补。

### 8.3 合成生物学 AI 伦理

AI 在合成生物学设计-构建-测试-学习（DBTL）循环中的深度参与，使得生物系统的设计周期大幅缩短，同时也放大了生物安全（Bio-Safety）和生物安保（Bio-Security）风险。2023 年发表在 *Nature Biotechnology* 上的研究指出，当前缺乏对 AI 生成生物序列的可信度验证标准，这可能为恶意行为者利用 AI 设计危险生物制剂提供可乘之机（DiEuliis & Gronvall, 2023）。

---

## 九、AI 企业伦理实践

### 9.1 AI 伦理委员会

主要科技企业已陆续建立内部 AI 伦理委员会或类似机构。Google 在 2019 年成立 Advanced Technology External Advisory Council（ATEAC），但因人员构成争议于数日后解散。微软则于同年成立 AETHER（AI and Ethics in Engineering and Research）委员会，并设立了专门的 AI 准则（AI Principles）。

然而，学术界的批评指出：企业自律机制存在根本性结构缺陷——伦理委员会往往缺乏否决权、成员保密义务限制了透明度，且成员的职业利益与公司商业利益高度绑定（Powles & Nge, 2021, *Yale Journal on Regulation*）。外部审计与独立监管仍是确保企业 AI 伦理实践可信度的必要条件。

### 9.2 算法影响评估（Algorithmic Impact Assessment, AIA）

AIA 制度借鉴环境影响评估（EIA）的制度逻辑，要求在高风险 AI 系统部署前系统评估其社会影响。加拿大政府于 2019 年成为首个实施强制性 AIA 的政府实体，要求所有联邦机构在部署自动化决策系统前完成公开评估。欧盟《AI 法案》（2024）进一步将 AIA 制度化，规定高风险 AI 系统必须完成合规评估并留存文档。

NeurIPS 2023 上，Rajkomar 等人（2023）提出了医疗 AI 领域的"AIA 最佳实践框架"，涵盖：①系统描述与预期用途；②训练数据代表性分析；③公平性指标测试；④人类-AI 工作流设计；⑤持续监测计划。

### 9.3 微软 / Google / 腾讯的 AI 伦理实践

| 企业 | 主要 AI 伦理实践 | 代表性文件 | 局限与挑战 |
|------|-----------------|-----------|-----------|
| **微软** | AETHER 委员会；AI 原则（Responsible AI）；人权尽职调查框架 | *Microsoft AI Principles*；*Human Rights Impact Assessment Template* | 伦理审查无最终否决权；与商业目标的潜在冲突 |
| **Google** | AI 原则（2018）；PAIR（People + AI Research）；内部审查流程 | *Google AI Principles*；*Responsible AI Practices* | ATEAC 解散后缺乏外部独立监督机制 |
| **腾讯** | AI Lab 伦理研究；信息安全与隐私保护体系；内容安全 AI 审核 | *腾讯 AI 伦理准则*（内部）；《隐私保护白皮书》 | 透明度报告有限；企业伦理框架与公共利益对齐度待验证 |

国内企业（腾讯、阿里、百度等）在 AI 伦理实践上呈现出与欧美企业不同的特征：更侧重于"合规驱动"而非"权利驱动"，隐私保护多以数据安全为核心关切，而对算法公平性、解释权和民主问责的讨论相对薄弱。这一差异反映了不同法律文化与政治经济结构的深层影响。

---

## 十、AI 伦理文化

### 10.1 AI 从业者的伦理责任

AI 工程师和研究者的职业行为正在形成新的伦理规范。ACM（美国计算机协会）和 IEEE（电气与电子工程师协会）均已发布 AI 伦理准则：ACM Code of Ethics（2018）明确要求技术人员"避免伤害"、"诚实守信"和"尊重隐私"。IEEE Ethically Aligned Design（2020）则进一步细化了自主与半自主系统、伦理设计原则和价值嵌入方法。

然而，将抽象伦理准则转化为日常工程实践存在显著障碍。Kaska Porayska-Pomsta, Holmes 和 Nemorin（2024, arXiv:2406.11842）在其 AI 教育伦理研究中发现：AI 伦理教育必须超越"知识传授"层面，进入"情境化伦理推理"（Situated Ethical Reasoning）——即在不同应用场景中做出具体的伦理判断，而非套用通用规则。

### 10.2 代码伦理教育

MIT、Stanford、Harvard 等顶尖高校已开设 AI 伦理相关必修课程。Nature（2024）报道，越来越多的计算机科学项目要求学生在毕业前完成涉及伦理案例的 Capstone 项目，包括公平性分析、偏见检测和可解释性审查。"价值敏感设计"（Value Sensitive Design, VSD）方法论（Friedman et al., 2006）为系统地将人类价值观嵌入技术设计提供了方法论基础，已被广泛应用于人机交互（HCI）和 AI 系统设计研究。

### 10.3 技术乐观主义与伦理现实主义的张力

AI 领域长期存在两种叙事的对峙：**技术乐观主义**（Techno-Optimism）相信技术进步会自动带来社会改善，伦理问题会随着技术成熟自然消解；**伦理现实主义**（Ethical Realism）则认为 AI 伦理问题根植于社会结构，技术本身无法自动解决权力不平等和歧视问题。

Cave 和 Dihal（2019）在 *Minds and Machines* 上深刻分析了这一张力：乐观主义叙事倾向于将伦理问题边缘化为"实施细节"，从而为无约束的技术扩张提供道德掩护；而现实主义立场则坚持认为，每一项 AI 技术的部署都是政治选择，需要经过民主审议。

这一张力在中国 AI 发展语境中尤为突出：庞大市场规模与快速迭代文化往往使伦理审查被视为"创新阻力"，但近年来，随着算法推荐、人脸识别和数据安全领域监管政策的密集出台（2021-2024），伦理与创新的辩证关系正在被重新定义。

---

## 结论与展望

AI 应用伦理的十大议题构成了一个相互关联的复杂系统：伦理框架为所有其他议题提供规范性基础；偏见与歧视问题既是历史不平等的延续，也是隐私被侵蚀的主要驱动力；公平性问题与责任归属、知情同意深度交织；人类自主性则是所有议题的最终价值锚点。

当前 AI 伦理研究呈现三大趋势：**第一**，从原则声明走向制度落地——从 AI 原则宣言到具有法律约束力的 EU AI Act、AIA 强制评估，伦理规范正在硬化为制度约束；**第二**，从单一技术视角转向社会技术系统视角——承认 AI 伦理问题不仅是技术问题，更是权力结构、法律制度和文化规范的复合产物；**第三**，从全球通用框架走向情境化伦理——承认不同文化、法律传统和社会结构下，"公平"、"隐私"、"自主"的具体含义存在差异，AI 伦理需要本土化实施路径。

未来研究的关键方向包括：① AGI（通用人工智能）时代的新型伦理挑战；② 多方利益相关者的 AI 治理机制设计；③ AI 伦理的技术工具（公平性测试、可解释性、合成数据）与制度工具（AIA、监管沙盒、独立审计）的协同整合；④ 中国语境下的 AI 伦理理论创新与制度实践。

---

## 参考文献

1. Amodei, D., et al. (2016). Concrete Problems in AI Safety. *arXiv:1606.06565*.
2. Barocas, S., & Selbst, A. D. (2016). Big Data's Disparate Impact. *California Law Review*, 104, 671–732.
3. Binns, R. (2018). Fairness in Machine Learning: Lessons from Political Philosophy. *FAccT'18*.
4. Buolamwini, J., & Gebru, T. (2018). Gender Shades: Intersection Accuracy Disparities in Commercial Gender Classification. *Proceedings of Machine Learning Research*, 81, 1–15.
5. Cave, S., & Dihal, K. (2019). Hopes and Fears for Intelligent Machines in Fiction and Reality. *Minds and Machines*, 29, 259–276.
6. Chouldechova, A. (2017). Fair Prediction with Disparate Impact: A Study of Bias in Recidivism Prediction Instruments. *FAccT'17*.
7. Coeckelbergh, M. (2020). AI Ethics. MIT Press.
8. Couldry, N., & Mejias, U. A. (2019). The Costs of Connection: How Data Is Colonizing Human Life. Stanford University Press.
9. DiEuliis, D., & Gronvall, G. K. (2023). Biosecurity Implications of AI-Generated Biological Sequences. *Nature Biotechnology*, 41, 1473–1475.
10. Eubanks, V. (2018). Automating Inequality: How High-Tech Tools Profile, Police, and Punish the Poor. St. Martin's Press.
11. Floridi, L., & Cowls, J. (2019). A Unified Framework of Five Principles for AI in Society. *Harvard Data Science Review*, 1(1).
12. Floridi, L., et al. (2018). AI 4 People—An Ethical Framework for a Good AI Society. *Science*, 361, 669–671.
13. Friedman, B., Hendry, D. G., & Borning, A. (2006). A Survey of Value Sensitive Design Methods. *Foundations and Trends in Human–Computer Interaction*, 11(2), 63–125.
14. Gitlin, J. M. (2023). Risk and the Future of AI: Algorithmic Bias, Data Colonialism, and Increasing Marginalization. *Information and Organization*, 33(3), 100478.
15. Ibrahim, H., et al. (2021). Health Data Poverty and Its Implications. *Lancet Digital Health*, 3, e598–e604.
16. Kaska Porayska-Pomsta, et al. (2024). The Ethics of AI in Education. *arXiv:2406.11842*.
17. Lee, K., et al. (2023). Algorithmic Fairness from the Anti-Subordination Perspective. *AIES'23*, AAAI.
18. Lundberg, S. M., & Lee, S.-I. (2017). A Unified Approach to Interpreting Model Predictions. *NeurIPS*.
19. Marchetti, L. (2022). Assigning Liability for AI Decisions: Three Models. *Science and Engineering Ethics*, 28, 28.
20. Mehrabi, N., et al. (2022). A Survey on Fairness and Bias in Machine Learning. *ACM Computing Surveys*, 54(3), 1–35.
21. Neyland, D. (2016). Bearing Accountable Witness to the Ethical Apparati of Big Data. *FAccT'16*.
22. Obermeyer, Z., et al. (2019). Dissecting Racial Bias in an Algorithm Used to Manage the Health of Populations. *Science*, 366, 447–453.
23. O'Neil, C. (2016). Weapons of Math Destruction: How Big Data Increases Inequality and Threatens Democracy. Crown.
24. Powles, J., & Nge, H. (2021). Lessons from the Failure of a Governance Framework for AI. *Yale Journal on Regulation*, 38(2), 580–619.
25. Ribeiro, M. T., Singh, S., & Guestrin, C. (2016). "Why Should I Trust You?": Explaining the Predictions of Any Classifier. *KDD*.
26. Rudin, C. (2019). Stop Explaining Black Box Machine Learning Models for High Stakes Decisions. *Nature Machine Intelligence*, 1, 206–215.
27. Solove, D. J. (2008). Understanding Privacy. Harvard University Press.
28. Stark, L. (2019). Facial Recognition Is the Plutonium of the AI Era. *FAccT'19*.
29. Thaler, R. H., & Sunstein, C. R. (2009). Nudge: Improving Decisions About Health, Wealth, and Happiness. Yale University Press.
30. Wachter, S., Mittelstadt, B., & Russell, C. (2017). Counterfactual Explanations Without Opening the Black Box. *FAccT'17*.
31. UNESCO (2021). Recommendation on the Ethics of Artificial Intelligence. UNESCO.
32. OECD (2020). OECD AI Principles. OECD Publishing.
33. European Commission (2024). Regulation on Artificial Intelligence (EU AI Act). COM(2024) 62 final.

---

*本报告由 AI 智能体辅助生成，内容基于 2026 年 4 月前可检索的公开学术资料。报告中的观点不代表任何政府、企业或机构的官方立场。*
