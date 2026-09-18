---
title: "SoftwareX — Software-Use Agent 设计与验证计划"
type: project
created: 2026-08-28
updated: 2026-08-28
status: active
area: "[[Research]]"
due: 2027-02-01
priority: P1
tags: [project, research, sigcomm, computer-use, agent, xcap, benchmark]
---

# SoftwareX：Software-Use Agent 的设计与验证

> 配套文档：[[softwarex-software-use agent Xcap as a case]]
>
> **v4 修订（重心翻转）**：本项目的**第一贡献是 agent 的架构设计**，不是 dataset/benchmark。数据与 benchmark 是验证载体，同时因其为实验室多年积累的专有数据而本身构成次级贡献。

---

## 0. 贡献排序（v4 定稿）

| 序 | 贡献 | 性质 |
|---|---|---|
| **① 主** | **Software-Use Agent 架构** —— 把 coding agent 赖以成立的六项机制，逐一为 GUI 软件重新设计 | 系统设计 |
| ② 支撑 | **Substrate**：可寻址状态 / typed action / 合成 verifier | 使能技术 |
| ③ 支撑 | **专有实验数据 + benchmark**：streaming/teleop 跨层 trace，实验室积累，非公开数据集 | 评估载体 + 次级贡献 |
| ④ 发现 | 跨层因果归因的失效边界；synthetic→real 迁移差 | 网络领域结论 |

### 数据作为贡献的正确写法

它是**实验室多年积累的专有数据**，不是公开数据集。这带来一正一负：

- ✅ **正**：真实性无可替代。跨层同步的 streaming / teleop 数据在公开世界里基本不存在，这是别人复制不了的护城河。
- ⚠️ **负**：审稿人一定会问"我能跑吗"。

**应对**（必须提前设计，不能等 rebuttal）：
1. **Agent 与 substrate 全部开源** —— 主贡献是可复现的。
2. **保留公开镜像层（Wireshark）** —— 见 §3，它同时是 agent 的开发环境，任何人都能跑完整流程。
3. **释放派生产物而非原始 trace**：任务定义、ground truth、可核验证据标注、聚合统计。原始数据以"申请获取"形式说明。
4. 论文里明确定位：*公开层负责可复现性，专有层负责真实性验证*。

---

## 1. ⭐ 主贡献：Agent 架构（论文的核心章节）

### 出发点

Coding agent 之所以 work，是因为 repo 恰好白送了六件东西。GUI 商业软件一件都没有。**逐一把它们造出来，就是这篇论文的设计部分。**

| # | Coding agent 白拿的 | 机制 | GUI 世界的缺口 | **我们的机制** |
|---|---|---|---|---|
| 1 | README / 代码可读 | `ls -R`、`grep` | 没有文档，界面语义未知 | **M1 Onboarding：自主 affordance 发现** |
| 2 | `cat` 免费且精确 | 文件读取 | 观测昂贵（截图 token）且有损；482k 行不可能全读 | **M2 分层惰性观测 + 注意力策略** |
| 3 | 编辑操作确定生效 | 编辑器回报 | 点击可能静默失败（焦点/时序/错控件） | **M3 事务化动作：intent → act → verify → repair** |
| 4 | **`git checkout` 让试错免费** | 版本控制 | **GUI 没有 git。误操作可能不可逆，回滚要秒级到分钟级** | **M4 检查点感知的投机探索** |
| 5 | `make test` | 测试框架 | 没有任何内建正确性信号 | **M5 合成 verifier 栈** |
| 6 | `CLAUDE.md` / `skills/` | 人写的持久记忆 | 无人写，无种子 | **M6 自撰软件手册 + skill 库** |

> 这张表就是论文 §3 的骨架，也是消融实验的骨架 —— **六个机制，六组 ablation**。

### 六个机制的设计要点

**M1 — Onboarding（自主认知）**
三级递进：**L0 静态收割**（脚本遍历每个菜单/面板/对话框，dump 状态与截图，无副作用）→ **L1 受控试探**（对每个 affordance 施加动作、观察 state diff、撤销，学出 action→effect 语义）→ **L2 任务驱动探索**。
产出是一份**可验证的软件操作手册**：每条 `action → effect` 断言都被实测过。
> 这是风险最低的贡献 —— 不依赖 skill 学习成功。**建议作为保底结果。**

**M2 — 分层惰性观测**
观测在 GUI 世界是有成本的（这是与 coding agent 最被低估的差异）。设计：状态序列化为可折叠层级树，agent 显式决定展开哪个子树；配额制（每步观测预算）；对高基数视图（482k 条消息）只暴露摘要 + 查询接口而非全量。
**可测指标**：达到同等成功率所需的观测 token 数。

**M3 — 事务化动作**
每个动作是一个事务：声明 intent → 执行 → 用 state diff 核验后置条件 → 不符则诊断并重试/回退。GUI 动作静默失败是 computer-use agent 最主要的错误来源之一，而 coding agent 从不面对这个问题（编辑器会报错）。
**可测指标**：静默失败检出率；无 M3 时的错误传播长度。

