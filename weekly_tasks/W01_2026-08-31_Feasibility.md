---
title: "W01 可行性穿刺 (8/31 – 9/4)"
type: project
created: 2026-08-28
status: active
area: "[[Research]]"
week: 1
priority: P1
tags: [project, research, sigcomm, weekly-task]
---

# W01 — 可行性穿刺
**2026-08-31（一） → 09-04（五）** · Gate G0 于周五

> 母计划：[[PLAN-SoftwareX-Sandbox-Benchmark]] · [[TASK-GT-GENERATION-PLAN]] · [[PAPER-STRUCTURE-AND-STORYLINE]]

---

## 0. 本周唯一目标

> **本周不建造任何东西。本周的工作是把六个不确定性变成六个 yes/no。**

任何"顺手把 X 也做了"的冲动都要压住 —— 那是 W02 的事。

| # | 不确定性 | 由谁回答 | 当前状态 |
|---|---|---|---|
| **U1** | XCAP 能否吃我们的数据？保留哪些层？ | A1–A3 | 🟡 **风险已降**：确认支持 `CSV/txt/Tektronix/Ethereal(pcap)/TEMS` 导入。<br>🔴 **但发现硬约束**：CLI 建模仅支持 `drm/drx/scn/scx/ISF/DLF`，**pcap/CSV 只能走 GUI Import → 数据准备无法自动化**。CSV 导入还要求前 4 列为 `Day/Time Stamp/Latitude/Longitude`。**仍需实测** |
| **U2** | 哪个时钟是权威的？（T6） | B2 + A4 | 🟡 **先验很强**：官方文档明写 *"displays Time Difference value using **Chipset Time** between two Messages"*；样本 3 行也显示 `Time` 列间隔仅 5–6 µs（像排序键）。**仍需整条 trace 验证 + 漂移模型** |
| **U3** | 开源 diag parser 通路是否成立？ | B1 | 🟡 **结构已验证**：手工解出 `[外层15B][inner_len 2B][log_code 2B][QC 64位 ts 8B][payload]`，`0xB064`/`0xB087` 与 Message 列一致。**parser 尚未写，逐字段对账未做** |
| **U4** | XCAP 的 UI 是否机器可读？ | A5 | ⬜ **未动**，需上机 |
| **U5** | capacity 真的是 1 吗？ | ~~A7~~ | ✅ **已答：是 1。** `Multi Process` 手册（安装目录，2026-09-01 到手）说明它是**单实例内**把 log 切 2 组并行解析再合并，不是多实例。**U5 关闭，capacity=1 假设成立，时间线不变。** A7 改为验证 T8a（目录结构影响） |
| **U6** | 能程序化拿到什么？ | A6 | ✅ **基本已答**：拿到完整 CLI 命令表（`INI/FIN/KX/CM/MG/PAF/MOP/MDEL/UTC/FEP/MEP/PGEP/UDREP` + 11 档采样率 + 通配符）。**剩下的是上机验证语法，不是探索** |

**周五必须能在一页纸上写出这六个答案。**

---

## 0.5 ✅ 开工前已完成的桌面研究（8/28–8/29）

W01 正式开工前，已通过文档与样本分析提前解决了一批工作。**这直接改变了本周的任务分配**（见各条的 🔄 标注）。

