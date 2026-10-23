# 自适应推理计算

**面向语言模型推理的测试时计算自适应分配：复现、评估与边界分析。**

本项目关注推理模型如何根据问题及推理过程中的信号调整测试时计算，以及如何在回答质量、计算开销与推理延迟之间进行可靠比较。项目以可复现实验为基础，比较自适应早停或计算分配方法与预算匹配的基线，并分析这些方法在哪些条件下有效、在哪些条件下会失效。

## 研究背景

推理模型面对不同难度的问题，所需的推理计算并不相同。固定生成预算便于控制，但可能在简单问题上消耗过多计算，也可能无法充分处理困难问题。自适应方法尝试利用答案置信度、答案一致性等信号，决定继续推理、提前停止或分配额外计算。

这一方向已有大量研究，本项目不预设存在尚未解决的学术空白。尤其是，ICML 2026 论文 [*Conformal Thinking: Risk Control for Reasoning on a Compute Budget*](https://proceedings.mlr.press/v306/wang26hd.html) 已系统研究置信度早停与风险控制：论文将 Yang 等人（2025）的 Confidence 信号纳入实验，利用分布无关的风险控制选择停止阈值，并提出基于推理进展的下阈值机制。因此，“给 DEER 的置信度阈值做风险校准”本身不作为本项目的创新主张。

## 项目定位与当前阶段

本仓库目前处于文献梳理、基线验证与研究问题筛选阶段。现阶段目标是建立可信的实验流程，理解并复现具有代表性的自适应推理方法，再根据已有工作与实验结果确定一个边界明确的问题。项目将区分已有论文结论、本人复现结果和后续待验证的观察，不把复现或更换实验设定直接表述为创新。

DEER（Dynamic Early Exit in Reasoning Models）是当前计划中的置信度早停基线和学习对象，而非预设的新方法。未来可以考察更小模型或受限推理环境中的复现情况，但这些只能视作文献核查后的候选实证问题；模型规模、量化方式、校准集大小、任务和分布变化均需与既有研究逐项比较后，才能讨论是否形成有意义的研究贡献。

## 研究范围与评估原则

- 聚焦语言模型推理时的算法与实证问题，不以大规模预训练或重写生产级推理引擎为目标。
- 与可信的固定预算和自适应基线比较，并明确说明预算如何定义及对齐。
- 分别报告准确率、生成 token 数和墙钟延迟；token 数不等同于 FLOPs、显存成本或实际延迟。
- 记录模型版本、提示词、数据切分、随机种子、推理配置与实现差异。
- 报告不确定性、失败案例和适用范围；无论正面还是负面结果，都不超出实验所支持的结论。
- 在完成相关工作对照前，不宣称研究新颖性或性能领先。

## 当前基线：DEER

[DEER（Dynamic Early Exit in Reasoning Models）](https://proceedings.iclr.cc/paper_files/paper/2026/hash/8df90a1440ce782d1f5607b7a38f2531-Abstract-Conference.html) 在推理过程的转换点生成试探答案，并根据答案置信度决定继续推理或提前结束。其公开实现适合作为理解和复现置信度早停的起点。受设备和实验规模影响，本地实验如缩减模型、量化或样本规模，将明确标注为受限复现，并记录与原论文设置的差异。

- [DEER 论文（ICLR 2026）](https://proceedings.iclr.cc/paper_files/paper/2026/hash/8df90a1440ce782d1f5607b7a38f2531-Abstract-Conference.html)
- [DEER 作者代码仓库](https://github.com/iie-ycx/DEER)
- [Conformal Thinking 论文（ICML 2026）](https://proceedings.mlr.press/v306/wang26hd.html)
- [DEER 基线复现方案](docs/deer_reproduction_plan.md)

## 仓库内容

- [`docs/research_plan.md`](docs/research_plan.md)：研究定位、候选问题、实验阶段与阶段门。
- [`docs/paper_tracking.md`](docs/paper_tracking.md)：相关论文地图与研究差异核查问题。
- [`docs/experiment_protocol.md`](docs/experiment_protocol.md)：评估控制、指标与报告约定。
- [`docs/deer_reproduction_plan.md`](docs/deer_reproduction_plan.md)：DEER 基线的受限设备复现方案。
- [`templates/paper_note.md`](templates/paper_note.md)、[`templates/experiment_log.md`](templates/experiment_log.md)：论文阅读和实验记录模板。
- [`research_log/README.md`](research_log/README.md)：研究日志索引。

## 预期成果

项目计划形成一套可复现的评测流程、一份有文献依据且范围清楚的研究问题、预算公平并带不确定性分析的实验结果、对失效情形与局限的讨论，以及一份适合学术交流和研究生复试展示的短研究报告。发表论文或取得领先结果不是预设的项目结论。