> ⭐ **动作空间不是均质的像素平面 —— Filtering 是一个声明式对象**（[[XCAP-KB-FINDINGS]] §5.7.5）：
> 一个 filter = 有序的 `(参数, 运算符 ∈ {=,<>,>,>=,<,<=}, 值, 连接词 ∈ {AND,OR,None})` 列表，有名字、存于 Filter List、可存取。
> **它应当作为 typed action 的第一等公民** —— 可生成、可校验、可 diff，不必用拖拽像素表达。
> 这给 M3 一个重要的设计陈述：*成熟的商业软件内部本就存在结构化的意图层，只是被包在 GUI 里*。agent 的动作空间设计应当去**发现并直达这一层**，而不是一律退化为坐标点击。这与 M6"编译成原生格式"是同一条主线的两端。

**M4 — 检查点感知的投机探索（⭐ 我认为最有新意的一个）**
Coding agent 敢乱试，是因为 `git checkout` 几乎免费。GUI 里回滚昂贵，且有些操作不可逆。
agent 必须维护一个**可逆性模型**（哪些动作安全可试 / 需先建检查点 / 绝不能碰），并做**检查点调度**：在回滚代价与探索收益之间权衡。

> ⭐ **User Guide 给出了明确的三级检查点层级**，M4 可以直接建在上面（各级代价需 E0.4 实测）：
>
> | 级 | 操作 | 代价 |
> |---|---|---|
> | **L1 工作区级** | 清 filter、`All Window Close`、切换/重载 `.wrk` workspace | 秒级 |
> | **L2 Model 级** | `All Close` → 重开已有 Model（*"existing model can be opened **saving parsing time**"*） | 十秒级 |
> | **L3 重建级** | 从 log 重新 parse 建 Model | 分钟级～更久 |
>
> ⚠️ 手册明确 **Filtering 是全局的**（*"Apply filtered data to **all** parameter windows"*）→ 单窗口操作会污染全局，L1 reset 必须显式清 filter。这条也是状态污染的头号来源。

> ⭐⭐ **CLI 已提供全套检查点原语**（[[XCAP-KB-FINDINGS]] §1.1）：`INI`(启动) / `KX`(强杀) / `MOP "<model>"`(重开 Model) / `MDEL`(删) → **三级层级可完全脚本化，不必依赖 GUI 操作**。E0.4 从"设计方案"降为"测各级耗时"。
>
> ⭐⭐ **软件里存在真实的不可逆操作**（非我们杜撰）—— `LTE DCI Analysis` 手册原文：
> > *"The result window does **not saved** and **cannot be re-opened**."*
>
> DCI Analysis 与 RB Scheduling Map 都需先跑昂贵的 `Reprocess`，且结果窗口**易失、关掉即失、无法检查点**，只能重做。
> **论文里直接引用这句话**作为"GUI 没有 `git checkout`"的实证，比抽象论证有力得多。
>
> ⚠️ **另一个复杂度来源**：工具会**产生持久化的派生状态** —— Area/Route Binning 的结果会作为新参数挂进参数树，Route Binning 还能**再生成一个新 Model**。**状态空间可递归增长**，reset 必须清理这些派生物。

> 🔴🔴 **迄今最贴合 M4 的素材：观测面在建模时被冻结**（[[XCAP-KB-FINDINGS]] §5.7.4）。
> `Selective Parsing` 手册建议：*"Select **Log viewer** option before creating the model **to avoid creating Message viewer**"*（更快），而 *"**Reparsing** can be executed when further analysis is needed"*（分钟级）。
>
> 这不是"不可逆"，而是更微妙也更普遍的一类：**可逆但代价高，且代价在做决定的那一刻完全不可见**。
> 一个只想"快点建完模"的 agent 会在第 1 步砍掉自己的观测面，而损失要到第 5 步查证据时才暴露 —— 这正好是当前 GUI agent 最缺的**前瞻性决策**能力，也是 M4 从"回滚调度"扩展为"**信息保留调度**"的理由。
>
> **可直接做成 E3 的诊断任务对**：同一份 log、同一个症状，一条要求 L3 消息级证据、一条不要求。看 agent 会不会做出不同的建模选择。这是对该能力的干净测量，且不需要新数据。

> ⚠️ **环境规格必须钉住 Multi Process 开关**（`tools > options > General > Common`）：开启时 Model 目录下会出现 **sub-model 子目录**，直读 Model 的离线管线会崩；且它会因混合日志格式或 `PAF` **静默降级**为单进程，解析超时则**静默判 Failure 并跳到下一个 job**。详见 GT 计划的新威胁 **T8**。（**不影响 capacity=1** —— 这是单实例内的并行解析。）

这是一个真正的系统设计问题，coding agent 领域根本不存在，且天然对接 live-interaction 预算约束。**建议做成设计部分的招牌机制。**

