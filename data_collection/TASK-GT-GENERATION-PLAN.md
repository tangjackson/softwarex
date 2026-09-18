---
title: "任务与 Ground Truth 生成计划"
type: project
created: 2026-08-28
updated: 2026-08-28
status: active
area: "[[Research]]"
priority: P1
tags: [project, research, sigcomm, benchmark, ground-truth, xcap]
---

# 任务与 Ground Truth 生成计划（v1 — 已据 User Guide 5.33 修订）

> 母计划：[[PLAN-SoftwareX-Sandbox-Benchmark]]
> 依据：`XCAP User Guide 5.33.xx (rev9).pdf`（Accuver，2025.01，55 页）
>
> 本文档只把**前期确认与准备工作**写实，规模化生成待 Stage 0 结果回来后细化。

---

## 0. 手册带来的三个结论（先看这个）

### 结论 A ⭐⭐⭐ —— XCAP 的"escape hatch"极其丰富，母计划 E0.2 基本已被预先回答

手册 Chapter 4 (Analysis Method) 与 Chapter 5 (Tools) 里列出：

| 能力 | 出处 | 意义 |
|---|---|---|
| **File Export using Command Prompt** | Ch.4 Analysis Method | 🔴 **命令行导出存在** —— 这就是 XCAP 的 `cat` |
| **pgDatabase Usage** | Ch.4 Analysis Method | 🔴 **PostgreSQL 后端** —— 可能可直接 SQL 读解析后的数据 |
| **Batch Export Feature** | Ch.4 | 批量导出 |
| **Scheduler**（Model Creation Plan） | Ch.4 / Tools–Misc | 定时从指定目录批量建模，**不需要驱动 GUI** |
| **Multi Process** | Ch.4 | 可能突破"capacity=1"的悲观假设 |
| **User Defined Report / KPI Parameter** | Ch.4 / Tools–Report | 可编程的抽取与指标定义 |
| **Export/Import Option → `*.oxf`** | Tools–Setting | 导出 Filter List / Favorite / Correlation / Auto Report / Legend / Options，**且可再导入** |
| **Workspace Export/Import → `*.wrk`** | Workspace List Manager | 工作区可序列化 |
| Report 输出 pdf / excel / doc / html | Tools–Report | 导出 oracle |
| Data export → Excel / PPT / CSV / Txt | Ch.1 Features | 同上 |

> **行动改变**：Stage 0 的重点从"有没有 escape hatch"变成"**这些 hatch 各自能拿到什么、精度如何**"。其中 **pgDatabase 与 command-prompt export 优先级最高** —— 如果 pgDatabase 可读，整条离线 GT 管线可以直接建在数据库上，成本骤降。

### 结论 B ⭐⭐⭐ —— XCAP 自带"技能"持久化格式，母计划的 M6 要改设计

`Template Manager` / `Favorite Manager` / `Workspace (.wrk)` / `Options (.oxf)` 四者合起来，等于软件**原生提供了一套可复用分析配置的序列化格式**。

> 这意味着 agent 学到的 skill **不必是"回放一串点击"，而可以编译成 XCAP 自己的 `.wrk` + `.oxf` + Template**。
>
> 类比：coding agent 学会一个流程后，不是记住一串 shell 命令，而是**写一个 Makefile**。

这比动作回放稳健得多（对布局/分辨率漂移天然免疫），而且是一个很漂亮的设计主张：**agent 写入宿主软件自己的持久化格式**。已回写母计划 M6。

### 结论 C ⭐⭐ —— XCAP ↔ Wireshark 存在双向数据桥

- **Import**：支持 `CSV, txt, Tektronix, Ethereal(=Wireshark), TEMS`
- **Converter**：`Convert to PCAP (Ethernet PPP)` —— *"Extract PCAP data from Log data. (If analyze in Wireshark, use the corresponding Converter.)"*
- 另有 `Convert to DLF (Qc format)`、`Convert to DAT (Anritsu)`、`ISTP`、`IntegrationConverter`

两个后果：

1. **E0.0 风险大幅下降**：pcap 可导入，CSV/txt 可导入（app 层指标有路径进来）。
2. ⭐ **解锁一个很强的对照实验**：**同一份数据、同一个问题，在 XCAP 与 Wireshark 两个界面里各做一遍**。这能把"软件操作能力"与"领域推理能力"**干净地分离开** —— 领域推理相同，差异全部来自 UI。这是一个别人做不了的消融，已回写母计划。

---

### 结论 D ⭐⭐⭐ —— 导出样本实测：离线 GT 管线有一条**无损、免 license** 的现成通路

依据 `sample data.md`（Logging Message 导出，CSV）。

**格式**：`Index, Time, Lon, Lat, Chipset Timestamp, Code, Message, Detail`
⚠️ 实测**表头 8 列、数据行 10 字段**（Message 后有两个未命名空列）—— parser 不能按表头列数切分。

**🔴 最重要的发现：`Detail` 末尾的 hex dump 是标准 Qualcomm diag log packet，可无损还原。**

实测解码（`[外层 15B][inner_len 2B][log_code 2B][QC 64-bit ts 8B][payload]`）：

| 消息 | outer_len | inner_len | log_code | 与 Message 列一致？ |
|---|---|---|---|---|
| msg1 | 55 | 40 | **0xB064** | ✅ LTE MAC UL transport block |
| msg2 | 399 | 384 | **0xB087** | ✅ LTE RLC DL statistics |

> **推论**：`导出 CSV → 抽 hex → 用开源 Qualcomm diag parser（MobileInsight / SCAT / QCSuper）解析` 构成一条
> **完全不依赖 XCAP、可并行、免 license、且无损**的离线 GT 管线。
>
> 这比自己写 Detail 文本 parser 好得多（Detail 是每个 log code 各不相同的 ASCII 艺术表格），也让 GT 具备了**独立于被测软件**的第三方权威性 —— 审稿人会喜欢这一点。
>
> 同时它天然解决 T3：GT 管线看到的是**原始码流**，不再比 agent 少。

