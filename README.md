# 自适应推理计算：研究路线与实验仓库

> 面向考研复试科研展示的个人研究项目。当前处于“研究问题筛选/预研”阶段；这里的方向是候选假设，不代表已经确认学术新颖性。

## 当前选定的复现起点（2026-10-23）

首个主线 baseline 选为 **DEER（Dynamic Early Exit in Reasoning Models, ICLR 2026）**：在推理过程的转换点生成试探答案，根据答案置信度决定停止还是继续思考。作者开放了 Transformers 与 vLLM 实现，且方法无需训练，适合先做小规模算法复现。论文代码和模型规模比你当前 8GB 显存更重，因此第一阶段只做缩小版可复现性验证，不声称复现论文全部结果。

- 论文：[ICLR 2026 页面](https://proceedings.iclr.cc/paper_files/paper/2026/hash/8df90a1440ce782d1f5607b7a38f2531-Abstract-Conference.html)
- 作者代码：[iie-ycx/DEER](https://github.com/iie-ycx/DEER)
- 具体复现步骤与边界：[docs/deer_reproduction_plan.md](docs/deer_reproduction_plan.md)

## 一句话方向

研究小型语言模型在固定推理预算下，能否根据题目难度或早期答案不确定性，动态分配额外的采样与验证计算，从而更好地平衡推理正确率和计算成本。

**当前研究问题（待文献查重）**：DEER 的提前停止置信度，在小型推理模型和题目难度变化下是否可靠？能否只用验证集校准阈值，在预设错误风险下减少 token 消耗？先复现 DEER，再通过实验确认这个问题是否已被直接回答。

目前只是可检验的问题。`s1`、自适应验证粒度和测试时 scaling 等工作已经覆盖邻近方向；必须先完成相关工作核对，再确定具体贡献。见 [`docs/paper_tracking.md`](docs/paper_tracking.md)。

## 为什么这个方向适合当前基础

根据目前能看到的代码与笔记，这不是对能力的定论，而是一个有证据的项目选择：

- 你表示学习过 Stanford CS336；课程涵盖从数据、Transformer 与训练到评估、对齐和推理强化学习。
- 你的 [`nano-vllm-expend`](https://github.com/kimYe121/nano-vllm-expend) 展示了推理链路实现与系统学习：PagedAttention、调度、KV Cache、采样、模型执行；`LEARNNOTE.md` 有结构化模块笔记。
- 该项目的路线图已经列出 speculative decoding，说明你对“推理时怎样分配/节省计算”有持续兴趣；个人设备是 RTX 4060 Laptop 8GB，适合从小模型、固定预算、可复现实验起步，不适合把大规模预训练或 32B 训练当作前提。
- 你已有电影评论分类课设，可说明做过完整的监督学习流程；你也判断它的新颖性不足。因此新项目的重点应从“再做一个应用”转向研究问题、强基线、受控比较与误差分析。

### 当前能力判断（待通过项目验证）

**已有证据较强**：Python/PyTorch 项目实践、把模型系统拆成模块理解、持续记录技术笔记、在算力约束下规划实验。

**还需要用研究过程补强**：提出可证伪假设、读懂并区分相邻论文、设计公平对照、统计不确定性、从失败案例中归纳边界。项目计划将这些当作训练目标，而不是假设你已经具备。

## 研究边界

本项目优先做算法/实证研究，不以重写推理引擎、训练大模型或追求“发顶会”为目标。`nano-vllm-expend` 作为背景能力与后续可选运行后端，不把原仓库改造成研究项目；本仓库独立记录假设、文献、实验和结论。

第一阶段比较：

1. 单次贪心/单样本推理；
2. 固定预算的多样本采样与答案投票（如 Best-of-N / self-consistency）；
3. DEER 作者实现的动态早停；
4. 验证集选择的 DEER 置信度阈值，并分析不同风险目标下的 accuracy–token 成本曲线。

第 4 项是待检验的基线候选，不先宣称为新算法。若文献已充分解决，就调整问题，而非换名包装。

## 目录

- [`docs/research_plan.md`](docs/research_plan.md)：研究问题、假设、边界与阶段计划。
- [`docs/paper_tracking.md`](docs/paper_tracking.md)：论文地图、阅读顺序与文献追踪规则。
- [`docs/experiment_protocol.md`](docs/experiment_protocol.md)：公平比较、指标、数据隔离和复现实验约定。
- [`templates/paper_note.md`](templates/paper_note.md)：单篇论文阅读卡片。
- [`templates/experiment_log.md`](templates/experiment_log.md)：单次实验记录模板。
- [`research_log/README.md`](research_log/README.md)：每周研究日志索引。

## 成功标准

复试项目的成功标准不是“宣称达到 SOTA”，而是能展示：

- 一个边界清楚、可以被实验推翻的问题；
- 至少两个可信强基线和预算公平的对照；
- 可复现的代码、模型/数据/提示词/随机种子记录；
- 正面或负面结果均有误差分析与适用范围；
- 一份短论文式报告和一份 8–10 分钟复试讲解材料。

## 第一周先做什么

按 [`docs/research_plan.md`](docs/research_plan.md) 的阶段门推进。第一周先读主线论文、完成“已覆盖/未覆盖”矩阵，并做本地模型的最小可运行性检查；在文献和基线完成前不投入时间训练新模型。