**M5 — 合成 verifier 栈**
三层 oracle：导出文件比对 / 状态谓词 / **可核验证据断言**（见 §2）。在零演示设定下，**verifier 是唯一的教师**，所以它从工程细节升格为核心技术。

**M6 — 自撰持久记忆（⭐ 已据 User Guide 改设计）**
两类产物，都是人类可读、可 diff、可版本管理的文件，刻意与 `CLAUDE.md`、`skills/` 同构：
- **软件手册**（M1 的产出，跨任务共享）
- **skill 库**：`precondition（对序列化状态的谓词）/ params / 动作程序 / postcondition 断言 / provenance / cost`

> ⭐⭐⭐ **已被支持 KB 强验证**（见 [[XCAP-KB-FINDINGS]]）：**这不再是我们的提议，而是厂商既有的架构。**
>
> XCAP 官方工作流本就分两段：
> ```
> GUI  Tools → Favorite Manager → 勾选参数 → 导出 <name>.fav
>                                     ↓
> CLI  XCAP-M.exe -CM <log> <out> -FEP <name>.fav -MG -SamplingRate
> ```
> 消息侧同理（`Batch Export → Logging Message → Setting` 造 `.mes`），报告侧有 UDR 模板。
>
> **因此 skill 应编译成宿主软件自己的持久化格式**：`.fav`（KPI 模板）/ `.mes`（消息模板）/ UDR 模板 / `.wrk` / `.oxf`。
> 论文里的论证强度因此完全不同：*"专业软件本就区分「编写分析意图」与「批量执行」；agent 只需学会前者，后者是免费的。"*
> 类比从"写 Makefile 而不是记 shell 历史"升级为：**这个软件本来就有 Makefile，只是需要人来写。**
>
> ⚠️ 仍待确认：`.fav` / `.mes` / `.oxf` / `.wrk` 是否**可程序化生成**。若只能 GUI 生成，反而更强化"必须用 GUI"的论证。
> ~~`Chapter 4_Usage of Favorite Info.pdf` 是关键~~ → **已读**（2026-09-01，见 [[XCAP-KB-FINDINGS]] §5.7.1）。它**没给文件格式**（该问题转 W02-X3 直接查看 `.fav` 字节），但给了三条更重要的语义：
>
> | 官方原文 | 对 M6 的含义 |
> |---|---|
> | *"Favorite configuration **is applied for the other Models**"* | ⭐⭐ **skill 跨数据集可迁移** —— 这正是"技能"区别于"一次性脚本"的定义性性质，官方明确支持 |
> | *"Favorite configuration in an analysis type **does not applied for the other analysis type**"* | skill 库必须按 **`(参数集合, 分析类型)` 二元索引**；Graph 上学到的不能用于 Map。这是真实存在的表征约束，不是我们编的 |
> | *"**Recursive** option enables you to open certain parameters under different parameter categories"* | ⭐ **XCAP 唯一的参数化/通配机制**（`RSSI @ Scan Index#*`），对应"把一次性脚本改写成带参数的函数"。**agent 是否主动启用 Recursive** 是 M6 泛化能力的一个干净的可观测指标 |
>
> 因此 M6 的技能表征直接定为：`skill = (params_glob, analysis_type, filters, sampling, → .fav)`，其中 `filters` 已知是结构化四元组列表（§5.7.5），可生成可 diff。**M6 的产物落地格式问题基本解决，只剩序列化通路待验。**

**⭐ 技能的双投影（skill dual projection）**

一个自撰技能同时落到两个地方，两者是同一对象的两个投影：

```
agent 探索出有效组合 → 抽象为 skill ─┬→ ① 编译为 .fav / .mes / UDR   （给宿主软件，跨 Model 可迁移）
                                     └→ ② 注册为一个新的 typed action（给 agent 自己，跨任务可复用）
```

三个收益：

| # | 收益 |
|---|---|
| 1 | ⭐ **技能获取变成可观测的量** —— 动作面随时间增长。「第 k 个任务时有多少个自撰动作」「多少比例的动作由自撰动作而非原语承担」，是 M6 现成的曲线 |
| 2 | **两个投影互为校验** —— `.fav` 可被 CLI 独立执行并导出结果，即"这个技能是否真的有效"有一条**不经过 agent 的验证通路** |
| 3 | **消融变干净** —— B0–B6 阶梯即"启动时给多少动作"：B0 只有 `screenshot`+`click_xy`，B6 允许自撰动作持久化 |

> ⚠️ **红线**：初始动作面必须**语义中立**（见 [[weekly_tasks/W02_2026-09-07_Substrate]] S2）。
> 若我们预先提供带领域词汇的动作（`open_rrc_signaling_view()` 之类），就等于把 M1 与 M6 的答案直接塞进环境 ——
> **动作表就是我们声称不存在的那份软件手册。**