**⭐ 另一面：`Detail` 是纯文本，且信息极密**（key:value + 对齐表格 + 比特级解码 `3E:0/0/1/11110 = R/R/E/LCID [Long BSR]` + hex）。
→ **这就是 XCAP 的 `cat`**，而且对 LLM 极友好。Message 窗口的 Detail 面板应作为 substrate 的**最高信息密度观测面**。

---

### 🔴🔴 T6（新）— 两个时钟，偏差 ~17.6 秒且在漂移

实测同一条 CSV 的两个时间列：

| # | `Time` | `Chipset Timestamp` | 偏差 | Δ`Time` | Δ`Chipset` |
|---|---|---|---|---|---|
| 1 | 17:12:43.557462 | 17:13:01.162808 | **+17.605346 s** | — | — |
| 2 | 17:12:43.557467 | 17:13:01.164583 | **+17.607116 s** | **5 µs** | 1775 µs |
| 3 | 17:12:43.557473 | 17:13:01.164608 | **+17.607135 s** | **6 µs** | 25 µs |

两个问题：

1. **偏差不是常数**（17.605346 → 17.607116 → 17.607135），说明两个时钟在相对漂移，不能用单一常数校正。
2. ⚠️ **`Time` 列的相邻间隔只有 5–6 µs，不像真实消息间隔**；而 `Chipset` 间隔（1775 µs / 25 µs）是合理的。
   **强假设：`Time` 是 host/解析侧赋予的单调排序键，`Chipset Timestamp` 才是设备真实时钟。** 必须在 S0.1 证实或推翻。

> **影响**：`"t=128.42 发生了什么"`、任何时间窗聚合、任何跨层证据对齐，**答案取决于用哪个时钟**。
> 若 app 层指标（rebuffer / 控制 RTT）还带第三个时钟（PC/服务器），就是**三时钟问题**。
>
> **这必须在做任何时间相关任务之前解决**，否则整个跨层归因的头条实验立不住。已提升为最高风险之一。

---

### T7（新）— 计数器 vs 事件

样本中 `num_rlf_issued : 279`、`last_rlf : 269931339` 等是**累计计数器**，不是事件。
> 在 t 读到 279 ≠ "此刻发生了 279 次 RLF"。真正的事件是**相邻采样的差分**。
> detector 与 GT 必须区分 counter / gauge / event 三类语义，否则会把累计量当瞬时量。这是 T5 的近亲。

---

### T8（新）— 解析路径非确定性 + 观测面在**建模时**被冻结

来源：安装目录分册 `Multi Process` / `Selective Parsing`（详见 [[XCAP-KB-FINDINGS]] §5.7.3–5.7.4）。

**T8a — Model 的物理结构随一个全局开关而变。** Multi Process 开启时，XCAP 把 log 切成 2 组分别解析，产出 **sub-model 子目录**再合并；关闭时是单一 Model。任何直读 Model 目录的离线管线都会因此崩掉。更糟的是它会**静默降级**：混合格式（`drm+scn`、`drm+drmw`）、acu/ecu 日志、或选了 `PAF`，都会自动退回单进程，不通知。且解析超时会被**判定为 Failure 并继续下一个 job** —— 无人值守批处理里会出现静默缺失的结果。

**T8b — 观测面在建模时就被冻结了。** 为了提速而在建模前选 Log viewer（跳过 Message viewer），会让**后续所有消息级问题不可回答**，除非 Reparsing（分钟级）。也就是说：

> **不是"不可逆"，而是"可逆但代价高，且代价在做决定的那一刻不可见"。**

这对 GT 的直接含义：**同一份 log 的"正确答案"取决于建模选项**，与 T5 同构但更隐蔽 —— T5 至少在分析窗口里能看到配置，T8b 的损失只在你去查证据时才显形。

**对策**：`model` 段必须钉住三个新字段（见 §2.1），且 GT 生成时**必须记录建模选项**，任务发布时把它作为环境的一部分固定下来，不允许 agent 自选（除非该任务本身就是考察建模决策——见下）。

> 💡 **T8b 同时是一个任务设计的机会而非纯威胁**：同一份 log、同一个症状，出两条任务——一条要求 L3 消息级证据，一条不要求。观察 agent 是否在两种情况下做出**不同的建模选择**。这是对"前瞻性决策"能力的干净测量，建议列入 E3。

> ⚠️ **不推翻 capacity=1**：Multi Process 是单实例内部的并行解析，许可证瓶颈不变。
> ⚠️ ~~实测更正：实例上限 = 2（写死）~~ 🔴 **09-09 推翻，待封口**，见 [[weekly_updates/W02]] §7。**规划按 capacity = 1。**

**T8c（09-03 实测新增）— 用 CLI 还是 GUI 建模，决定观测面**

| 产物 | GUI 全流程 | CLI `-CM` |
|---|---:|---:|
| `MSG_FILTER_L*.dat` | 337 张 | **2 张** |
| `xCOMMON_BMF_*.dat` | 86/78 张 | **0** |
| **`xCOMMON_L3_Failure_Reason`** | **132 条** | 🔴 **不存在** |

⇒ **比 `viewer_mode` 那条更隐蔽**：`viewer_mode` 至少是个可见的勾选框，
而"用 CLI 还是 GUI 建模"**看起来完全是个效率选择**，没有任何提示说它会砍掉一整类产物。
⇒ `model` 段必须增字段 **`build_path: gui | cli`**。

---

### T9（新，09-03 实测）— "层"不是被测软件的一等公民

pgDatabase 的 308 张表按 **RAT × 功能块**组织：LTE 141 / 5G-NR 46 …；
KPI 134 / RF 77 / Layer1 41 / Tput 30 / Signaling 19 / 🔴 **`Layer2` = 0**。