| 已完成 | 产出 | 对 W01 的影响 |
|---|---|---|
| ✅ 通读 `XCAP User Guide 5.33`（55 页） | 7 种分析视图、参数树、Model 概念、全局 Filter、采样语义 | 多项任务从"探索"降为"验证" |
| ✅ 抓取支持 KB 全部 3 页 41 篇文章 | [[XCAP-KB-FINDINGS]] | — |
| ✅ **下载并精读 17 份分册 PDF** → `xcap_docs/` | CLI 完整命令表、Scheduler、Batch Export、Area Binning、DCI Analysis、Message Duration、Missing Neighbors、DSDS 等 | 🔄 **C1/C2 大部分已完成** |
| ✅ 分析导出样本 `sample data.md` | hex = 标准 QC diag packet（已解码验证）；两时钟差 17.6 s 且漂移；CSV 表头 8 列/数据 10 字段错位 | 🔄 **B1/B2 的可行性已确立** |
| ✅ 任务阶梯 L1–L5 设计定稿 | [[TASK-GT-GENERATION-PLAN]] §0.5 | 🔄 **B3' 只剩写样例** |
| ✅ T5 升级：配置空间清单 + **静默覆盖**发现 | 采样率 11 档、分箱公式 9 种、Sector/PSC 类参数**无视所选公式强制 Mode** | 🔄 **A4 标定实验范围扩大** |
| ✅ T7 被官方证实 | RLC DL Dropped PDUs = 计数器 1 秒内差分（109→228 差 119） | 🔄 B3 现象清单必须带语义类型列 |
| ✅ M4 检查点原语确认 | `INI`/`KX`/`MOP`/`MDEL` 可脚本化；发现真实不可逆操作（DCI Analysis 结果窗口"不保存不可重开"） | 🔄 **A8 从"设计方案"降为"测耗时"** |
| ✅ M6 得到厂商架构验证 | GUI 造 `.fav`/`.mes`/UDR 模板 → CLI 批量消费 | 🔄 新增"检查模板文件是否文本可编程生成"（→ W02-X3） |
| ✅ Release Note 清单已定位 | XCAP 线 7+ 篇（`20250828 O5.33.7.18` 等） | 🔄 **新增 B6**，可离线做 |
| ✅ 现成任务模板已识别 | Message Duration（内建 min/max/avg+阈值）、Area Binning Result、Finding Missing Neighbors、**Event Call Extract（故障案例批量切片）** | 🔄 W02-T2 可直接收割 |

### ⬜ 桌面研究未能解决的

| 仍缺 | 备注 |
|---|---|
| `Usage of Favorite Info` / `Multi Process` / `pgDatabase Usage` / `Selective Parsing` / `Filtering` 分册 | **KB 里没有独立文章**。User Guide 反复说 *"that comes with it"* → **很可能随软件安装** → 🔄 已并入 **A1** |
| SIGCOMM '27 CFP | C6 未做，仍需查 |
| RAN troubleshooting 成本数字 | C7 未做，Intro 的地基 |
| 所有需要上机的项（Track A 全部） | 无机器，全部待做 |

---

## 1. 三条并行轨道

```
Track A  ── XCAP 机器（串行，capacity=1，最稀缺）
Track B  ── 离线（可并行，不占 license，人多就多开）
Track C  ── 异步请求与行政（长周期，必须周一上午就发出）
```

> ⚠️ **Track C 周一上午必须发出**。索取文档、确认 license、约专家都有等待周期，晚一天就压到下周。

---

## 2. Track C — 周一上午发出（30 分钟，回报最大）

| ID     | 状态            | 任务                                                                                                                                                                                                                 | 收件人                     | DoD                                   |
| ------ | ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------- | ------------------------------------- |
| **C1** | 🟡 **大部分已完成** | ~~索取随附分册 PDF~~ → **17 份已从 KB 下载到 `xcap_docs/`**。<br>**仍需索取 KB 里没有的 5 份**：`Usage of Favorite Info`（最关键，决定 M6）、`Multi Process`、`pgDatabase Usage`、`Selective Parsing`、`Filtering`。<br>⚠️ 先做 A1 —— 它们可能就在安装目录里，那就不必发信 | 厂商 / 内部 IT              | 5 份到手，或确认在安装目录                        |
| **C2** | ✅ **已答**      | ~~询问 CLI / pgDatabase / Multi Process 有无文档~~ → CLI 有完整命令表；`PGEP` 确认存在；**Multi Process 无独立文档**，转由 A7 上机确认                                                                                                           | —                       | —                                     |
| **C3** | ⬜             | 确认 **license 形态**：几把 key？AMC 状态？是否有 client-server 版？**是否允许同机多实例**                                                                                                                                                  | 采购 / IT                 | 书面答复                                  |
| **C4** | ⬜ **优先级上升**   | 确认 **Map 引擎（MapX / MapXtreme）授权** —— ⚠️ 现已知 **Area Binning 也依赖它**，影响面比原估大                                                                                                                                          | 同上                      | yes/no。若 no → Map 与 Binning 类任务从分类学删除 |
| **C5** | ⬜ 🔴 **重要性上升** | 约 **5–10 位工程师**做 motivating study（W03–W04 执行，现在只约时间）。<br>⚠️ **记录表必须含「单 case 时间按活动分解」**（在 XCAP 里操作 / 等待重处理 / 查文档 / 讨论 / 写报告）—— C7 完成后发现，**公开数据能建立成本包络，但无法把成本归因到 GUI 工具**，那是 §4 立论的前提，只能靠专家研究拿到 | 内部                      | 日历上有 3 个以上确认 + 记录表定稿                 |
| **C6** | ⬜ 🔴          | 查 **SIGCOMM '27 CFP**：deadline、页数、匿名要求                                                                                                                                                                             | —                       | 回填四份计划的 `due`                         |
| **C7** | ✅ **已完成**     | ~~收集 RAN troubleshooting 成本数字~~ → **四组数字到手**，见 [[PAPER-STRUCTURE-AND-STORYLINE]] §5.5：China Unicom 2024（€18M / 158,400 person-days / 30–60min per case / ≤12 cases/day / 知识编码 ≥2 个月）、China Unicom+Nokia SWAT 2022（€24M / 251,000 working-days）、Ericsson OPEX 拆分、TM Forum 2025 CSP 调研 | — | — |