无种子自举：成功轨迹 → 按 state-diff 边界切分 → 常量抽象为参数 → 写成 skill（优先编译为原生格式）→ 在留出实例上验证 → 入库。

### 贯穿性约束：Live-interaction 预算

XCAP 的 **capacity 目前未定**（🔴 **09-09 推翻了 "写死 = 2"**：`Num_of_Process=1`（复选框未勾）时**三个进程 + 三个正常主窗口并存、零弹框**；勾上 `Multi Process Usage` 后第 3 个才被拦 —— **关系与原结论相反**。封口只差一次 2 分钟的实测，见 [[weekly_updates/W02]] §8）。
**规划一律按保守值 capacity = 1 推进**，实测更高再放宽。
而**探索本身就在消耗这个稀缺资源**。所以整个架构在一个显式预算下运行：

`maximize Performance − λ · N_live_interactions`

由此产生本项目独有、且在免费 simulator 环境里根本不存在的问题：

> **固定 N 次真实软件交互，多少花在"认识软件"（M1），多少花在"完成任务"？**

---

## 2. 关键架构决定：Ground Truth 与被测软件完全解耦

```
   raw network traces
          │
          ├───────────────────────┐
          ▼                       ▼
  【离线 GT 管线】           【被测 GUI 环境】
  脚本 / tshark / 注入标签     XCAP (capacity 待定)
  免 license·可并行·免费       Wireshark (可并行)
          │                       │
          ▼                       ▼
   task + answer + 可核验证据 ◀── agent 作答
          │                       │
          └─────── Evaluator ─────┘
```

**推论**：
1. 任务与 GT 的生成**完全不占 license**，9 月就能造出上千任务，排期风险大降。
2. GT **不需要专家在软件里操作** —— 根因从原始 trace 离线确定。
3. **证据的定义**：不是"点击路径匹配专家演示"（无演示数据，做不到），而是**一组可对 trace 程序化核验的断言**：
   ```json
   {"answer":"handover interruption",
    "evidence":[{"type":"event","name":"RRCReconfiguration","t":12.481,"pci":301},
                {"type":"event","name":"RLF","t":12.607},
                {"type":"metric","name":"uplink_gap_ms","value":126,"window":[12.48,12.61]}]}
   ```
   更便宜，且不偏向任何一位专家的工作流。
4. **可答性过滤按模板而非按实例**：live 验证 30–50 个模板可答 → 离线批量扩成数百实例。

---

## 3. 环境三层（Wireshark 的角色已升级）

```
        Agent 开发环境          真实性验证        外部可复现
             │                     │                 │
        Wireshark              XCAP + 专有        Wireshark
        可并行·可换版本         streaming/teleop    公开发布
             │                     │                 │
      M1–M6 迭代打磨          最终验证 + 头条结果   审稿人可跑
      受控 UI 漂移消融         capacity 待定
```

**v4 的重要改动**：Wireshark 从"可复现性摆设"提升为 **agent 的主开发环境**。理由是主贡献变成 agent 设计，而 agent 的迭代打磨绝不能被 XCAP 的实例上限卡住。**M1–M6 在 Wireshark 上开发成熟，再移植到 XCAP 验证。** 这同时天然构成"跨软件泛化"证据。

跨域第二 app（LibreOffice/QGIS）仍然砍掉 —— 泛化由 Wireshark↔XCAP 与 streaming↔teleop 两条轴承担。

### ⭐ v5 新增：XCAP ↔ Wireshark 数据桥 → 一个别人做不了的对照实验

User Guide 确认两条通路：
- **Import**：`CSV, txt, Tektronix, Ethereal(=Wireshark), TEMS`
- **Converter**：`Convert to PCAP (Ethernet PPP)` —— *"Extract PCAP data from Log data. (If analyze in Wireshark, use the corresponding Converter.)"*

于是可以构造 **paired tasks：同一份底层数据、同一个问题，在 XCAP 与 Wireshark 两个界面各做一遍**。

> 领域推理完全相同，差异**全部来自 UI**。这把「软件操作能力」与「领域推理能力」干净地分离开 ——
> 这是一个现有 computer-use benchmark 都做不到的消融，因为它们的任务无法在两个软件里等价复现。**建议作为论文的一张核心图。**

---

## 4. 时间线（22 周，agent 设计已扩容到 11 周）

