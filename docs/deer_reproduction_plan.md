# DEER 复现计划（个人设备缩小版）

更新日期：2026-10-23  
目标：先做可信的最小复现，再判断能否发展成研究问题。完整论文结果需要多种大模型和 benchmark，不是本阶段承诺。

## 为什么选它

DEER 是 ICLR 2026 论文，提出在推理过程的转换点生成试探答案，利用该答案的 token 概率决定继续思考或提前结束。作者仓库包含 Hugging Face Transformers 和 vLLM 路径，代码目前主要支持数学推理基准。算法训练成本低，适合把时间用在复现、置信度分析和错误诊断上，也能承接你已有的 vLLM/KV cache 学习。

来源：

- 论文：[ICLR 2026 proceedings](https://proceedings.iclr.cc/paper_files/paper/2026/hash/8df90a1440ce782d1f5607b7a38f2531-Abstract-Conference.html)
- 代码：[作者仓库](https://github.com/iie-ycx/DEER)
- 代码许可证：作者仓库标注 MIT。复现时仍要保留其版权与引用信息。

## 设备约束与诚实范围

本地 RTX 4060 Laptop 8GB 不适合照论文的 7B/14B/32B 全量精度与长上下文配置跑大批量实验。先做一个小模型、一个自动判分任务、少量样本的运行验证；依据显存实测决定模型、量化和最大长度。若需要量化或缩短 token 上限，必须记录，并标为“缩小版复现”。不可把这个结果与论文原始大模型数字直接作优劣比较。

作者 README 推荐 vLLM 路线，但其依赖文件写有较老的 vLLM 上限。不要把旧依赖列表直接安装到主力环境；为 DEER 单独建环境，按当前 vLLM/Transformers 对所选模型的支持情况锁定版本。Windows 原生环境可能受 vLLM 支持限制；若本机使用 Windows，优先通过 WSL2/Linux 环境尝试，或先走作者 Transformers 实现。

## 第一阶段：复现 sanity check

1. 阅读论文方法部分和作者实现中的 `vllm-deer-qwen3.py`、`vanilla_deer.py`、`check.py`，写清楚停止点、置信度计算、续写提示、答案解析分别如何实现。
2. 克隆作者仓库到本项目目录之外的独立文件夹，避免把上游代码和自己的分析代码混在一起：

   ```bash
   git clone https://github.com/iie-ycx/DEER.git
   ```

3. 先跑 20–50 道 GSM8K 题，只确认权重可加载、生成链路工作、答案可判分、输出含必要字段；这个规模不用于结论。
4. 做两个对照：普通推理（不提前退出）与 DEER。模型、提示词、随机种子、最大 token 上限、样本 ID 和解码参数保持一致。
5. 保存逐题结果，至少记录：`sample_id`、`model_revision`、`method`、`seed`、`is_correct`、`generated_tokens`、`wall_time_sec`、DEER 每次试探答案置信度、停止位置、错误类型。
6. 人工检查至少 30 个解析样本，确认 boxed answer / 数值规范化和正确性标签没有系统性错误。

## 第二阶段：可复现主实验

- 数据：GSM8K 先作为开发任务；validation 用于选择阈值，test 冻结后只评一次。采用固定样本 ID，并公开抽样脚本和 seed。
- 方法：Greedy、DEER 默认阈值、validation 调出的 DEER 阈值。若计算预算允许，再加固定 token 上限对照。
- 指标：准确率、平均生成 token、总生成 token、wall-clock time；报告 accuracy–token 曲线和 bootstrap 95% 区间。
- 运行：先单 seed 小规模确认，再扩大样本并至少 3 个 seed；记下每次运行的模型、库版本、显卡、量化和显存峰值。
- 对照公平：同一批题、同一模型和相同解码设定；每个方法的 token 上限与实际 token 都报告。token 数不等价于 FLOPs 或延迟。

## 与研究问题的关系

DEER 在本项目中的定位是置信度早停基线和学习对象，不是预设的新方法。ICML 2026 的 [Conformal Thinking](https://proceedings.mlr.press/v306/wang26hd.html) 已将 Yang et al. (2025) 的 Confidence 信号纳入实验，并在此类置信度早停框架上研究风险控制阈值、有限样本校正、双阈值机制和效率权衡。因此，“在 validation 上校准 DEER 阈值以控制错误率并减少 token”本身不作为本项目的创新主张。

后续如探索小模型、量化、较小校准集或受限算力环境，必须先逐项核对 Conformal Thinking 的模型/数据设置及其验证集规模和分布偏移消融。即使某个条件尚未覆盖，也只能先称为待验证的实证问题，不能直接推断为学术空白或创新贡献。对作者实现和仓库状态的描述应以当前实际检查为准，不沿用旧版本 README 的代码发布状态。

### 可证伪假设

- H1：能否在受限设备上复现 DEER 基线的核心行为与可解释的准确率—成本权衡？
- H2：小模型或特定推理配置下的结果与现有论文报告有何一致或偏离之处？先做相关工作核对再解释差异。
- H3：关于风险阈值、迁移或 token 节省的任何后续假设，都必须与 Conformal Thinking 的方法及实验直接比较后再提出。

## 阶段门与停止规则

| 阶段 | 交付 | 继续条件 |
|---|---|---|
| 环境 smoke test | 20–50 条逐题输出、可复现配置 | 显存可控，判分人工抽查可靠 |
| 基线复现 | Greedy 与 DEER 对照、token 与延迟统计 | DEER 运行逻辑可解释，偏差有记录 |
| 文献定位 | DEER / Conformal Thinking / 近邻方法对比表 | 明确哪些只是复现、哪些问题可能尚未回答 |
| 后续实证问题 | 经文献核对的问题定义、冻结 test 结果、失败分析 | 结论限定在可验证范围内，不把改设定直接说成新颖性 |
| 复试整理 | 报告、代码、图表、演示讲稿 | 明确区分论文结果、本人复现和本人发现 |

若最小模型上 DEER 的置信度字段无法可靠记录，或自动判分无法验证，就先修复实验管线，不推进“新方法”结论。