> 🔄 **C1/C2 已大幅减负**，省下的时间应投到 **C6/C7** —— 这两项是 Intro 的地基，一直没人做。

> **C7 别拖。** [[PAPER-STRUCTURE-AND-STORYLINE]] 里说了：没有这个数字，Intro 的动机是悬空的。

---

## 3. Track B — 离线（可立即开工，不等 XCAP）

### B1 ⭐ 验证开源 diag parser 通路（U3）— 本周最高回报 🟡 **前置已完成**

**✅ 已完成（桌面）**：`sample data.md` 里 `Detail` 末尾的 hex 已解码验证为标准 Qualcomm diag packet：
`[外层 15B][inner_len 2B][log_code 2B][QC 64-bit ts 8B][payload]`，实测 msg1 = `0xB064`、msg2 = `0xB087`，与 `Message` 列一致。
**⬜ 剩余**：B1.1–B1.5 全部未做（parser 未写、对账未做）。**可行性已确立，现在是纯工程。**

| 步骤 | 内容 |
|---|---|
| B1.1 | 写 CSV parser。⚠️ **表头 8 列但数据行 10 字段**（Message 后两个未命名空列）；Detail 内含换行与逗号 → 必须按引号正确切分，不能按列数 |
| B1.2 | 从 Detail 抽出尾部 hex block，还原为二进制 |
| B1.3 | 接 **MobileInsight / SCAT / QCSuper** 解析 |
| B1.4 | **逐字段对账**：parser 输出 vs XCAP 的 Detail 文本。至少覆盖 3 个 log code（建议 0xB064 MAC UL TB、0xB087 RLC DL stats、0xB0A4 PDCP DL stats） |
| B1.5 | 记录不一致项与覆盖率 |

- **工时**：1.5 天（1 人）
- **DoD**：`B1_diag_parser_validation.md` —— 含覆盖率、逐字段对账表、不一致清单
- **产物**：`tools/xcap_csv_parser.py`、`tools/diag_extract.py`
- **判定**：
  - ✅ 对得上 → **离线 GT 管线当场定案**，建在开源 parser 上，独立于 XCAP，同时消解 T3
  - ⚠️ 部分对上 → 记录哪些 log code 不行，评估是否够用
  - ❌ 对不上 → 退回自写 Detail 文本 parser（无底洞，需重新评估 GT 策略）

### B2 ⭐ 时钟问题定案（U2）— 阻塞项 🟡 **先验已很强**

> ✅ **新增证据（桌面）**：`Chapter 4_Message Duration Time Analysis.pdf` p2 官方原文 ——
> *"When logged with Chipset Time, it displays Time Difference value using **Chipset Time** between two Messages."*
> ~~**XCAP 自己算消息间时延用的就是 Chipset Time。** 假设可先按 `canonical_clock = chipset` 推进~~
> ✅ **B2 已完成，规格已改为 `canonical_clock = TIME_STAMP`**（chipset 降为 `verification_clock`）。
> 三条理由：chipset 列只存在于 9% 的表；它在 app 层比无线层差 **5 小时**；XCAP 自己按 `TIME_STAMP` 分桶。详见 [[TASK-GT-GENERATION-PLAN]] §2.1。