| Phase | 周期 | 内容 |
|---|---|---|
| **P0 可行性** | 9/1 – 9/7 | E0.0 数据兼容性 + **聚合语义标定**（🔴）、E0.1 UIA 探针（**含拖放**）、E0.2 hatch 实测、E0.3 RPC 边界、E0.4 **检查点层级实测**、E0.7 ~~Multi Process~~ ✅ 已结（capacity 修正为 **2**） |
| **P1 Substrate** | 9/8 – 9/28 | 状态序列化 / typed action / verifier 栈 / recorder；XCAP 与 Wireshark 双接入 |
| **P2 离线 GT 与任务** | 9/8 – 10/19 **并行·不占 license** | 数据索引、故障分类学、oracle-first 生成器、synthetic tier ≥300 任务 |
| **P3 M1 Onboarding** | 9/29 – 10/19 | L0 收割 / L1 受控试探 → 软件手册；模板可答性验证 |
| **⭐ P4 Agent 设计** | **10/6 – 12/20（11 周）** | **M2–M6 逐个实现与消融，主战场在 Wireshark，XCAP 定期验证** |
| **P5 Baselines** | 11/1 – 11/25 | B0–B6 阶梯，**感恩节前出图** |
| **P6 实验与写作** | 12/1 – 2/1 | 12/10 开写；1/5 freeze；1/5–1/20 XCAP 最终评测；1/20 内审 |

关键日期：**12/10 动笔**（不是 1 月）；**1/5–1/20 XCAP 串行评测预留 10 个工作日**（`4 agent × 100 任务 × 2 seed ≈ 800 runs`，3 min/run）。多 seed 全部放 Wireshark / synthetic。

---

## 5. Baseline 阶梯（对应六机制的消融）

| | Baseline |
|---|---|
| B0 | 人类专家（~20 例校准会话，仅作上界与定性对照） |
| B1 | 纯截图 + 点击（OSWorld 式） |
| B2 | B1 + accessibility tree |
| B3 | B2 + **状态序列化 + typed action**（substrate） |
| B4 | B3 + **M1 软件手册** |
| B5 | B4 + **M2/M3**（惰性观测 + 事务化动作） |
| B6 | B5 + **M4/M6**（检查点投机探索 + skill 库）= 完整系统 |

❌ 删除 imitation / behavior-cloning 类 baseline —— **无演示数据，不适用**。论文里写成设定，不是缺陷。

---

## 6. Metrics

| 类别 | 指标 |
|---|---|
| 任务 | Success Rate（按任务类与 workload 分报） |
| **RCA** | Answer accuracy / **证据可核验率** / 层归因准确率 |
| **⭐ 跨层** | **层间混淆矩阵**（RAN 被误判为 transport 等）—— 最"网络"的一张图 |
| **⭐ M1** | 手册覆盖率；action→effect 准确率；下游增益（B3→B4） |
| **⭐ M2** | 达同等成功率所需的观测 token |
| **⭐ M3** | 静默失败检出率；无 M3 时的错误传播长度 |
| **⭐ M4** | 检查点次数 vs 探索收益；不可逆误操作率 |
| **⭐ M6** | skill 复用率；复用成本降幅（25 → 4 actions）；**streaming→teleop 语义迁移增益**；漂移修复率与 episodes-to-repair |
| **⭐ 预算** | 固定 N 下"探索占比 vs 性能"曲线；ISE(k) |
| **⭐ Sim2real** | `SR_synthetic − SR_real`，分任务类 |
| 环境 | RSR / EFR / T_recover / state contamination |

---

## 7. 研究问题

- **⭐ RQ1（主）** Coding agent 的六项赖以成立的机制，能否为封闭 GUI 软件重新设计出来？各值多少分？
- **⭐ RQ2** 零文档零演示下，agent 能否自主建立一个封闭商业软件的**可验证操作模型**？（M1，**保底结果**）
- **⭐ RQ3** 在回滚昂贵且部分不可逆的环境中，如何调度检查点与投机探索？（M4，**招牌机制**）
- **RQ4** 无种子的 skill 自举能否降低后续同类任务成本？（M6，对话 SkillsBench 的否定结论）
- **RQ5** 固定 live 交互预算下，探索与利用如何分配最优？
- **RQ6** 合成故障上的表现能否预测真实故障？跨层因果推理的失效边界在哪？

**风险结构**：RQ2 保底（几乎一定做得出）→ RQ1/RQ3 主线 → RQ6 头条（正反结果都能发）→ RQ4/RQ5 加分（失败可作为与 SkillsBench 一致的负面结果）。**这个结构是健康的。**

---

## 8. 风险与止损