⇒ **XCAP 的轴里没有 MAC / RLC / PDCP / NAS。**
把表归到协议七层是**把一个轴投影到另一个轴，投影没有唯一解** ——
换一套正则，RF/PHY 从 **107 张变 435 张**。

**对 GT 的直接影响**：

| 字段 | 影响 |
|---|---|
| `causal_chain[].layer` | 🔴 **"层"必须显式定义并声明投影规则**，不能当作自明的 |
| `primary_attribution.layer` | 同上；**N1 混淆矩阵的行列标签需重新定义** |
| 论文表述 | **不得出现"某层多少表 / 多少记录"**，除非同时给出投影规则 |

✅ 稳健的部分：**"≥3 层非空"怎么投影都成立** ⇒ 跨层任务本身不受威胁。

⭐ **建议的解法**：`layer` 改为**由注入装置或协议事件定义**（RF 衰减器 → RF；netem → Transport；
RRC 信令 → RRC），**不由表名归类定义**。这样层标签来自物理事实，而非投影。

---

### T10（新，09-03 实测）— 跨层时间对齐的精度上限

| 量 | 值 |
|---|---|
| streaming 跨层对齐精度 | **± 1.5 s** |
| teleop（受手机钟 5.0 s 抖动限制） | **秒级** |
| streaming 应用层时钟 | 开机以来的秒（锚定后 `app_wall_time = monotonic + 1774208643.4`，191 会话极差 2.7 s） |
| teleop 应用层时钟 | Unix 毫秒 ⇒ **两批业务数据的应用层时间基准不同** |

🔴 **做不到帧级对齐** ⇒ **任何"这一帧对应哪个 slot / 哪次调度"的题面不成立**，必须从任务库排除。
`answer_scope` 的时间粒度上限 = **秒级**。



---

## 0.5 ⭐ 任务阶梯：为什么 RCA 是顶端而不是全部

### 为什么要重构任务分类

"为什么是 RCA，它和一般数据分析有什么区别？"——这个问题的答案改变了任务设计。

**本质区别在于任务描述的完备性**：

| | 一般数据分析 | RCA |
|---|---|---|
| 任务描述 | **完备**（告诉你算什么） | **不完备**（只说发生了什么，不说去看哪里） |
| 过程 | 固定，可查手册 | **在调查中被发现**，自适应 |
| 答案 | 唯一，由数据决定 | **多重，取决于解释层级** |
| 何时停止 | 算完就停 | **认为被说服了才停**（主观） |
| 验证 | 精确 | rubric + 证据 |

### 但纯 RCA 有三个代价

1. **本文档 T1/T2 的全部痛苦都来自 RCA。** 若任务是"算 t1–t2 平均吞吐"，GT 精确且免费。
2. **RCA 混淆两种能力**：失败可能是"不会操作软件"或"不懂协议栈"。而论文的贡献是**软件使用** —— 用混淆变量测目标变量。
3. 🔴 **结论 D 反过来削弱了 GUI 的必要性**：既然 hex 是无损 diag packet、开源 parser 能全解，审稿人必问

> **"信息都在原始码流里，为什么 agent 需要开 GUI？直接离线解析推理不就行了？"**

> **必须正面回答**：XCAP 的价值不在数据，在**沉淀其中的领域工程**（跨层时间对齐、KPI 计算、RB scheduling map、DCI 分析、neighbor list、sector swap 等内建分析）。离线重实现整套是没人会做的事。
> ⚠️ 但这是**实用论据，不是科学必然性论据**。**离线 GT 管线做得越好，"必须用 GUI"的论证越弱** —— 这个张力要在论文里主动处理，不能等 reviewer 提。
> **纪律**：离线管线只作为 **oracle**，绝不作为 agent 的替代解法路径。

### 任务阶梯（每级新增一个难度维度）

| L | 任务类 | 新增的难度维度 | 答案 | Oracle 类型 |
|---|---|---|---|---|
| **L1** | 检索定位 | 找到目标 | 唯一 | 精确（状态谓词） |
| **L2** | 计算测量 | **在给定配置下聚合** | 唯一（**给定配置**） | 精确 ← **T5 在这里咬人** |
| **L3** | 配置构建 | 产生指定视图/状态 | 唯一状态 | 状态谓词 |
| **L4** | 对比 | 多 trace / 多实体 / 跨层对齐 | 唯一 | 精确（导出文件） |
| **L5** | **调查（RCA）** | **假设生成 + 证据收集 + 何时停止** | **多重** | rubric + 可核验证据 |

### 这样重构的四个好处

1. **L1–L4 有精确 oracle** → 干净测量"软件使用能力"，零 GT 歧义，且可大规模自动生成
2. ⭐ **L1–L4 与 L5 的性能落差本身就是一个头条发现** —— 它把"会操作软件"与"懂这个领域"分离开，而这正是论文真正想测的
3. **去风险**：若 L5 的 GT 最终不可行，L1–L4 仍构成一篇论文
4. ⭐ **N2 不再依赖 RCA** —— T5（配置决定真值）恰恰在 **L2** 咬得最狠（"平均吞吐是多少"就是一般分析问题）。**安全垫独立于 RCA 成立**

> 📌 **好消息**：母计划原有的 T1–T6 任务类其实已经包含这个阶梯，只是没被当作阶梯用。
> 缺的是让**「oracle 类型」与「难度递进」**这两件事变成承重的。这是重构表述，不是推翻重来。

### 连带的定位调整

**LLM 做 RCA 正在变成红海**（云上 incident RCA 那条线已经很挤），而 **software-use 仍是空白**。

> 定位成"我们用 LLM 做 RCA" → 与一群跑得很快的人竞争，delta 很薄。
> 定位成"我们研究 agent 能否操作专业软件，以 RAN 诊断为领域" → delta 干净。

**因此：论文的研究对象是 professional software use，RCA 是最难的那一类任务实例，而不是反过来。** 这与母计划"agent 设计是第一贡献"一致。