**已知**（来自 sample，3 条消息）：

| # | `Time` | `Chipset Timestamp` | 偏差 | Δ`Time` | Δ`Chipset` |
|---|---|---|---|---|---|
| 1 | 17:12:43.557462 | 17:13:01.162808 | +17.605346 s | — | — |
| 2 | .557467 | .164583 | +17.607116 s | **5 µs** | 1775 µs |
| 3 | .557473 | .164608 | +17.607135 s | **6 µs** | 25 µs |

**待检验假设**：`Time` 是 host/解析侧的单调排序键，`Chipset Timestamp` 才是设备真钟。

| 步骤 | 内容 |
|---|---|
| B2.1 | 在**整条 trace**（不是 3 条）上统计两列的间隔分布。若 `Time` 间隔恒定在 µs 量级 → 排序键假设成立 |
| B2.2 | 画偏差随时间的走势 —— 是线性漂移、阶跃、还是随机？ |
| B2.3 | 拟合换算模型（**不能用常数**） |
| B2.4 | 确认 app 层指标（rebuffer / 码率 / 控制 RTT）用的是**第几个时钟** |
| B2.5 | 用 QC 64-bit 原始时间戳（B1 已能解出）交叉验证 `Chipset Timestamp` 列 |
| B2.6 | **选定规范时钟**，写进 `analysis_config.canonical_clock` |

- **工时**：1 天（1 人，B1.2 完成后可并行）
- **DoD**：`B2_clock_resolution.md` —— 含分布图、漂移走势图、换算模型、**规范时钟决定**
- **⚠️ 若此项未定案，W02 起所有时间相关任务停摆**

### B3' ⭐ 任务阶梯落地 🟡 **设计已定稿，只剩写样例**

> ✅ **已完成（桌面）**：阶梯设计定稿于 [[TASK-GT-GENERATION-PLAN]] §0.5 —— **L1 检索 → L2 计算 → L3 配置 → L4 对比 → L5 调查(RCA)**，含 oracle 类型划分与"为什么 RCA 是顶端而非全部"的论证。
> ⬜ **剩余**：B3'.1–B3'.3，即写 15 个具体样例。

关键性质：**L1–L4 有精确 oracle，不受 T1/T2/T3 影响，因此可以立刻开始生成，不必等 S0.5 共现分析。**

| 步骤 | 内容 |
|---|---|
| B3'.1 | 每级各写 **3 个具体任务样例**（自然语言 + 期望答案 + oracle 类型） |
| B3'.2 | 标注每级的 oracle 来源：L1/L3 状态谓词、L2/L4 离线计算或导出比对、L5 rubric |
| B3'.3 | 对 L2 特别标注 **`analysis_config` 依赖**（T5 在 L2 咬得最狠） |

- **工时**：0.5 天 · **DoD**：`task_ladder_v0.md`，15 个样例任务
- **为什么这周就做**：它决定 W02 的任务生成器先写哪一级。**答案是先写 L1–L4** —— 精确、可自动生成、不阻塞于 GT 方法学。

### B3 现象清单（只列表，不写代码）

按层列全"值得扫描的现象"，为 W02 的 detector battery 做准备。参考 [[TASK-GT-GENERATION-PLAN]] S0.4 的七层表。

- 每条现象标注：定义、所需字段、是否 XCAP 有内建分析功能可交叉校验、**语义类型（counter / gauge / event）**
- ⚠️ **T7**：`num_rlf_issued : 279` 是累计计数器不是事件，真正的事件是相邻采样差分。**语义类型这一列不能省。**
- **工时**：0.5 天 · **DoD**：`S0.4_phenomenon_taxonomy.md` v0，每层 ≥3 条

### B4 数据盘点

对已有 streaming / teleop 数据建索引：workload、时长、设备、场景、是否有 app 层指标、是否有 transport 层、**是否有注入故障标签**。

- **工时**：0.5 天 · **DoD**：`S0.1_data_index.csv`
- **🔴 顺带回答一个悬而未决的问题**：synthetic 是 **(a) 受控故障注入实验（有层标签）** 还是 **(b) 真实 trace 上程序化生成任务（无层标签）**？这决定 RQ6 的层混淆矩阵能不能做。