| 风险 | 触发 | 应对 |
|---|---|---|
| **🟡 XCAP 打不开你们的数据**（已降级） | E0.0，**本周** | 手册确认支持 `CSV/txt/Tektronix/Ethereal(pcap)/TEMS` 导入 → 风险下降。重点转为"导入后**保留哪些层**"；退路仍是转换 / 补采 / 换锚点 |
| **🔴 T5 分析配置决定真值** | E0.0 聚合语义标定 | 默认 **Sampling Rate = 1 sec 平均**（事件/信令不采样）。若离线 GT 算瞬时值，所有指标题会系统性错位**且不被察觉**。见 [[TASK-GT-GENERATION-PLAN]] §1-T5 |
| 🔴 跨层时间戳对不齐（ms 级） | 数据盘点，9/19 | 降级为单层 RCA，重划头条。注：XCAP 自带 `Range to Sync` / `Sync Manager` 可能已提供对齐 |
| ✅ ~~**Multi Process 推翻 capacity=1**~~ | ~~E0.7~~ | **已结**：Multi Process 是单实例内并行解析，不推翻。<br>🔴 **09-09 再次推翻**："写死 = 2" 不成立，且方向相反。**capacity 待封口，规划按 1** |
| **无 Map 引擎授权** | E0.1 | 手册：Map 窗口需 MapX/MapXtreme 引擎并重启生效。无授权则 Map 类任务全部不可做，任务分类学要相应删减 |
| 🔴 synthetic 无层标签 | 数据盘点 | RQ6 混淆矩阵做不了；补做受控注入需**提前 8 周排期** |
| **审稿人质疑数据不可复现** | 提交 | §0 的四条应对必须在正文，不能等 rebuttal |
| XCAP UIA 不透明 | G0，9/7 | 转 OCR+vision；substrate 贡献反而更大 |
| 隔离机不能出网 | E0.3，第一周 | 第一天做 RPC 分离（controller 联网侧 / worker 隔离侧），成本为零 |
| **agent 迭代被实例上限卡死** | 10 月起 | **主开发在 Wireshark**；replay 语料靠 M1 收割增量填充。⚠️ ~~capacity=2 允许一实例跑 agent、一实例跑 oracle/CLI~~ **该假设已撤回** |
| 六个机制做不完 | 11 月中 | 优先级 M1 > M5 > M3 > M4 > M2 > M6，按此顺序砍 |
| 1 月跑不完 | 1/15 | 预定最小可发表集（2 agent × 60 任务 × 1 seed）先跑 |

---

## 9. 本周立刻要做

- [ ] 🔴 **E0.0：建 Model —— 确认能载入、走哪条导入路径、导入后保留哪些层**
- [ ] 🔴 **聚合语义标定**：同一参数用 `All` 与 `1sec` 各建一个 Model，量化读数差异（T5）
- [ ] 🔴 **E0.7 重开**：Multi Process 是单实例内并行解析（这半仍成立）；但 ~~实例上限 = 2（写死）~~ **09-09 被推翻，待封口**
- [ ] 🔴 **pgDatabase 是否可只读连接** —— 若可，离线 GT 管线成本骤降
- [ ] 🔴 **`File Export using Command Prompt` 语法与覆盖范围** —— XCAP 的 `cat`
- [ ] 🔴 **确认 synthetic 性质**：(a) 受控故障注入（有层标签，RQ6 成立）还是 (b) 程序化生成任务（无层标签）
- [ ] 检查跨层时间戳对齐精度（app 指标 ↔ RAN trace 是否到 ms）
- [x] ~~确认 SIGCOMM '27 deadline~~ → **已确认：deadline ≈ 2027-02-01，正文 12 页**
- [ ] **决定数据释放策略**（派生产物 vs 原始 trace vs 申请制）
- [ ] **索取随附分册 PDF**（尤其 `Tools_Analysis` 与 `Tools_Setting_Options`）+ 命令行/数据库文档
- [ ] 盘点人力、key 数量、可得专家小时、**Map 引擎授权**
- [ ] E0.1 UIA 探针（半天，**务必含拖放**）
- [ ] 建 repo `softwarex-agent/`（主体是 agent），第一个 commit 就有 RPC 边界
- [ ] 写 M1-L0 静态收割脚本原型（7 个分析 tab + Message/Signaling Flow/Synchro Viewer）

---

## Actions

### P0 可行性（9/1 – 9/7）
- [ ] E0.0 数据兼容性 + **聚合语义标定** ← 最高优先
- [ ] E0.1 UIA 探针（**含拖放**：参数树 → 工作区，Alt+拖放叠加）
- [ ] E0.2 hatch 实测（pgDatabase / CLI export / `.oxf`·`.wrk` 优先），详见 [[TASK-GT-GENERATION-PLAN]] S0.2
- [ ] E0.3 隔离机拓扑 + RPC 边界
- [ ] E0.4 **三级检查点代价实测**（工作区级 / Model 级 / 重建级）—— M4 的物理基础
- [ ] E0.5 输入注入稳定性 / E0.6 冷启动脚本 + L0 原型
- [x] ~~E0.7 Multi Process 到底是什么~~ → 已结，见上；capacity 由 1 修正为 **2**
- [ ] Gate G0 决策会

### P1 Substrate（9/8 – 9/28）
- [ ] 状态序列化（稳定 / 可 diff / **可折叠层级 —— M2 的前提**）
  - ⚠️ **归一化分两类**（[[XCAP-KB-FINDINGS]] §5.5.4）：*表现性*字段（滚动偏移、光标）归一化掉；**语义性字段（窗口尺寸、图表可视范围）必须保留并纳入 `analysis_config`** —— 手册明确 *"export is processed according the displayed size"*，**导出结果取决于窗口尺寸**
