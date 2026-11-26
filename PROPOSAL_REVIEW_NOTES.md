# OSPP 申请书样稿对比与 HugeGraph 修改建议

对 `26ec80107`，最值得投入的是写清现有代码可以复用到什么程度、最不确定的执行路径如何尽早验证，以及任务书中的每项交付如何验收。四份样稿共 62 页，篇幅从 8 页到 25 页。样稿与公开中选结果可以帮助理解评审语境，实际中选概率无法从这组材料估算。 [OSPP 项目列表](https://summer.ospp.ac.cn/org/projectlist?lang=zh) · [本项目任务书](Task.md)

## 四份样稿各自值得学什么

| 样稿 | 具体写法 | 对 HugeGraph 的启发 |
| --- | --- | --- |
| [KubeEdge-Sedna 异构边缘 AI 模型部署适配与预检工具](8e3138d9-3bd2-4e07-911d-6463e9605b60.pdf)，8 页 | 第 2 页用部署兼容问题直到 Pod 创建、设备挂载或模型加载才暴露的场景引出需求。第 4、6 页还分别说明了节点识别、运行时制品可用、模型实际运行这三种证据，以及环境缺失时如何计入验收。 | 用一个业务任务说明权限与预算如何影响结果和完成状态。后端支持表也应区分源码有接口、真实调用跑通和通过参考用例。 |
| [BanyanDB 向量化查询追踪与瓶颈诊断](915fcb23-212f-4231-ab85-4010628e03ca.pdf)，11 页 | 第 2–4 页先交代可复用的 Measure 追踪实现，再把 Stream/Trace 的结果一致性、追踪体量和关闭追踪时的开销写成 `AT-1` 至 `AT-9` 验收项。第 7 页给出三个可复现的诊断案例。 | 每条关键需求都配一个具体断言或演示。权限隔离、超时与无路径的区别、作业状态未知时的处理，都可以用小图或固定输入验证。 |
| [Dubbo-Go Triple 协议性能分析与优化](98e94c91-99a0-4111-a2ca-6f4c64cca3bc.pdf)，25 页 | 第 4–12 页交代 WSL2 环境、重复采样、三端初测和每请求分配口径。第 12–23 页把 profiling 结果对应到代码路径和优化方案，并标明 HTTP/2 探索项依据源码核查。 | 选一两项影响方案的前期证据，写明版本、命令、输入、实际结果和适用范围。可以借鉴同环境对照方法，性能承诺应以本项目实际基线为依据。 |
| [增强 Dubbo Admin 的可观测能力](c2d205c1-5938-4ffe-b901-65d83c4400ab.pdf)，18 页 | 第 7–8 页逐项列出现有配置、路由、处理器和 UI 的源码位置及计划改动。第 15–17 页给每周安排可检查的产物，并用一条 trace → log → metric 链路串起端到端验收。 | 「前期调研」写到函数和调用路径。「设计方案」用一条真实可讲清的任务贯穿 SDK、路由、执行和业务对象读取。「项目排期」写清每周能拿什么给导师看。 |

## 对当前申请书的修改顺序

**1. 需求分析先把用户场景和验收条件写清。** 保留现有稿里的三类必需能力和独立安装边界。用一个两跳可达性例子解释成功、权限和预算之间的区别。A、C 均可见，唯一两跳路径经过 B。有权访问 B 的调用者得到可达。无权访问 B 的调用者在完成搜索后得到不可达。搜索因预算耗尽而中断时，结果应明确标记未完成。接着用一张表对应任务书的交付要求、可观察行为和验收证据。把 entry points、作业轮询与回放实现留给后面的技术方案，需求分析保留它们带来的可验证行为。 [现稿需求分析](PROPOSAL.md#需求分析)

**2. 前期调研给导师可以点开检查的证据。** 用「代码位置 · 已确认行为 · 仍需验证」写三到四行，优先选影响设计的发现。

- `GremlinGenerateSynthesize` 接收图模式和示例并生成 Gremlin 文本，可以作为提示方案的参考。结构化 IR、编译校验，以及同模型直接生成 Gremlin 的对照试验属于拟开展工作。 [现有 Gremlin 生成器](hugegraph-llm/src/hugegraph_llm/operators/llm_op/gremlin_generate.py)
- `TraverserManager.k_neighbor` 现有签名接收起点和最大深度。这给出了首个真实调用的切入点。方向、边标签和遍历工作量限制能否通过选定调用路径落实，需要按版本验证。 [遍历客户端](hugegraph-python-client/src/pyhugegraph/api/traverser.py)
- Vermeer 客户端提供创建任务和查找任务的方法。用一次真实作业核对图数据加载、作业终态、ID 映射和参考输出，再确认各算法及其参数的支持范围。 [Vermeer 作业客户端](vermeer-python-client/src/pyvermeer/api/task.py)
- Tiny Library 已有图书对象、借阅规则、审计和测试，可以展示对 Ontogeny 边界的理解。引用时写清实际查阅或复现过的部分。带作者关系的图推理场景仍是拟议演示，当前样例的借阅流程与它对应不同的验收项。 [Tiny Library 说明](ontogeny/domains/tiny-library/README.md) · [现有测试](ontogeny/tests/test_tiny_library.py)

**3. 设计方案用一条请求串起模块，再交代各条路径的风险。** 一张图配一个精简的 `Task`、`Plan`、`Result` 示例即可说明独立包、提供者、HugeGraph 和 Ontogeny 各自负责什么。解释模型先生成 IR、搜索前限定授权图视图、作业提交状态未知时禁止直接重试的原因。为每项附上相应的验证方式，例如图模式引用无效时拒绝向后端提交、在不同授权视图中验证可达性、丢失作业应答时验证提交次数。 [现有技术设计](.workflow/ontogeny-reasoning/design.md)

**4. 排期把最不确定的工作提前。** 沿用 PRD 的前两周目标，交付一次有上限的真实遍历、一次带追踪的自然语言查询和一次对照参考输出的算法作业。由此确认后端版本、调用约束和评测试跑结果。后续每周写「工作 · 可验收产物 · 依赖或风险」，按共享契约和遍历、Text2Gremlin、算法、集成与基准的顺序安排可独立评审的小 PR，并给后端差异、评测修订和社区 review 留出时间。官方指南也建议按周设里程碑，给调试预留约 15% 的弹性。 [PRD 的首个里程碑](.workflow/ontogeny-reasoning/requirements.md#7-first-milestone-verification-and-risks) · [OSPP 申请指南](https://docs2026.summer.ospp.ac.cn/help/student%20application%20advice)

**5. 个人经历用相关成果说明匹配度，摘要最后定稿。** 个人部分优先选能链接的社区 PR、Issue、复现记录或相关项目，写清本人完成的工作、结果和对应的申请任务。每周投入写实际能保证的时间。摘要用一小段交代现有缺口、独立包的方案、四个月的主要交付和验收。官网模板要求申请书包含项目名称、详细技术方案、开发时间计划，简历单独提交。 [OSPP 项目申请模板](https://docs2026.summer.ospp.ac.cn/upload/%E7%94%B3%E8%AF%B7%E6%9D%90%E6%96%99%E6%A8%A1%E6%9D%BF.pdf) · [OSPP 申请指南](https://docs2026.summer.ospp.ac.cn/help/student%20application%20advice)

## 提交前优先确认的技术口径

- **核心交付范围。** 将五项结构化遍历、六项算法和三组基准评测映射到四个月内的验收产物。请导师确认可运行的 Agent SDK 示例是否满足产出要求，以及完整 Agent 编排能否作为扩展项。 [任务书](Task.md) · [PRD 验收范围](.workflow/ontogeny-reasoning/requirements.md#6-deliverables-and-release-acceptance)
- **后端与架构。** 向导师确认首个 HugeGraph 和算法后端组合，以及 RISC-V64 是否要求真机运行。申请书按证据写清查阅、构建、运行与正确性验证达到哪一步。 [技术设计的主要风险](.workflow/ontogeny-reasoning/design.md#8-risks-and-first-milestone-decisions)
- **输出与评测。** 确认距离、排名等计算值在 ID-only 输出要求下的许可范围。提前明确 CypherBench 选定用例如何共享图 ID、比较结果与证据子图，以及哪些失败会进入指标分母。性能以相同数据、模型和资源约束下的对照结果为准。 [PRD 的评测约定](.workflow/ontogeny-reasoning/requirements.md#4-evaluation-and-success-metrics) · [任务书](Task.md)

申请阶段可以把已有功能复现、源码阅读、设计取舍和导师确认记录写成证据。新功能和基准结果随开发进度补齐。OSPP 学生指南将申请阶段的规划设计和中选后的项目开发分别列出。 [OSPP 学生指南](https://docs2026.summer.ospp.ac.cn/help/student%20guide)
