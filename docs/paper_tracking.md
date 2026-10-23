# 论文追踪与阅读顺序

## 研究主题

测试时计算 / 推理时扩展（test-time compute, inference-time scaling）、推理搜索与验证、预算自适应分配、小模型推理效率。

## 初始论文地图

| 论文 | 会议/年份 | 主要问题与方法 | 对本项目的意义 | 必读核查点 |
|---|---|---|---|---|
| [Dynamic Early Exit in Reasoning Models (DEER)](https://proceedings.iclr.cc/paper_files/paper/2026/hash/8df90a1440ce782d1f5607b7a38f2531-Abstract-Conference.html) | ICLR 2026 | 在推理转换点生成试探答案，以 token 概率置信度决定继续或提前回答；training-free | **首个复现主线**：作者公开 Transformers/vLLM 代码，数学任务可自动判分；需在小模型上缩小规模并记录复现偏差 | 论文置信度定义、Qwen3 特殊处理、默认阈值的选择方式、停止 token、代码支持任务和模型范围、实际 token/延迟 |
| [Conformal Thinking: Risk Control for Reasoning on a Compute Budget](https://proceedings.mlr.press/v306/wang26hd.html) | ICML 2026 | 把推理预算选择表述为风险控制，通过验证集选择停止机制以控制错误风险和计算成本 | **紧邻研究，必须先查重**：可作为统计风险控制的理论参照；作者代码仓库目前标注 preliminary | 风险定义和保证依赖哪些假设？停止信号、校准集与 DEER 有何不同？实验规模与模型是否可复现？ |
| [s1: Simple test-time scaling](https://aclanthology.org/2025.emnlp-main.1025/) | EMNLP 2025 | s1K 监督微调与 budget forcing；研究延长/控制推理计算 | 固定预算控制的重要基线；论文使用 32B 模型，不能假设其结果可直接迁移到 8GB 单卡小模型 | 哪些结论来自数据训练、哪些来自 inference-time forcing？小模型能否复现？成本口径是什么？ |
| [Rethinking Optimal Verification Granularity for Compute-Efficient Test-Time Scaling](https://papers.neurips.cc/paper_files/paper/2025/hash/8011b23e1dc3f57e1b6211ccad498919-Abstract-Conference.html) | NeurIPS 2025 | 用可变验证粒度统一 beam search 与 Best-of-N，并提出自适应 VG-Search | 与“动态决定何时继续计算”高度相邻，是避免重复研究的关键论文 | 它的自适应决策是否按题自适应、按过程自适应？使用什么 verifier/模型/预算？公开代码是否可复现？ |
| [Kinetics: Rethinking Test-Time Scaling Law](https://papers.neurips.cc/paper_files/paper/2025/hash/79ada0f9bc4e41192ad5a80d13c8ca7e-Abstract-Conference.html) | NeurIPS 2025 | 将计算与访存成本纳入 TTS 效率分析，讨论模型规模、注意力成本与稀疏注意力 | 提醒不要把更多 token 简化等同于更便宜/更快；评估必须报告实际成本 | 论文的硬件/模型规模结论在本地小模型上是否成立？我们的项目是算法效果还是系统效率？ |
| [CS336: Language Modeling from Scratch](https://cs336.stanford.edu/) | Stanford 课程 | 从 tokenizer、模型架构、训练、数据到 SFT/推理 RL 的完整语言模型链路 | 你的知识主线；本项目更靠近模型推理与 reasoning，而不是重复完整课程作业 | 哪些课程作业已熟练？哪些概念需要补：概率校准、采样、评估、统计检验？ |

## 追踪渠道

- **已接收论文优先：** NeurIPS、ICML、ICLR、ACL、EMNLP、COLM 正式论文集/OpenReview accepted 记录。
- **预印本：** arXiv 用于发现新工作，状态标注为“预印本/未确认录用”，不等同于顶会成果。
- **搜索词：** test-time compute、inference-time scaling、adaptive compute allocation、verifier-guided search、self-consistency、budget forcing、reasoning efficiency、small language models。
- 每次新增论文记录首次公开日期、录用状态、论文版本日期，避免把新预印本误说成已发表论文。

## 单篇论文的筛选问题

1. 它到底解决什么问题，目标指标是什么？
2. 研究对象是训练时算法、推理时算法，还是系统实现？
3. 最强基线与预算是否公平？成本用 token、FLOPs、美元还是 wall-clock？
4. 关键结论需要多大模型、何种 verifier、多少 GPU？是否能在本地缩小复现？
5. 它和本项目假设重合在哪里？未回答的问题是什么？
6. 哪些结果可能来自 prompt、benchmark contamination、模型规模或随机性？

## 研究差异矩阵

每读一篇就更新：

| 工作 | 自适应单位 | 决策信号 | 固定成本口径 | 模型范围 | 数据/任务 | 已解决 | 还不确定 | 与本项目重合/差异 |
|---|---|---|---|---|---|---|---|---|
| VG-Search | 待阅读后填 | 待填 | 待填 | 待填 | 待填 | 待填 | 待填 | 高重合，先确认再定题 |
| s1 | 待阅读后填 | 待填 | 待填 | 待填 | 待填 | 待填 | 待填 | 预算控制基线 |

## 论文阅读顺序

1. DEER：完整理解置信度计算、提前退出和代码偏差，先做小样本运行。
2. Conformal Thinking：检查风险控制与阈值校准候选问题的重合程度。
3. VG-Search 与 `s1`：补齐相邻搜索方法和固定预算控制基线。
4. Kinetics：区分 token budget、实际延迟、显存与访存成本。
5. 根据上述论文 references/citations 补查 2024–2026 的 test-time scaling、self-consistency、verifier/search 与 early-exit 工作，并明确所有预印本和录用状态。

每篇读完复制 [`../templates/paper_note.md`](../templates/paper_note.md) 到 `research_log/papers/`，文件名用 `YYYY-venue-short-title.md`。