**下游影响**：
- 本文档 §1 的 T1–T10 威胁**主要作用于 L5**；L1–L4 只受 T5/T6/T8 影响（配置、时钟、建模选项），处理成本低得多
- §5 评分 rubric 分两套：L1–L4 用精确匹配，L5 用 rubric
- **S0.4 detector battery 仍然必要**（L5 需要），但 L1–L4 的任务生成可以立刻开始，不必等 S0.5

---

### 0.5.1 ⭐ L5 拆为 L5a / L5b（09-02 定，09-05 由 D2/D3 门确认）

| 档 | 考什么 | Oracle | 需故障标签 | 门 |
|---|---|---|---|---|
| **L5a 证据链构造** | 跨层**找齐并组装**证据："t 前后各层发生了什么，按时间与层次组织" | **证据可核验性**（这条证据在数据里能不能被验证） | ❌ | **D2** 🟡 只差"稳定共现"定义 |
| **L5b 根因归因** | 在候选中**判定主因** | rubric + 可核验证据 | ✅ | **D3** 🟡 只差 T-13 专家试标 |

⭐ **L5a 考的正是 software-use 能力**（驱动软件把散落在多层、多视图、不同时钟基准下的证据找齐），
**不考领域推理**，因而**不需要故障标签，可从现有数据生成**。
它把"L5 做不了"从全有全无变成**分级降级**。

⭐ **一道现成的 L5b 样板题（09-03 实测产出）**：

> *"这次切换造成了多长的数据中断?"*

实测:切换 → 数据 gap 只有 **8 / 62 = 13%**，**87% 的切换在 RLC 吞吐上留不下任何痕迹**。
⇒ 这道**看起来最自然的跨层题，在 1Sec 档数据上无解**，
而 **agent 会算出"0 秒中断"并且很自信** —— 正是 L5 要抓的那类错误。
⚠️ 最可能的解释是 1Sec 采样档分辨不出百毫秒级 gap（**口径问题不是物理问题**），
需用 `All`/`100ms` 档重建 Model 证伪 → 这本身又是一道 T5 题。

---

## 1. GT 的效度威胁（T1–T10）

> ⚠️ **适用范围**：以下威胁**主要作用于 L5（RCA）**。L1–L4 只受 T5（配置）、T6（时钟）与 T8（建模选项）影响。

### T1 — 标签不完备（"implicit B"）

瞄准 A，trace 里同时存在未受控的 B。GT 写 A，agent 答 B —— agent 可能是对的。

> ⚠️ **最危险的一类，因为它与能力反相关。** 越强的 agent 看得越多、越容易发现 B，因而被判错得越多。benchmark 系统性惩罚更好的模型，且该偏差在总分上完全不可见。

### T2 — 因果链的多重性与层级

`rebuffer ← 吞吐骤降 ← SINR 恶化 ← 覆盖空洞`。每一环都"是"原因。agent 答"吞吐骤降"不是错，是**层级不对**。

### T3 — 可观测性不对称（XCAP 比你的脚本看得多）

XCAP 暴露 L1/L2/L3(RRC/NAS)/TCP-IP 全栈、PRB 调度图、DCI per-TTI、Beamforming SSB Index Array、Hex/Bit 级原始码流。离线 GT 管线只解析你写了 parser 的字段 → **GT 可能比 agent 掌握的信息更少**，颠倒 benchmark 前提。

### ⭐ T5（新）— 分析配置决定"真值"本身

这是读手册读出来的，而且我认为**比 T1 更隐蔽**。

手册里至少四处配置会**静默改变同一个问题的答案**：

| 配置 | 手册出处 | 危害 |
|---|---|---|
| 🔴 **Sampling Rate（默认 1 sec）** | Ch.3 建模第 8 步 | *"1 sec: each timestamp displays **average** log data collected for 1 second"*。**GUI 里读到的指标默认是 1 秒平均值，不是瞬时值。** 而 `All` 选项才显示原始时间戳。另注：*"Sampling Interval is not applied for Call Events and Signaling Messages"* —— 即**事件与信令不被采样，指标被采样**，两者时间语义不一致 |
| **Selective Parsing** | Ch.3 第 5 步 | 建模时可只解析部分参数 → **同一份 log 可以产出内容不同的两个 Model**。模型里没有的参数在树中显示为灰色 |
| **Filtering 是全局的** | Ch.1 Features | *"Apply filtered data to **all** parameter windows"* → 跨窗口状态污染。reset 必须清 filter |
| **Merge vs Separate / Area Binning** | Ch.3 第 4 步 / Map Bar | 多 log 合并成一个时间序 Model vs 各自独立；Bin 内取代表值 —— 都改变聚合语义 |

> **结论**：`"t 时刻的平均吞吐是多少"` 的真值不是原始数据的函数，而是
> **`f(原始数据, 解析配置, 采样配置, 滤波状态, 分箱配置, 时钟选择, 窗口尺寸)`**。

### 🔴 T5 的两个升级版（据分册精读，见 [[XCAP-KB-FINDINGS]] §5.5）

**(a) 配置空间比预想大得多**

| 维度 | 实测选项数 |
|---|---|
| Sampling Rate | **11 档**（All / 10ms / 100ms / 200ms / 500ms / 1000ms / 2000ms / 3000ms / 4000ms / 5000ms / 10000ms），**不设=1s** |
| Area Binning 的 `Sampling Function` | **9 种**（Average / Medium / Median / Max / Min / First / Last / Mode / Apply-to-All），且**显示位置也不同**（箱中心 vs 原位置） |
| Message Duration 的 `First/Last` | 重复消息取首条还是末条 |
| Message Duration 的呼叫区间 | `With Regard to Call`（默认）vs `Regardless of Call` |
| 时区 | CLI 有 `UTC` 开关 |
| 窗口尺寸 | 影响导出内容（*"export is processed according the displayed size"*） |
| 并列 tie-break | *"displayed at the **firstly founded** position"* |

**(b) 🔴🔴 存在"静默覆盖" —— 钉死配置也不够**

Area Binning 原文：