- [ ] Typed action layer（**返回 state diff —— M3 的前提**）
- [ ] Verifier 栈（M5）
- [ ] Recorder；XCAP + Wireshark 双接入
- [ ] R1 文档：GUI 软件的 repo / diff / make test 是什么

### P2 离线 GT 与任务（9/8 – 10/19，不占 license）
- [ ] 数据索引（workload / app 指标 / 层标签 / **时间对齐精度**）
- [ ] workload × 层 故障分类学
- [ ] Oracle-first 生成器 + 可核验证据 schema
- [ ] synthetic tier ≥300 任务

### P3 M1 Onboarding（9/29 – 10/19）
- [ ] L0 静态收割 → UI map + affordance 清单
- [ ] L1 受控试探 → action→effect 软件手册
- [ ] 30–50 模板的 live 可答性验证
- [ ] R2 文档：零演示下的软件自主认知

### ⭐ P4 Agent 设计（10/6 – 12/20，主战场 Wireshark）
- [ ] M2 分层惰性观测 + 注意力策略
- [ ] M3 事务化动作（intent → act → verify → repair）
- [ ] M4 检查点感知的投机探索 + 可逆性模型
- [ ] M6 skill 无种子自举 / 检索 / 组合
- [ ] streaming → teleop 语义迁移实验
- [ ] 漂移注入（Cosmetic·Layout·Naming·Structural·Semantic）与 skill 修复
- [ ] 探索-利用预算实验（RQ5 主图）
- [ ] 移植到 XCAP 并定期验证
- [ ] R3 文档：架构定稿 + claim 表（claim ↔ 图 ↔ 实验）

### P5 Baselines（11/1 – 11/25）
- [ ] B0–B6 阶梯
- [ ] **感恩节前拿到阶梯图**

### P6 实验与写作（12/1 – 2/1）
- [ ] 12/10 开写；1/5 freeze；1/5–1/20 XCAP 评测；1/20 内审；提交

---

## Notes

- **v4 净变化**：贡献排序翻转，agent 架构升为第一贡献；新增六机制表（M1–M6）作为论文 §3 骨架与消融骨架；agent 设计期从 6 周扩到 11 周并提前到 10/6；Wireshark 从"可复现性摆设"升为 **agent 主开发环境**；新增 M4 检查点投机探索为招牌机制；数据定位为专有积累并配套四条可复现性应对。
- **最有新意的两个点**：**M4**（GUI 没有 `git checkout`，回滚昂贵且部分不可逆 → 检查点调度问题，coding agent 领域不存在）与 **M1**（零文档下自动产出可验证的软件操作手册）。前者是招牌，后者是保底。
- **最容易翻车的四处**：① E0.0 拖到 10 月才验 ② 六个机制都想做深 ③ 1 月才动笔 ④ agent 迭代被实例上限卡死（务必主开发在 Wireshark）。

### v7 净变化（据安装目录 4 份分册，2026-09-01，见 [[XCAP-KB-FINDINGS]] §5.7）

| # | 变化 | 影响面 |
|---|---|---|
| 1 | ⭐⭐ **M6 的技能格式落定**：Favorite **跨 Model 可迁移**、跨 analysis type 不可 → skill 库按 `(参数集合, 分析类型)` 二元索引 | M6 从"提议"变为"有官方语义支撑的表征" |
| 2 | ⭐ **Recursive = XCAP 唯一的参数化机制**（参数树 glob）→ "agent 是否主动启用 Recursive"成为 M6 泛化能力的可观测指标 | M6 / E5 |
| 3 | ⭐ **M3 拿到第一个真正结构化的动作**：Filtering = `(param, op, value, linker)` 四元组列表，可生成可 diff | M3 typed action 的范例 |
| 4 | 🔴 **Filter 是全局的** → `apply_filter` 的 state diff 必须扫全工作区；`ckpt.workspace()` 清 filter 由"保守"变"必需"，且范围确认为全局一次即可 | M3 / M4 |
| 5 | 🔴 **新威胁 T8**：Multi Process 改变 Model 磁盘结构（sub-model 子目录）+ 静默降级 + 超时静默判 Failure | GT / 离线管线 / 环境规格 |
| 6 | ✅ **U5 关闭，capacity=1 成立** —— Multi Process 是单实例内并行解析，不是多实例。W02 的 G0 分支消除一条 | 时间线**不变** |
| 7 | 🔴🔴 **M4 拿到最好的素材（T8b）**：建模时选 Log viewer 会冻结观测面，*可逆但代价高、且代价在决策时不可见* → M4 从"回滚调度"扩展为"**信息保留调度**"；配 E3 诊断任务对 | M4 / E3 |
| 8 | ⚠️ **Selective Parsing 仅支持 Qualcomm LTE** → GT schema 的该字段须按 `(chipset, RAT)` 门控 | 任务生成器 |
| 9 | ⭐ **《沉默的破坏性默认值》成为论文 §4 的一个小节** —— 三个可引用原文的实例（Area Binning 覆盖 / 多 log 导出覆盖 / 采样率默认 1s） | 论文 §4，支撑 M3+M5 的必要性 |
| 10 | `.fav` **文件格式**仍未知（手册只给语义）→ 成为 M6 唯一的未知，W02-X3 优先级上升 | W02 |
| 11 | W01：A7 改写（U5→T8a 验证）、新增 A7'（T8b 计时）、A6 增 Favorite 迁移实测、C1' 关闭（4/5 到手），**仅剩 `pgDatabase Usage`** | W01 |