### B5 建 repo 骨架

- `softwarex-agent/`（主体是 agent，不要叫 bench）
- **第一个 commit 就要有 RPC 边界**：`controller/`（联网侧）· `worker/`（隔离机侧）· 二者之间是 socket/gRPC，不是函数调用
- `tools/` 放 B1 的产物
- **工时**：0.5 天 · **DoD**：CI 能跑通一个 hello-world 的 controller↔worker 往返

---

## 4. Track A — XCAP 机器（串行，按此顺序）

> ⚠️ **全程录制**。从第一次双击图标开始就开录（截图 + UIA dump + 动作），哪怕录制器还很粗糙。这是 replay 语料的第一批，也是 M1 的原料。**这周浪费掉的 live 时间补不回来。**

| ID | 任务 | 工时 | DoD |
|---|---|---|---|
| **A1** | 装好 XCAP + key，冷启动跑通。记录启动耗时。<br>⭐ **顺手两件事**：① **翻安装目录找文档文件夹** —— KB 里缺的 `Usage of Favorite Info` / `Multi Process` / `pgDatabase Usage` / `Selective Parsing` / `Filtering` 很可能随软件安装（User Guide 反复说 *"that comes with it"*）② **跑 `XCAP-M.exe -?`** 拿该版本权威命令全集 | 0.5d | 能开能关 + 文档清单 + `-?` 输出 |
| **A2** ⭐ | **建 Model**：拿 1 条 streaming + 1 条 teleop 数据。记录走哪条导入路径（XCAL 原生 / Ethereal-pcap / CSV / txt / TEMS / Converter）**及导入后保留哪些层** | 1d | `S0.1_data_layer_inventory.md`：层 × 字段 × 可见 × 可导出 |
| **A3** | 逐层清点：L1 / L2 / L3(RRC,NAS) / TCP-IP 各能看到什么。**特别确认 `Synchro Viewer (TCP Flow)` 能否激活**（需在 Selective Parsing 勾选 Modem/Network adapter 数据）—— 这是 transport 层视图，跨层任务的关键 | 0.5d | 同上 |
| **A4** 🔴 | **采样语义标定（T5）—— 范围已大幅扩大**：<br>① 采样率**不是 2 档而是 11 档**（`All/10/100/200/500/1000/2000/3000/4000/5000/10000ms`，不设=1s）—— 至少测 `All`/`1s`/`100ms` 三档<br>② 确认事件/信令确实不被采样<br>③ 🔴 **逐参数验证"静默覆盖"** —— Area Binning 对 **Sector / PSC 类参数无视所选公式强制用 Mode**。**不能假设选了 Average 就是 Average**<br>④ 分箱 9 种公式（Average/Medium/Median/Max/Min/First/Last/Mode）的实际行为与显示位置<br>⑤ 与官方 KPI 文档口径交叉验证（LTE PDSCH Throughput / RLC DL Dropped PDUs） | **1d**（原 0.5d） | `S0.1_aggregation_semantics.md` ← **N2 的实证基础** |
| **A5** ⭐ | **UIA 探针（U4）**：dump 控件树。**必须单独测拖放** —— 参数树 → 工作区、Alt+拖放叠加。这是 XCAP 的主要交互方式，也是 GUI agent 的硬骨头 | 0.5d | `E0.1_uia_report.md`：树是否有语义节点？拖放能否程序化触发？ |
| **A6** ⭐ | **Hatch 实测（U6）** —— ⚠️ **CLI 已有完整文档**（见 [[XCAP-KB-FINDINGS]] §1.1），本项从"探索"改为"验证"：<br>① **先跑 `XCAP-M.exe -?`** 拿该版本权威命令全集（10 分钟，最高性价比）<br>② 验 `-CM ... -FEP ....fav -MG <SamplingRate>` 端到端<br>③ 🔴 **`PGEP` + PostgreSQL 服务器配置** —— 通了则离线 GT 管线建在库上。⭐ **步骤已文档化**（§5.8.5）：装 **PG 10** → `pg_hba.conf` 设 `host all all all md5`（⚠️ 文档把文件名拼成 `pg_hpa.conf`）→ 防火墙放行 TCP **5432** → XCAP 内 `File-New-Options-pgDatabase` 填连接串。<br>&nbsp;&nbsp;&nbsp;⚠️ 建模时**必须勾** `Keep the created model in the Model Manager`（默认不勾 = 上传后丢弃 Model），且该项**仅单进程有效**；`[DB Option]` 默认只建 4 类表，需按数据的 chipset/RAT 勾选。<br>&nbsp;&nbsp;&nbsp;🔴 **绝不要勾** `UpdateTime is Null → delete` —— 一次性、不可逆、会删库<br>④ `.fav` / `.mes` / `.oxf` / `.wrk` 是文本还是二进制、能否程序化生成（⭐ **手册只给了语义没给格式，这条现在是 M6 唯一的未知**）<br>④' ⭐ **实测 Favorite 跨 Model 迁移**：在 Model A 上建一个 Favorite，切到 Model B 直接打开 —— 官方称可迁移（§5.7.1），这是 M6"技能"定义的地基，必须亲眼验证；顺带测 **Recursive** 勾选后的行为<br>⑤ 实测评论区反馈的"文件夹路径批量导出报错"是否属实 | 0.5d | `S0.2_hatch_matrix.md` |
| **A7** ✅→🔄 | ~~**`Multi Process` 到底是什么（U5）** —— 能否多实例并行？~~ → **文档已答：不是多实例，是单实例内把 log 切 2 组并行解析后合并。capacity=1 不变，U5 关闭。**<br>🔄 **改为验证 T8a（0.25d）**：① 记录 `tools > options > General > Common` 里该开关的**默认值** ② 开/关各建一次 Model，**对比 Model 目录结构**（是否出现 sub-model 子目录）③ 确认混合格式（`drm+scn`）与 `PAF` 会否静默降级 | 0.25d | ~~yes/no~~ → `E0.6_multiprocess_effect.md`：默认值 + 两种目录结构截图。**离线管线的目录假设据此定稿** |
| **A7'** 🔴 | **建模选项决定观测面（T8b，M4 的核心素材）** —— 同一条 log 建两次 Model：① 勾 **Log viewer**（跳过 Message viewer）② 默认全量。测 **两者建模耗时差** 与 **消息级查询在 ① 下是否真的不可得**，再测 **Reparsing 的代价**。<br>这三个数字直接支撑"可逆但代价高、且代价在决策时不可见"的论证，也是 E3 诊断任务对的定价依据 | 0.25d | `E0.7_viewer_mode_cost.md`：`t_build(log_only)` / `t_build(full)` / `t_reparse` 三个数 + 观测面差异清单 |
| **A8** | **三级检查点代价实测（M4 的物理基础）** —— ⭐ **CLI 原语已确认存在**：`INI`(启动) / `KX`(强杀) / `MOP <model>`(重开 Model) / `MDEL`(删) → **三级可完全脚本化,不必依赖 GUI**。<br>测：① 清 filter + `All Window Close` + 重载 `.wrk` ② `KX` → `INI` → `MOP` ③ 从 log 重新 `-CM`。**各 3 次取均值** | 0.5d | `E0.4_checkpoint_cost.md` |
| **A9** | 输入注入稳定性：连续 200 次动作（**含拖放**），统计静默失败率、焦点丢失率 | 0.25d | `E0.5_input_reliability.md` |
| **A10** | 隔离机网络拓扑（U-infra）：能否出网？LLM 调用放哪侧？ | 0.25d | 确认 B5 的 RPC 边界设计可行 |
| **A11** | L0 静态收割原型：脚本遍历 7 个分析 tab + Message / Signaling Flow / Synchro Viewer，逐个 dump 状态 | 0.5d | 能跑完不崩，产出第一份 UI map |

