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

## 可能形成的后续问题（必须先查重）

**假设**：DEER 的 token-level 答案置信度可能在题目难度或模型规模变化时失准；在 validation 上按目标错误率校准停止阈值，或许能比固定阈值更稳定地控制错误风险与 token 成本。

这只是候选问题。ICML 2026 的 [Conformal Thinking](https://proceedings.mlr.press/v306/wang26hd.html) 已研究推理预算的风险控制，属于强相关工作；必须读懂其风险定义、适用条件和实验设置，再决定自己的问题是不是有差异。作者仓库当前 README 仍写着 preliminary code，完整代码待发布，因此它现阶段适合作为理论参照，不作为第一篇完整复现代码基线。

### 可证伪假设

- H1：validation 校准的阈值在固定风险目标下，能在独立 test 上维持接近目标的错误率。
- H2：同一阈值跨难度组或模型规模迁移时可能失效；若失效，报告失效范围，不事后调 test 阈值。
- H3：在满足相同错误风险约束时，校准阈值能减少平均生成 token；若不能，假设不成立。

## 阶段门与停止规则

| 阶段 | 交付 | 继续条件 |
|---|---|---|
| 环境 smoke test | 20–50 条逐题输出、可复现配置 | 显存可控，判分人工抽查可靠 |
| 基线复现 | Greedy 与 DEER 对照、token 与延迟统计 | DEER 运行逻辑可解释，偏差有记录 |
| 文献查重 | DEER / Conformal Thinking / 近邻方法对比表 | 研究问题有清楚未回答部分 |
| 阈值研究 | validation 校准规则、冻结 test 结果、失败分析 | 风险控制与成本结果能复核 |
| 复试整理 | 报告、代码、图表、演示讲稿 | 明确区分论文结果、本人复现和本人发现 |

若最小模型上 DEER 的置信度字段无法可靠记录，或自动判分无法验证，就先修复实验管线，不推进“新方法”结论。