> *"If you run binning for **Sector and PSC Type** parameters, **only Mode value is displayed whichever formula you select**."*

> **对某些参数类型，工具会静默忽略你显式指定的聚合公式，且不提示。**
>
> → 真值不只取决于配置，还取决于 **参数类型 × 配置** 的交互。
> → **`analysis_config` 钉死配置是必要但不充分的**；GT 管线必须逐参数标定，不能假设"选了 Average 就是 Average"。
> → **论文里这句话比"默认 1 秒平均"更有冲击力**：连显式指定的分析语义都可能被静默改写。
> → S0.1-6 的标定实验要扩展：**对每类参数分别验证所选公式是否真的生效**。
>
> 如果离线 GT 管线算瞬时值、而 agent 从 GUI 默认视图读 1 秒平均值，**两者会在每一道指标题上系统性不一致，而且你不会察觉**。

**这是本次读手册最重要的收获，也是一个可以写进论文的普适洞见**：专业软件的"真值"是**配置相关**的。现有 GUI agent benchmark 几乎都假设真值唯一，因为它们的目标软件（浏览器、Office）没有这层分析语义。

**两条应对，必须都做**：
1. **每个任务显式钉住 `analysis_config`**，并作为 initial state 的一部分被 reset 校验。
2. **离线 GT 管线必须复刻 XCAP 的聚合语义**（尤其 1 秒平均、事件不采样）。这需要在 S0.1 里实测标定，不能靠猜。

### T4 — 内建 RCA 算法的地位（**已降级**）

产品单页宣传了 "RCA Algorithm"，但 **55 页 User Guide 的功能清单里没有出现**。最接近的是 `Signaling Flow`（L3 信令呼叫流，含消息方向与 piggybacked NAS）。

→ **降级为待确认项**：可能在随附的分册 PDF 里、可能是企业版功能、也可能只是营销话术。S0.3 照常摸底，但不再作为高风险项。

---

## 2. 应对方法（v1）

### 2.1 GT 结构：带 confound 声明 + 钉死配置

```yaml
task_id: TELEOP-HO-017
model:
  source_logs: [teleop_run_07.drm]
  parse_mode: merge                    # merge | separate（CLI: MG / 默认 separate）
  selective_parsing: profile_full_v2   # ⭐ T5：钉住解析剖面
                                       #    ⚠️ 仅 Qualcomm LTE 可用 → 按 (chipset, RAT) 门控
  build_path: gui                      # ⭐ T8c：gui | cli —— CLI 建的 Model 缺一整类后处理产物
  viewer_mode: message                 # ⭐ T8b：message | log_only
                                       #    log_only 建模更快，但消息级证据不可得（需 Reparsing）
  multi_process: false                 # ⭐ T8a：true 会产生 sub-model 子目录，改变磁盘结构
  process_as_file: false               #    PAF；开启会静默把 multi_process 降级为单进程
analysis_config:                       # ⭐ T5：钉住分析语义
  sampling_rate: "1sec"                # "All" | "1sec" | ...  🔴 不显式设置时恒为 1sec
  filters: []                          # 🔴 filter 是**全局**的（跨 Graph/Map/Table/CDF/PDF），
                                       #    必须显式声明为空；每个 filter =
                                       #    (param, op ∈ {=,<>,>,>=,<,<=}, value, linker ∈ {AND,OR,None})
  binning: none
  workspace: baseline_v1.wrk
  canonical_clock: TIME_STAMP          # ⭐ T6：所有时间戳与分桶的基准（v2 由 chipset 改）
  verification_clock: chipset          #     仅在含该列的 9% 表中用于表内排序与交叉校验
  clock_offset_model: drift_v1         #     两钟换算模型（非常数）

symptom: {type: control_loop_timeout, t: 128.42, duration_ms: 2300}

causal_chain:                          # 有序，每环独立可核验
  - {level: 4, layer: APP,       claim: "control RTT > 200ms deadline", verifier: chk_rtt}
  - {level: 3, layer: TRANSPORT, claim: "uplink gap 126ms",             verifier: chk_ulgap}
  - {level: 2, layer: RRC,       claim: "handover to PCI 301 @127.9",   verifier: chk_ho}
  - {level: 1, layer: RF,        claim: "serving RSRP -118dBm for 1.4s pre-HO", verifier: chk_rsrp}

primary_attribution: {level: 2, layer: RRC, mechanism: handover_interruption}
answer_scope: "归因到 layer + mechanism；不要求 sub-mechanism"

known_confounds:                       # ⭐ T1：直接回应 implicit B
  - {phenomenon: "PRB 分配 t=126.8–127.2 骤降", layer: MAC,
     relation: consequence_of_primary, grading: partial}
  - {phenomenon: "TCP 重传突发 t=128.0",       layer: TRANSPORT,
     relation: consequence_of_primary, grading: partial}
  - {phenomenon: "邻区 PCI 305 同频干扰",       layer: RF,
     relation: plausible_alternative,  grading: full}     # 专家认可的另解，也算全对

observability_envelope:                # ⭐ T3
  gt_pipeline_parses: [rrc, nas, pdcp_tput, rsrp, rsrq, sinr, tcp_stats, app_metrics]
  xcap_additionally_shows: [prb_scheduling_map, dci_per_tti, beamforming_ssb_array, hex_bit_raw]
  # agent 依据 envelope 外字段作答 → 不自动判错，进 review 队列
```