**Track A 合计 ≈ 5.75 人日**（A7 减 0，新增 A7' +0.25 → 6.0） → 一个人干满一周刚好，两个人可以把 A5/A6 分开做。

---

## 5. 逐日建议

| 日 | Track C | Track A（XCAP 机器） | Track B（离线） |
|---|---|---|---|
| **一 8/31** | **C1–C7 全部发出（上午）** | A1 装机冷启动 · A10 网络拓扑 | B5 repo 骨架 · B1.1 CSV parser |
| **二 9/1** | 跟催 C3/C4 | **A2 建 Model（重头戏）** | B1.2–B1.4 hex → diag parser 对账 |
| **三 9/2** | | A3 逐层清点 · **A4 采样标定** | **B2 时钟定案** · B4 数据盘点 |
| **四 9/3** | | **A5 UIA 探针** · **A6 hatch 实测** | B1.5 收尾 · B3 现象清单 |
| **五 9/4** | | A7 Multi Process · A8 检查点 · A9 输入 · A11 L0 原型 | 汇总，写 Gate G0 一页纸 |

---

## 6. 周五 Gate G0 决策会（1 小时）

### 议程

1. **六个 yes/no 逐个过**（U1–U6，其中 **U5 已由文档关闭**），每个一句话
2. **三个分支决策**：