---

### v6 净变化（据支持 KB + 17 份分册 PDF 精读，见 [[XCAP-KB-FINDINGS]]）

| 变化 | 影响 |
|---|---|
| **CLI 完整命令表已获取** | `INI/FIN/KX/CM/MG/PAF/MOP/MDEL/UTC/FEP/MEP/PGEP/UDREP` + 11 档采样率 + 通配符。**U6 已答**，A6 从探索改为验证 |
| 🔴 **`PGEP` 证实 pgDatabase 通路** | 离线 GT 管线可建在 PostgreSQL 上 |
| ⭐ **M4 检查点原语可脚本化** | `KX`/`MOP`/`MDEL`/`INI`，E0.4 工作量大降 |
| ⭐ **找到真实不可逆操作的实证** | DCI Analysis 结果窗口"不保存、不可重开" |
| ⭐ **M6 = 厂商既有架构** | GUI 造 `.fav`/`.mes`/UDR 模板 → CLI 批量消费。论证强度质变 |
| 🔴 **T5 升级：存在静默覆盖** | Area Binning 对 Sector/PSC 类参数**无视你选的公式，强制用 Mode**。钉死配置必要但不充分 |
| **T5 配置空间远超预期** | 采样率 11 档、分箱公式 9 种、Message Duration 的 First/Last 与呼叫区间、`UTC`、窗口尺寸、并列 tie-break |
| ⚠️ **窗口几何是语义承重的** | *"export is processed according the displayed size"* —— 原"归一化掉窗口几何"的设计规则是错的 |
| ⚠️ **派生状态会递归增长** | Binning 结果挂进参数树，还能再生成新 Model |
| 🔴 **CLI 建模不支持 pcap/CSV** | 仅 `drm/drx/scn/scx/ISF/DLF`（Scheduler 另加 `aof/aox/zip`）。**pcap/CSV 只能走 GUI Import → 数据准备无法自动化**，直接影响任务规模。E0.0 关键约束 |
| ⭐ **Scheduler 可无人值守** | Local/FTP 模式、失败进 `Quarantine` 文件夹 → 现成的"环境失败 vs agent 失败"分流 |
| ⭐ **Release Note 富矿** | 一年 7+ 个 XCAP 版本 → M6 漂移实验可用**真实版本演化**，不必人工注入，且就在 XCAP 上做 |
| ⭐ **现成任务模板来源** | Message Duration Time（内建 min/max/avg + 阈值）、Area Binning Result（内建阈值统计）、Finding Missing Neighbors（三参数检测器）、Event Call Extract（**故障案例批量切片**） |
| ⚠️ **Map 引擎是硬依赖** | Area Binning、地图类任务全部依赖 MapX/MapXtreme 授权。C4 若为否，任务分类学要删减 |

### v5 净变化（据 XCAP User Guide 5.33 修订）

- **E0.2 基本被预先回答**：手册确认存在 `File Export using Command Prompt`、`pgDatabase`、`Batch Export`、`Scheduler`、`Multi Process`、`User Defined Report/KPI`、`.oxf`/`.wrk` 导入导出。重点从"有没有"转为"能拿到什么"。
- **M6 改设计**：skill 优先编译成 XCAP 原生的 `.wrk` + `.oxf` + Template/Favorite，而非回放点击 —— 对漂移天然免疫，且构成"agent 写入宿主软件自身持久化格式"这一设计主张。
- **M4 有了现成的三级检查点层级**（工作区 / Model / 重建），各级代价待 E0.4 实测。
- **新增 XCAP↔Wireshark 配对实验**（Import 支持 Ethereal，Converter 支持 Convert to PCAP）→ 同数据同问题两个 UI，干净分离"软件操作能力"与"领域推理能力"。**建议做成核心图。**
- **新增 T5 风险**：分析配置（默认 1 秒平均采样、Selective Parsing、全局 Filter、Binning）决定真值本身。详见 [[TASK-GT-GENERATION-PLAN]]。
- **E0.0 降级、E0.7 新增**：数据兼容性风险下降；`Multi Process` 若能并行则 capacity=1 假设推翻。（→ v7 曾结论"实例上限 = 2"，**09-09 推翻，待封口**。）
- **两个新的具体障碍**：交互大量依赖**拖放**（GUI agent 的硬骨头，M3 要专门处理）；Map 窗口需 **MapX/MapXtreme 引擎授权**，无则 Map 类任务全部不可做。