> ### ⭐ v2 变更：`canonical_clock` 由 `chipset` 改为 `TIME_STAMP`
>
> **原取值的首要理由已作废。** 当初选 chipset 的依据是"逆序率相差 35 倍（19.57% vs 0.00%）"，
> 但 W01-B2 查明那 19.57% 是**旧版读表器列偏移 bug 的产物**；重测后两个时钟列**都是 0.00%**（约 158 万对相邻记录）。
> **没有证据再支持原取值。** 改用 TIME_STAMP 的三条实测理由：
>
> | # | 理由 | 实测依据（W01-B2） |
> |---|---|---|
> | 1 | **可得性** | 6,401 张表中只有 **572 张（9%）** 含 chipset 列。按 chipset 立规格，**91% 的表无法实现** |
> | 2 | 🔴 **跨层一致性** | `TIME_STAMP` 跨层一致；`ChipTime` 在 **app 层事件表比无线层整整差 5 小时**（= 采集地 CDT），**且不报错**。跨层归因是本文头条，不能用一个跨层不一致的时钟做基准 |
> | 3 | **与 GUI 对齐** | **XCAP 自己按 `TIME_STAMP` 分桶。** GT 用 chipset 会与 agent 从界面读到的一切系统性错位，把**规格错位**误判成 **agent 错误** |
>
> ⚠️ **保留 chipset 作 `verification_clock`**：它更贴近设备侧真实事件，在含该列的表内用于排序与交叉校验；
> 但**不作为跨层对齐与分桶的基准**。
>
> ⭐ **那个 5 小时偏差应当直接做成一道 L5 题** —— 按 chipset 做跨层对齐会整体错 5 小时，
> 两边时间戳各自都"看起来正常"，没有任何报错。这是 T6 最强的实例。

**流程上的关键改变**：GT 不再是"注入了 A 所以 GT=A"，而是
**注入 A → 对该 trace 跑全现象扫描（不只扫 A）→ 逐个标注 relation 与 grading**。

### 2.2 分歧驱动的 GT 迭代精化（低成本，本身即方法论贡献）

专家标不起全部，但**标得起分歧**：

```
GT v1（自动） → 跑 baseline agents → 收集所有"被判错"的答案
  → 聚类（相同错误模式归为一类）→ 专家只审集群代表（几十个而非几千个）
  → 合法另解写进 known_confounds → GT v2
```

**写进论文**：*"GT v1→v2 共裁决 N 个分歧集群，其中 X% 的初判错误实为合法另解。"* 这个数字本身就是对 T1 的量化证明，也是关于"RCA benchmark 有多难构建"的发现。

---

## 3. Stage 0 —— 前期确认与准备（本月重点）

按依赖排序。**S0.1 未过之前不要写生成器代码。**

### S0.1 数据可用性与聚合语义标定 🔴 最高优先

| # | 检查项 | 通过标准 / 产出 |
|---|---|---|
| 1 | 取 1 条 streaming + 1 条 teleop 数据建 Model | 能建成、不报错 |
| 2 | 若非 XCAL 原生格式，走哪条导入路径？ | Ethereal(pcap) / CSV / txt / TEMS / Converter，**记录导入后哪些层还在** |
| 3 | 逐层清点可见字段 | L1 / L2 / L3(RRC,NAS) / TCP-IP 各层实际能看到什么，逐项列表 |
| 4 | app 层指标能否进入并被同步显示 | rebuffer / 码率 / 控制 RTT 是否作为可同步参数出现（CSV Import 路径可能是关键） |
| 5 | **`Synchro Viewer (TCP Flow)` 是否可用** | 手册：*"Activated only when Modem/Network adapter data is included"*，且需在 Selective Parsing 里勾选 —— **这是 transport 层视图，跨层任务的关键** |
| 6 | 🔴 **采样语义标定** | 同一参数分别用 `All` 与 `1sec` 建两个 Model，比对同一时刻读数，**量化差异**；确认事件/信令确实不被采样 |
| 🔴 7 | **T6 时钟问题定案** | ① 证实/推翻"`Time` 是排序键、`Chipset Timestamp` 是设备真钟"；② 量化两者漂移随时长的走势；③ 确认 app 层指标用的是第三个时钟吗；④ **选定一个规范时钟并写进 `analysis_config`**；⑤ 确认 XCAP 的 `Range to Sync` / `Sync Manager` 用的是哪个 |
| 8 | Selective Parsing 剖面 | 定义并固化 1–2 个标准解析剖面，作为所有任务的基线 |

**产出**：
- `S0.1_data_layer_inventory.md` —— 层 × 字段 × 可见 × 可导出 的总表
- `S0.1_aggregation_semantics.md` —— ⭐ **XCAP 聚合语义标定报告**，离线 GT 管线据此实现

> 这两份是后面一切的地基：它们同时定义了**任务的可能空间**、`observability_envelope` 与 `analysis_config`。

### S0.2 Escape hatch 实测（重点已从"有没有"变成"能拿到什么"）

按优先级：

| 优先 | 项目 | 要回答 |
|---|---|---|
| 🔴 0 | **开源 diag parser 通路**（结论 D） | 用 MobileInsight / SCAT / QCSuper 解析导出 CSV 里的 hex，与 XCAP 的 `Detail` 文本**逐字段交叉校验**。**通过则离线 GT 管线定案** |
| 🔴 1 | **pgDatabase** | 是什么 schema？能否只读连接？解析后的参数/事件/消息是否都在里面？若可读则更省事 |
| 🔴 2 | **File Export using Command Prompt** | 语法是什么？能导出哪些窗口/参数？能否参数化时间窗与小区？能否完全脱离 GUI？**决定能否无人值守批量导出** |
| 🔴 2b | **导出体量与可行性** | 样本中单条消息的 Detail 就有数十行。482k 条消息的全量导出规模多大？是否必须先 filter 再导出（→ 与全局 filter 状态耦合）？ |
| 3 | Batch Export / Scheduler | 批量与定时的粒度；能否用于无人值守的数据准备 |
| 4 | Report（pdf/excel/doc/html）+ User Defined Report | 6 类报告各自导出什么字段；是否结构化可解析；能否参数化 |
| 5 | **`.oxf` / `.wrk` 格式** | 是文本还是二进制？可否程序化生成？—— **决定 M6 能否编译成原生格式** |
| 6 | **Multi Process** | 到底是什么？能否多实例并行？**若能，capacity=1 的假设要修正** |
| 7 | Converter → PCAP | 导出的 pcap 保留哪些层？能否支撑 §0-C 的 XCAP↔Wireshark 配对实验 |