| 条件 | 决策 |
|---|---|
| U1 = 否（XCAP 吃不了我们的数据，且转换后丢关键层） | **换锚点软件**，Wireshark 升主，XCAP 降为小规模泛化验证。整个 §1 motivation 重写 |
| U4 = 否 **且** U6 = 否（UI 不可读 **且** 无程序化导出） | **换锚点软件**。两者只要有一个为真就继续 |
| ~~U5 = 是（Multi Process 可并行）~~ | ~~**好消息** —— 重算吞吐预算，W02 起时间线可放宽~~ → **已排除**：文档确认为单实例内并行解析。**不必再等这个分支，按 capacity=1 规划** |

3. **W02 计划确认**（依 U2/U3 的结果决定 detector battery 怎么写）

### 交付物

`W01_GateG0_summary.md` —— 一页纸，六个答案 + 三个决策 + W02 是否照原计划。

---

## 7. 本周明确不做

防止范围蔓延，以下全部推到 W02+：

- ❌ 写 detector（B3 只列表，不写代码）
- ❌ 状态序列化器（要等 A5 的结论才知道怎么设计）
- ❌ 任何 agent 代码
- ❌ 任务生成器
- ❌ Wireshark 环境接入
- ❌ 专家研究的执行（W01 只约时间）

---

## Actions

### Track C（周一上午）
- [x] ~~C1 索取随附分册 PDF~~ → **17 份已下载至 `xcap_docs/`**
- [x] ~~C1' 索取 KB 缺失的 5 份~~ → **4/5 已到手**（从 `Analyzer Help.chm` 抽出，见 `xcap_docs/*.md`）：`Usage of Favorite Info` / `Multi Process` / `Selective Parsing` / `Filtering`。精读结论见 [[XCAP-KB-FINDINGS]] §5.7
- [x] ~~C1'' 只剩 `pgDatabase Usage` 一份~~ → ✅ **2026-09-02 到手**（13 页 PDF）。**关键路径上已无缺失文档。** 结论见 [[XCAP-KB-FINDINGS]] §5.8
- [x] ~~C2 询问 CLI / pgDatabase / Multi Process 文档~~ → CLI 命令表已获取；`PGEP` 确认存在；Multi Process 转 A7
- [x] C3 确认 license 形态与多实例许可(单hardware密钥 单实例)
- [ ] C4 确认 Map 引擎授权（⚠️ Area Binning 也依赖，影响面变大）
- [ ] 🔴 C5 约 5–10 位工程师（W03–W04 执行）+ **记录表必须含「单 case 时间按活动分解」**（公开数据唯一填不了的缺口）
- [x] 🔴 C6 查 SIGCOMM '27 CFP，回填 `due`
- [x] ~~C7 收集 RAN troubleshooting 成本数字~~ → **四组已到手**，见 [[PAPER-STRUCTURE-AND-STORYLINE]] §5.5