**产出**：`S0.2_hatch_matrix.md`
> 每条可结构化导出的路径 = 一个**免费 oracle** + 一批**现成任务模板**。

### S0.3 内建分析功能清点（= 现成的任务模板库）

手册已列出一批领域分析功能，逐个实测其输入/输出/可导出性。**这些本身就是高质量任务模板**：

| 来源 | 功能 |
|---|---|
| Tools–Analysis | Coverage Info、LTE RB Scheduling Map、**LTE DCI Analysis (per TTI)**、LTE PCI Match Ratio、LTE Shadow Area Estimation、LTE/5G-NR Cell Throughput、5G-NR DSS、**Sector Swap Analysis**、5G-NR Beamforming Info、5G-NR SCS Frame Format |
| Ch.4 Analysis Method | Message Duration Time Analysis、Delta Map（前后对比）、M-to-M Statistics、**PSYNC/PSC Pollution Analysis**、**Neighbor List Analysis**、**Finding Missing Neighbors**、Serving Cell Line Drawing、Area Binning、Event Call Extract from Statistics、Voice Wave Silence Detection |
| Viewer | **Signaling Flow**（L3 呼叫流，含方向与 piggybacked NAS）、**Synchro Viewer (TCP Flow)** |

同时在此确认 **产品单页宣传的 "RCA Algorithm" 是否真实存在**（可能在随附分册 PDF / 企业版）。

**产出**：`S0.3_builtin_analysis_catalog.md`

### S0.4 现象 detector battery（应对 T1 的核心准备，离线，不占 license）

第一步：列出"值得扫描的现象"清单，按层组织：

| 层 | 现象（每条最终要有一个 detector） |
|---|---|
| RF/PHY | 覆盖空洞、SINR 恶化、同频干扰、**PSC/PSYNC pollution**、波束切换异常 |
| MAC/调度 | PRB 分配骤降、调度饥饿、BLER 升高、MCS 塌陷、**DCI 异常** |
| RLC/PDCP | 重传激增、缓冲堆积 |
| RRC/移动性 | 切换（成功/失败/乒乓）、重建、RLF、**missing neighbor**、**sector swap** |
| NAS/核心网 | 注册拒绝、会话建立失败、鉴权异常 |
| Transport | RTT 尖峰、TCP 重传突发、拥塞窗口塌陷、丢包 |
| App | rebuffer、码率切换、控制回路超时、丢帧 |

> 注意：其中带粗体的几项 **XCAP 已有内建分析功能**（S0.3）。这些 detector 可以**用 XCAP 的输出做交叉校验**，是很好的正确性检查。

第二步：每条现象写一个可对原始数据运行的 detector，输出 `{存在?, 时间窗, 强度, 置信度}`。
**⚠️ detector 的聚合语义必须与 S0.1-6 标定的 XCAP 语义一致。**

**产出**：`detectors/` + `S0.4_phenomenon_taxonomy.md`
> 一次投入三处复用：① 全现象扫描（发现 implicit B）② 生成 causal_chain ③ 核验证据（`verifier: chk_*`）。**Stage 0 性价比最高的一项。**

### S0.5 共现分析 —— 量化 implicit B 到底有多严重

detector 就绪后对全部已有数据跑一遍（离线、免费、可并行）：

- **现象共现矩阵**：注入 A 时 B 同时出现的频率
- 每条 trace 平均触发多少个现象
- **"干净 case"占比**：只有单一显著现象的 trace 有多少

| 共现情况 | 策略 |
|---|---|
| 罕见 | 单标签 GT 够用，`known_confounds` 作保险 |
| 普遍 | **必须**走 §2.1 结构化 GT；考虑把"识别多重并发问题"设为一类任务 |
| 极普遍 | **"单一根因"在你们数据上不成立** → 任务改为"排序/枚举贡献因素"。这会是一个诚实且有价值的转向 |

> ⚠️ **S0.5 可能改变任务的基本形态，必须排在规模化生成之前。** 这是本文档最重要的排期约束。

### S0.6 索取随附分册

手册正文反复引用一批分册 PDF（*"that comes with it"*），它们才是细节所在：

`Chapter 4_Selective Parsing` · `Chapter 4_Model Creation Plan (Scheduler)` · `Chapter 5_Main Window_File_File` · `Chapter 5_Main Window_Tools_Report` · `Chapter 5_Main Window_Tools_Analysis` · `Chapter 5_Main Window_Tools_Setting_Options` · `Chapter 5_Mouse Menu_{Graph,Table,Map,Message,CDFPDF,Event}` · `Chapter 5_Tree Mouse Menu` · `Chapter 5_DSDS Feature`

> **最想要的两份**：`Tools_Analysis`（领域功能细节 → 任务模板）与 `Tools_Setting_Options`（配置项全集 → T5 的完整清单）。另外单独问一句有没有 **命令行 / pgDatabase / API 的说明文档**。

---

## 4. Stage 0 时间表与依赖

```
第 1 周 ┃ S0.1 数据可用性 + 采样语义标定 🔴
       ┃ S0.6 索取分册（尽早，等待有周期）
       ┃ S0.4 现象清单（列表阶段，离线）
       ┃
第 2 周 ┃ S0.2 hatch 实测（pgDatabase / CLI export 优先）
       ┃ S0.3 内建分析功能清点
       ┃ S0.4 detector 实现（离线并行）
       ┃
第 3 周 ┃ S0.5 共现分析 ← 依赖 S0.4
       ┃ ★ 决策门 GT-G1：任务基本形态
       ┃
第 4 周 ┃ GT schema 定稿 v2；10 个手工任务样例
       ┃ 跑通「生成 → 执行 → 评分」全链路
```

**决策门 GT-G1（第 3 周末）**：依 S0.5 共现结果确定任务形态。此门未过不进入规模化生成。

---

## 5. 评分 rubric（v0，待 S0.5 后修订）

### L1–L4：精确匹配（无歧义）

| 判定 | 条件 |
|---|---|
| **Correct** | 答案/状态与 oracle 精确一致（数值题带容差声明） |
| **Config-mismatch** | 差异可由 `analysis_config` 解释（瞬时 vs 1 秒平均、时钟选择）→ **不判错**，进 review 并回头修 GT 管线 |
| **Wrong** | 其它 |

> L1–L4 是**软件使用能力**的干净度量。T1/T2/T3 在这里不适用。

### L5（RCA）：rubric + 证据

| 判定 | 条件 |
|---|---|
| **Full** | 命中 `primary_attribution`，**且**所提证据全部通过核验 |
| **Full（另解）** | 命中 `grading: full` 的 `plausible_alternative`，且证据通过核验 |
| **Partial** | 命中 `causal_chain` 上其它环，或命中 `grading: partial` 的 confound |
| **Unverifiable** | 结论字面正确但**证据核验不通过** ← 捕捉蒙对 |
| **Config-mismatch** | ⭐ 答案与 GT 的差异可由 `analysis_config` 解释（如瞬时 vs 1 秒平均）→ 不判错，进 review 并回头修 GT 管线 |
| **Review** | 依据 `observability_envelope` 之外的字段作答 |
| **Wrong** | 所述现象不存在，或存在但与症状无因果关系 |

> **Unverifiable 必须单独报告**。RCA 候选就那么几个，蒙对很常见，不区分会严重高估成功率。

---

## 6. 待定问题

- [ ] 🔴 你们的数据走哪条导入路径？导入后保留哪些层？`Synchro Viewer (TCP Flow)` 能否激活？
- [ ] 🔴 synthetic 是 (a) 受控故障注入（有层标签）还是 (b) 真实 trace 上程序化生成任务？
- [ ] 🔴 注入实验的**注入日志有多完整**（类型/时刻/强度/持续）？决定 causal_chain 能自动生成多少
- [ ] **pgDatabase 是否可读**？—— 若可，离线 GT 管线成本骤降
- [ ] **Multi Process 到底是什么**？—— 若能多实例并行，capacity=1 假设要修正，整个 throughput 预算重算
- [ ] `.oxf` / `.wrk` 是否文本可编程生成？—— 决定 M6 能否编译成原生格式
- [ ] 产品单页的 "RCA Algorithm" 是否真实存在？在哪个分册/版本？
- [ ] 专家可投入的裁决小时数？（决定 §2.2 能跑几轮）
- [ ] 是否有 Map 引擎（MapX / MapXtreme）授权？—— 无则 Map 类任务全部不可做

---

## Actions

### 第 1 周
- [ ] 🔴 **结论 D 验证**：抽导出 CSV 的 hex → MobileInsight/SCAT/QCSuper 解析 → 与 `Detail` 文本逐字段交叉校验。**通过则离线 GT 管线定案**
- [ ] 🔴 **T6 时钟定案**（`Time` vs `Chipset Timestamp` vs app 侧时钟；漂移模型；选定规范时钟）
- [ ] S0.1-1~5 建 Model，产出层×字段清单
- [ ] 🔴 S0.1-6 采样语义标定（`All` vs `1sec` 双 Model 对比）
- [ ] 写 CSV parser（**注意表头 8 列 / 数据 10 字段的错位**，且 Detail 内含换行与逗号，必须按引号正确切分）
- [ ] S0.1-8 固化标准 Selective Parsing 剖面
- [ ] S0.6 索取分册 + 命令行/数据库文档
- [ ] S0.4 现象清单（按层列全，先不写代码）

### 第 2 周
- [ ] 🔴 S0.2-1 pgDatabase 可读性与 schema
- [ ] 🔴 S0.2-2 命令行导出语法与覆盖范围
- [ ] S0.2-3~7 批量/报告/oxf·wrk/Multi Process/PCAP 转换
- [ ] S0.3 内建分析功能清点 + 确认 RCA Algorithm 是否存在
- [ ] S0.4 detector battery 实现（离线并行，**聚合语义对齐 S0.1-6**）

### 第 3 周
- [ ] S0.5 全数据共现分析 → 共现矩阵
- [ ] ★ **决策门 GT-G1：任务基本形态**

### 第 4 周
- [ ] GT schema v2 定稿（含 `analysis_config` 与 `known_confounds`）
- [ ] 10 个手工任务样例
- [ ] 跑通生成 → 执行 → 评分全链路

---

## Notes

- **最重要的收获是 T5**：专业软件的"真值"是**分析配置的函数**，不是原始数据的函数。默认 1 秒平均采样这一条，就足以让所有指标类任务的 GT 系统性错位而不被察觉。现有 GUI agent benchmark 普遍没有这层问题，因为浏览器和 Office 没有分析语义。**这是一个可以写进论文的普适洞见。**
- **T1 与能力反相关**：越强的 agent 越容易被误判为错。任何 RCA benchmark 不处理这一点都不可信。
- **⭐ 导出样本带来的最大收获（结论 D）**：`Detail` 里的 hex 是标准 Qualcomm diag packet（已用 0xB064 / 0xB087 解码验证）→ **离线 GT 管线可以完全建在开源 diag parser 上，无损、免 license、独立于 XCAP**。这同时消解了 T3（GT 看到的是原始码流，不再比 agent 少），并给 GT 第三方权威性。
- **🔴 T6 是新的头号排期风险**：两个时钟差 ~17.6 s 且在漂移，且 `Time` 列极可能只是排序键。**任何时间相关任务在它定案之前都不能做。**
- **排期铁律**：T6 时钟定案 → S0.4 detector → S0.5 共现分析 → 才能规模化生成。
- **三个待验证的高价值 hatch**：开源 diag parser 通路（最高）、`File Export using Command Prompt`、`pgDatabase`；外加 `Multi Process`（可能推翻 capacity=1）。