### Track B（离线）
- [x] ~~B1.0 验证 hex 是否为标准 QC diag packet~~ → **已解码确认**（`0xB064`/`0xB087` 与 Message 列一致）
- [ ] B1.1 CSV parser（注意表头 8 列 / 数据 10 字段错位）
- [ ] B1.2 抽 hex block
- [ ] B1.3 接 MobileInsight / SCAT / QCSuper
- [ ] B1.4 逐字段对账（≥3 个 log code）
- [ ] B1.5 覆盖率与不一致清单 → `B1_diag_parser_validation.md`
- [x] ~~B2.0 时钟问题的先验判断~~ → **官方文档证实 XCAP 用 Chipset Time 算时延**
- [ ] B2 时钟定案（整条 trace 统计 + 漂移模型 + 定 `canonical_clock`）→ `B2_clock_resolution.md`（**阻塞项**）
- [x] ~~B3'.0 任务阶梯设计~~ → **已定稿于 [[TASK-GT-GENERATION-PLAN]] §0.5**
- [ ] B3' 写 15 个样例（L1–L5 各 3 个）→ `task_ladder_v0.md`
- [ ] B6 取回 XCAP 线全部 Release Note（KB 内，7+ 篇已定位），按漂移类别编码 → `drift_timeline_v0.md`
- [ ] B3 现象清单 v0（**必须含 counter/gauge/event 语义列** —— T7 已被官方证实）
- [ ] B4 数据盘点 + **确认 synthetic 是 (a) 还是 (b)**
- [ ] B5 repo 骨架（第一个 commit 就有 RPC 边界）

### Track A（XCAP 机器，串行）
- [ ] A1 装机冷启动
- [ ] A2 建 Model（streaming + teleop）← 重头戏
- [ ] A3 逐层清点 + `Synchro Viewer (TCP Flow)` 确认
- [ ] A4 采样语义标定（`All` vs `1sec`）
- [ ] A5 UIA 探针（**含拖放**）
- [ ] A6 hatch 实测（CLI / pgDatabase / oxf·wrk / 报告）
- [ ] A7 🔄 Multi Process 开关对 Model 目录结构的影响（T8a；U5 已由文档关闭）
- [ ] A7' 🔴 建模选项 → 观测面（T8b）：log_only vs full 的建模耗时 + Reparsing 代价
- [ ] A8 三级检查点代价
- [ ] A9 输入注入稳定性
- [ ] A10 隔离机网络拓扑
- [ ] A11 L0 静态收割原型
- [ ] **全程录制**

### 周五
- [ ] Gate G0 决策会
- [ ] `W01_GateG0_summary.md`

---

## Notes

- **本周最高回报的两项**：**B1**（开源 diag parser 通路 —— 通了则离线 GT 管线当场定案，且独立于 XCAP，审稿人更认）与 **B2**（时钟定案 —— 不定案则 W02 起所有时间相关任务停摆）。两项的**可行性都已由桌面研究确立，现在是纯执行**。
- **最容易被跳过、但代价最大的**：**全程录制**。这周的 live 时间浪费掉补不回来。
- **Track C 必须周一上午发出** —— 全是有等待周期的事。
- A2 若卡住，**不要死磕**。先跑 A5/A6（它们只需要任意一个能打开的 Model，XCAP 自带 sample 也行），把 U4/U6 先答掉。

### 🔄 桌面研究后的重新配平

**已减负**：C1/C2（17 份文档到手）· A6（CLI 从探索→验证，1d→0.5d）· A8（检查点从设计→测耗时）· B1/B2（可行性已立）· B3'（只剩写样例）

**这些时间应转投到**：

| 转投目标 | 理由 |
|---|---|
| ~~🔴 C6 / C7~~ | ✅ **两项均已完成**（C7 见 [[PAPER-STRUCTURE-AND-STORYLINE]] §5.5）→ 时间转投下一行 |
| 🔴 **C5 专家研究设计** | C7 完成后暴露出一个结构性缺口：**公开数据能建立成本包络，但无法把成本归因到 GUI 工具**。Intro 那句 *"almost all of that time is spent driving a proprietary GUI analyzer"* 的后半句只能靠专家研究拿到。→ **记录表必须按活动分解单 case 时间** |
| 🔴 **A4 采样标定（范围已扩大）** | 从"All vs 1sec 两档"变成 **11 档采样率 + 9 种分箱公式 + 逐参数验证静默覆盖**。这是 N2 的实证基础，**工时应从 0.5d 提到 1d** |
| **A2 数据兼容性** | U1 出现了新的硬约束（CLI 不支持 pcap/CSV），要多花时间确认 app 层指标的导入路径 |
| **A1 翻安装目录** | 可能一次性解决 C1' 的 5 份缺失文档 |

**新增（离线，可立即做）**：B6 Release Note 漂移编码 —— 这项在 W01 之前不存在，是 KB 挖掘的产物。
