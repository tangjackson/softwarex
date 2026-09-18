---
title: "XCAP 支持 KB 关键发现"
type: project
created: 2026-08-29
status: active
area: "[[Research]]"
priority: P1
tags: [project, research, sigcomm, xcap, reference]
---

# XCAP 支持 KB 关键发现

> 来源：`support.usa.accuver.com/portal/en/kb/user-guides/guides/xcap-m`（需登录，共 3 页 ~41 篇）
> 注：**XCAP-M ≡ XCAP**（已确认）。多数文章正文内联，少数为 PDF 附件。

---

## 1. ⭐⭐⭐ CLI 完整命令参考（`[XCAP-M] File Export using Command Prompt`）

**`XCAP-M.exe` 是一个真正的批处理入口** —— 建模 + 导出全部可脱离 GUI。

```bat
:: 导出信令消息
"XCAP-M.exe" -CM "<log folder>" "<out>" -MEP MBS "<fav path>+<name>.mes" L3

:: 导出 Favorite 指标列表
"XCAP-M.exe" -CM "<log folder>" "<out>" -FEP "<fav path>+<name>.fav" -MG

:: 导出 User Defined Report
"XCAP-M.exe" -CM "<log folder>" "<out>" -MG -UDREP "UDR1; UDR2"

:: 多个 fav 模板
"XCAP-M.exe" -CM "<log folder>" "<out>" -FEP "<f1>.fav; <f2>.fav"

:: 组合
"XCAP-M.exe" -CM "<log>" "<out>" -FEP "<n>.fav" -MG -UDREP "UDR1; UDR2"

:: 多 Log Set（一条命令串多个 -CM）
"XCAP-M.exe" -CM "LogSet1" "<out>" -FEP ... -CM "LogSet2" "<out>" -FEP ...

:: 消息导出带 GPS
"XCAP-M.exe" -CM "<log>" "<out>" -MEP G MBS "<n>.mes"

:: Table + Statistics / 仅 Statistics
"XCAP-M.exe" -CM "<log>" "<out>" -FEP "<n>.fav" Full
"XCAP-M.exe" -CM "<log>" "<out>" -FEP "<n>.fav" Stats

:: DSDS（双卡）
"XCAP-M.exe" -CM "<log>" "<out>" -MG -DSDS -FEP "<n>.fav"
```

### 1.1 完整命令表（`Chapter 4_File Export using Command Prompt.pdf`，7 页）

| 类别 | 命令 | 说明 |
|---|---|---|
| **进程生命周期** | `INI` / `FIN` / `KX` | 启动 / 关闭 / **强杀**程序 |
| **建模** | `CM "<in>" "<out>"` | Create Model |
| | `MG` | Merge；**不设则 Separate** |
| | `PAF` | Process as a file（日志被切分时用） |
| | **`MOP "<model path>"`** | ⭐ **Open Model** |
| | **`MDEL`** | ⭐ **Delete Model** |
| | `UTC` | ⭐ **时区选项**（与 T6 时钟问题直接相关） |
| **采样率** | `All` / `10ms` / `100ms` / `200ms` / `500ms` / `1000ms` / `2000ms` / `3000ms` / `4000ms` / `5000ms` / `10000ms` | 🔴 **不设则默认 1s** |
| **导出** | `FEP "<x>.fav"` | Favorite Export（KPI） |
| | `MEP` | Logging Message Export（旧命令 `MB`） |
| | `MEP MBS "<x>.mes"` | 按 Favorite 消息模板导出 |
| | **`PGEP`** | 🔴🔴 **PostgreSQL Data DB 导出** —— 前提：先在 XCAP 里配置 PostgreSQL 服务器 |
| | `UDREP "r1; r2"` | User Defined Report |
| **消息子选项** | `D` / `H` / `PK` / `L3` / `PU` / `BRC` | Detail / Hex / Packet / Layer3 / 部分 ueCapabilityInfo / brace 格式 |
| **其它** | `DSDS`、`Full`、`Stats`、`;`（分隔符） | |

> ⚠️ Logging Message 部分**不能含连字符 `-`**。KB 评论区有人反馈文件夹路径批量导出报错、只能逐个文件名（Samsung `.sdm`）—— 需实测。

**旧命令集**（`-EXPORT`，与新命令**不兼容**）：`-EXPORT <drm folder> <csv path> <fav path>` 配 `-MB` / `-TXT` / `-L3` / `-D` / `-H` / `-all` / `-mg` / `-BX`。
> ⭐ **完整九行命令表见 §5.7.2**（来自安装目录的 `Usage of Favorite Info`）——含两个此处没有的关键事实：多 log 导出会**静默互相覆盖**（`-mg` 是解法），以及 `-all` 之外**采样率恒默认 1 sec**。
> 💡 **`sample data.md` 极可能就是用 `-MB -D -H` 导出的**（含 Detail 文本 + hex）。

**支持的日志格式**：`drm`, `drx`, `scn`, `scx`, `ISF/DLF`（CLI 建模）；Scheduler 另支持 `aof`, `aox`, `zip`。
> ⚠️ **CLI 建模的格式列表里没有 pcap / CSV** —— 那两条只能走 GUI 的 Import。**E0.0 的关键约束。**

**支持通配符**：`C:\temp\log\DR24*.drx` 等。

⭐ **`-?` 打印帮助**（文档里写作 `"OPTis Analyzer.exe" -?`，说明二进制曾用此名）。
> **W01 最便宜、最高价值的一步：在装好的 XCAP 上直接跑 `-?`，拿到该版本的权威命令全集。**

### 1.2 三个直接的用途

1. **U6 已答且远好于预期。** 大量 L1–L4 任务的 oracle 可完全走 CLI，不占 GUI。
2. ⭐ **M4 的检查点原语齐了**：`KX`（杀）/ `MOP`（重开 Model）/ `MDEL`（删）/ `INI`（启动）—— **三级检查点层级可以完全脚本化**，不必依赖 GUI 操作。E0.4 从"设计方案"变成"测各级耗时"。
3. 🔴 **`PGEP` 证实 pgDatabase 通路存在** —— 配置 PostgreSQL 服务器后可直接导出到库。**离线 GT 管线可以建在结构化数据库上**，不必解析 CSV。这是 S0.2-1 的答案。

---

## 1.5 ⭐ Scheduler：无人值守的批量流水线（13 页）

`Tools → Scheduler`，两种模式：

**Local mode** — `(Single) Today` / `(Single) Tomorrow` / `(Repeat) Daily` / `(Repeat) Weekly`（可指定星期、时刻、起止日期）

**FTP mode** — `Realtime monitoring`（持续监控 FTP 目录，靠 `Allfileuploadcompleted.txt` 标识上传完成；失败重试 6 次 / 每 5 分钟；处理完移入 `_Comp_`）或 `Specified time`（单次拉取）

**建模选项**：`Sampling Rate`、`Separate` / `Merge` / **`Port Merge`**、`PAF`、**`MtoM`**（Mobile-to-Mobile，需配 Port Merge）、`Filter`（只处理满足条件的文件）
> 默认值：**Sampling Rate = 1，Merge**

**文件管理**：建模成功 → `Completed` 文件夹；**失败 → `Quarantine` 文件夹**
> 💡 **这是现成的"环境失败 vs agent 失败"分流机制**，可直接接进母计划的 job 状态机（`FAILED_ENVIRONMENT`）。

**`Extra Job (To Excel)` → `Batch Export Manager`**：建模完成后自动串接批量导出。

> **用途**：数据准备阶段（几百条 trace 建 Model）可以完全无人值守，不占 W01–W03 的人力，也不占 GUI 交互预算。

---

## 2. ⭐⭐⭐ 厂商自己就是"GUI 编写模板 / CLI 批量执行"的架构 —— M6 得到强验证

`[XCAP-M] Using CLI in XCAP-M` 描述的官方工作流：

```
GUI: Tools → Favorite Manager → 勾选参数 → 导出 <name>.fav
                                    ↓
CLI: XCAP-M.exe -CM ... -FEP <name>.fav -MG -SamplingRate
```

消息侧同理：`Options → Misc → Batch Export → Logging Message → Setting → New` 建模板 → 导出 `.mes` → CLI 引用。

> 🎯 **这正是母计划 M6 提出的设计：skill 不是"回放一串点击"，而是编译成宿主软件自己的持久化格式。**
> 区别在于——**这不再是我们的提议，而是厂商既有的架构**。论文里的论证强度完全不同：
> *"专业软件本就区分「编写分析意图」与「批量执行」两个阶段；agent 只需学会前者，后者是免费的。"*
>
> 类比从"写 Makefile 而不是记 shell 历史"升级为：**这个软件本来就有 Makefile，只是需要人来写。**

**M6 的产物应当是**：`.fav`（KPI 模板）、`.mes`（消息模板）、UDR 模板（`.xudr` / `.xlsm`）、`.wrk`（工作区）、`.oxf`（选项集）。

UDR 模板路径已知：`C:\Users\<user>\Documents\Accuver\XCAP-M\UserReports\Template`

---

## 3. ⭐⭐ KPI 计算逻辑有官方文档 —— T5 的直接解药

不再需要反向工程猜测 XCAP 的聚合语义。已确认存在四篇：

| 文章 | 内容 |
|---|---|
| `LTE PDSCH Throughput Calculation` | PDF 附件（2 MB，已下载） |
| `Qualcomm 5G-NR MAC PDSCH Info PCell KPI Calculation Logic` | 内联，见下 |
| `Qualcomm 5G-NR MAC PUSCH Info PCell KPI Calculation Logic` | 内联 |
| `RLC Statistics Summary / DL Dropped PDUs Calculation Logic` | 内联，见下 |

### 3.1 ⭐ T7（计数器 vs 事件）被官方例子直接证实

`RLC Statistics Summary / DL Dropped PDUs`：

> 参考 log code `[0xB84D] NR5G RLC DL Stats`。在 `23:06:47` 这一秒内有 4 条记录；
> `Num Dropped PDU` 从 **109 增至 228**，**差值 119** 即该秒的 KPI 值。

**即：KPI = 累计计数器在 1 秒 bin 内的差分。** 与我们此前标注的 T7 完全一致。
→ detector 与离线 GT 必须区分 counter / gauge / event，且**必须复刻 1 秒分箱**。

### 3.2 5G-NR MAC PDSCH KPI 的推导链（示例）

```
Number of Total Slots        ← 由 Frame/Slot 编号跨度算出
                               (20-8) + 20*(1022-918+1) + 17 = 2129
DL Slot Ratio                ← 由 SCS 决定；SCS=30kHz → (14 + 8/14)/20 = 72.86%
Number of Total DL Slots     = Total Slots × DL Slot Ratio = 2129 × 72.86% ≈ 1552
Number of Allocated DL Slots ← 数 [0xB887] NR5G MAC PDSCH Status 在该秒内的条数 (278)
Slot Utilization Ratio       = Allocated DL Slots / Total DL Slots
RB Num[Avg]_Slot Util DL     = RB Num[Avg] × Slot Utilization Ratio / 100
```

> **这是 T5 的教科书式例证**：一个看似简单的 KPI，真值依赖 SCS、帧/时隙编号、分箱边界、以及一条特定 log code 的记录数。
> **论文里可以直接引用这条推导**来论证"专业软件的真值是分析配置的函数"。

---

## 4. ⭐ UDR（User Defined Report）—— 可编程的报告平台

`[XCAP-M] Introduction of UDR`：

- 两步：`User Defined Report Setting`（设计模板）→ `User Defined Report`（发布 `.xlsm`）
- Report Type：`Normal`（单 model）/ `Comparison`（**2–50 个 model 对比**）← **L4 对比类任务的现成载体**
- 参数类型：Chart（PDF/CDF/PIE/BAR/Top10/Correlation/Box Plots）、Duration/Ratio、Table（带阈值判定 Good/Bad）、Msg.Duration
- **Table 类型支持阈值 + 目标百分比双条件判定**（例："RSRP>80 的比例超过 90% 判为 Good"）→ **可直接当 oracle 用**
- 支持 Binning Option；与 Filtering Manager 联动，滤波配置可 import/export
- Samsung BMT KPI Report 案例：**8 报告 / 167 sheet / 1727 KPI**，"原本人工两周的提取压缩到一天"

> **两个用途**：① UDR 模板是 M6 的一种 skill 产物；② UDR 的阈值判定机制可以直接充当 L2/L4 的 oracle。
> **一个隐患**：1727 个 KPI 说明参数空间极大，M2（分层惰性观测）的必要性被强化。

---

## 5. Import File Feature —— E0.0 的具体约束

`[XCAP-M] Import File Feature`（Import Scanner CSV）：

- 走 `Import File` 按钮导入 CSV
- ⚠️ **格式要求硬性**：前 4 列必须命名为 `Day`, `Time Stamp`, `Latitude`, `Longitude`
- 需确认时间戳格式与文件日期一致
- 导入后 KPI 出现在 **`Imported Data`** 参数树下

> **对 streaming / teleop 的 app 层指标**（rebuffer、码率、控制 RTT），这是把它们喂进 XCAP 并参与时间同步的**具体通路**。
> ⚠️ 但要求 GPS 列 —— 若 app 指标无 GPS，需先做列对齐/补齐。**W01 的 A2/A3 要专门验这条。**

---

## 6. ⭐ Release Notes 存在 → 真实 UI 漂移材料

相关文章里出现：`20250430_Analyzer_ReleaseNote_(O5.33.3.20)`、`20250327_Analyzer_ReleaseNote_(O5.33.2.17)`

> 结合 User Guide 的修订历史（5.22.xx 2019 → 5.33.xx 2025，共 9 版），
> **M6 的漂移修复实验可以用真实版本演化替代人工注入漂移**，而且是在 XCAP 上做，不必退到 Wireshark。真实漂移的说服力远高于合成漂移。
> **行动**：向厂商索取历史版本安装包与完整 release notes。

---

## 5.5 ⭐ 分册 PDF 精读所得（4 份）

### 5.5.1 🔴🔴 T6 时钟问题得到官方证实

`Chapter 4_Message Duration Time Analysis.pdf` p2 原文：

> *"When logged with Chipset Time, it displays Time Difference value using **Chipset Time** between two Messages."*

> **XCAP 自己在计算消息间时延时用的是 Chipset Time。**
> → 强烈支持我们的假设：**`Chipset Timestamp` 是权威时钟，`Time` 列只是排序键**。
> ~~→ `analysis_config.canonical_clock` 应定为 `chipset`~~
> ⚠️ **已被 W01-B2 的实测推翻。** 规格改为 `canonical_clock = TIME_STAMP`，chipset 降为 `verification_clock`。
> 「XCAP 用 Chipset Time 算消息间时延」这条**仍然成立**，但它只说明 chipset 适合**表内**时间差，不说明它适合**跨层基准** ——
> 而 chipset 列只存在于 9% 的表，且在 app 层比无线层差 5 小时。详见 [[TASK-GT-GENERATION-PLAN]] §2.1。

### 5.5.2 ⭐⭐ Message Duration Time Analysis = 现成的 L2/L4 任务生成器

- 对 **L3 Message / Autocall Event / SIP Message** 按用户定义的区间（`From` 消息 → `To` 消息）计算时延
- `First/Last`：重复消息时取第一条还是最后一条 ← **又一个改变真值的配置（T5）**
- `With Regard to Call`（按 Autocall 划分呼叫区间，默认）vs `Regardless of Call` ← 同上
- 输出 **min / max / average 统计**，并对不满足用户阈值的 Call 标出 Call Number ← **内建 oracle**

> **直接可用**：`"RRC Connection Request → RRC Connection Setup Complete 的时延是多少"` 这类任务，
> 有明确定义、有内建统计、有阈值判定。**L2/L4 的高质量模板来源。**

### 5.5.3 ⭐ 发现一类"不可逆/不可持久"的操作 —— M4 的真实素材

`Chapter 4_LTE DCI Analysis.pdf` p2 原文：

> *"The result window does **not saved** and **cannot be re-opened**."*

> **DCI Analysis 的结果窗口是易失的** —— 关掉就没了，要重新跑 Reprocess（per-TTI 重处理，昂贵）。
> RB Scheduling Map 同样需要先点 `Reprocess` 才能出结果。
>
> 🎯 **这正是 M4 要处理的情形，而且是软件里真实存在的、非我们杜撰的**：
> 有些动作的产物无法检查点、无法回滚，只能重做，且重做代价高。
> **论文里可以直接引用这句话**作为"GUI 没有 `git checkout`"的实证。

另：DCI Analysis 的参数列表**可存成文件并在其它项目中导入** → 又一种 M6 的 skill 产物。
构建该列表需要**从参数树拖放** → M3 的硬案例。

### 5.5.4 ⚠️ 窗口几何是语义相关的 —— 修正状态序列化器的设计

`Chapter 4_Exporting Batch using Batch Export Feature.pdf` p6：

> *"Configure Graph, map and chart according to Window size before saving workspace as **export is processed according the displayed size**."*

> 🔴 **与母计划 P1 的设计铁律冲突**：原先规定状态序列化要"归一化掉窗口几何"。
> 但 workspace / 图表导出的**输出内容取决于窗口尺寸** —— 几何在这里是**语义承重**的。
>
> **修正**：状态序列化区分两类字段 —— *表现性*（滚动像素偏移，可归一化）与 **语义性**（窗口尺寸、图表可视范围，**必须保留并纳入 `analysis_config`**）。

另：Batch Export 有两条路径 —— **Favorite 路径**（建模时同步导出 KPI）与 **Workspace 路径**（导出整个工作区视图）。再次印证"GUI 编写模板 / 批量执行"的两段式架构。

### 5.5.5 🔴🔴 Area Binning —— 迄今最强的 T5 证据

`Chapter 4_Area Binning.pdf`（7 页）。分箱的**代表值**由 `Sampling Function` 决定，共 **9 种**：

| 公式 | 语义 |
|---|---|
| `Average` | 箱内均值，显示在**箱中心**（按时间） |
| `Medium` | 最接近均值的那个**实际数据点**，显示在**原位置** |
| `Median` | 中位数据，原位置 |
| `Maximum` / `Minimum` | 极值，原位置 |
| `First` / `Last` | 箱内时间最早 / 最晚的数据，原位置 |
| `Mode` | 众数，箱中心，按时间序 |
| `Apply to All` | 对所有已选参数套用同一公式 |

> 🔴 **同一参数、同一箱，9 种公式给出 9 个不同的"代表值"，而且连显示位置都不同**（箱中心 vs 原位置）。
> 并列关系需 tie-break：*"If there are identical values at more than one position, ... displayed at the **firstly founded** position."*

> 🔴🔴 **更关键的一条 —— 静默覆盖**：
> *"If you run binning for **Sector and PSC Type** parameters, **only Mode value is displayed whichever formula you select**."*
>
> **即：对某些参数类型，工具会静默忽略你选的聚合公式。**
> 这是一个**配置钉死也解决不了**的 T5 变体 —— 真值不仅取决于配置，还取决于**参数类型与配置的交互**，而且软件不提示。
> **论文里这句话可以直接引用**，它比"默认 1 秒平均"更有冲击力：*连显式指定的分析语义都可能被静默改写。*

**同时也是资产**：
- Area Binning Result 窗口内建 **Min / Max / Avg / Total Count / Threshold / More-Less Than Threshold / Ratio** → **又一个现成 oracle**，可导出 xls
- Area 定义可 **Export/Import 为 Excel** → 又一种 M6 skill 产物
- 分箱结果会作为新参数挂到 `Route Binning` / `Area Binning` 树下 → **工具会产生持久化的派生状态**（状态序列化必须覆盖）
- `Create Model`：Route Binning 的结果可**再生成一个新 Model** → 状态空间可递归增长
- ⚠️ 建箱需**在地图上拖拽**（Rectangle/Polygon/Circle 三种模式）→ M3 硬案例，且**依赖 Map 引擎授权**

### 5.5.6 其余三份的可用点

| 文档 | 要点 |
|---|---|
| **Add User Column** | Message Viewer 里按**字符串匹配**新增自定义列（Name + String）。→ 一种轻量的"用户自定义抽取"，可作为 L3 配置类任务；`Find Dialog` + 书签 + Find All 是 M1 要收录的 affordance |
| **Event Call Extract from Statistics Window** | 从 Call Statistics 里把 Drop/Fail/Pending 等 **Event Call 抽取成独立 drm 文件**，并自动把呼叫事件与信令存成 text 到 event 文件夹。→ ⭐ **现成的"故障案例切片"机制**，可直接用来从大 trace 里批量切出 L5 的 RCA 案例。依赖 XCAL Auto call 采集 |
| **Finding Missing Neighbors** | 条件：`Threshold of CINR/RSSI among Neighbor` + `Difference with Serving` + `Consecutive Time duration Satisfied`。→ **三参数可调的检测器**，是 L2/L3 任务的好模板，也可与我们的 detector battery 交叉校验 |
| **DSDS Feature** | 双卡场景有完整 KPI 定义：`SIM Transition Direction`、**`SIM Transition Latency`**、`MSO RSRP at Handin/Handout`、CM DSDS Summary（Total/MSO→MNO/MNO→MSO Handovers、HO Mean[Sec]）。→ 若数据涉及双卡，这是一整套现成的跨层任务；否则可跳过 |

---

## 5.7 ⭐⭐⭐ 安装目录分册精读（`Filtering` / `Multi Process` / `Selective Parsing` / `Usage of Favorite Info`）

> 来源:`Analyzer Help.chm` 抽出的 4 份 htm(CP949 解码)。§7 曾判定这四份"KB 里没有",结论是"随软件安装" —— **已证实,且 W01-A1 翻安装目录这条行动可以提前收获**。

### 5.7.1 🔴🔴 Favorite = XCAP 的原生 skill 格式,且**跨 Model 可迁移**

原文两句关键:

> *"Favorite configuration in an analysis type does not applied for the other analysis type."*
> *"**Favorite configuration is applied for the other Models.**"*

合起来定义了 Favorite 的类型签名:

```
Favorite : (参数集合 × 分析类型) → 可复用的窗口配置
           ├─ 跨 analysis type 不迁移(Graph 的 fav 不能用于 Map)
           └─ 跨 Model    可迁移(在数据集 A 上编写,在数据集 B 上复用)  ⭐
```

**这正是 M1 要找的"技能种子"的原生载体**,而且是我们此前只能推测的那条性质:*在一次分析中习得的东西能否作用于下一份数据*。官方明确说能。于是:

- **M1 的产物有了落地格式**:agent 探索出的有效分析组合 → 存为 `.fav` → 后续任务直接复用,零额外 GUI 操作。
- **"技能积累"可被量化**:第 k 个任务时 Favorite 库的规模 / 命中率 / 复用节省的动作数,是一条现成的曲线(E5 可直接用)。
- ⚠️ 但迁移性有**边界**:跨 analysis type 无效。所以 skill 库必须按 `(params, type)` 二元索引,不能只按参数名。这是一个真实的、非平凡的技能表征约束 —— 论文里是好材料。

**Recursive 选项 = XCAP 唯一的参数化/通配机制**:

> *"Recursive option enables you to open certain parameters (e.g. RSSI and BSIC) under different parameters categories (e.g. Scan Index#)."*

即 `RSSI @ Scan Index#*` —— 对参数树做一次 glob。**这是把具体 Favorite 提升为参数化技能的唯一原生手段**,直接对应 coding agent 里"把一次性脚本改写成带参数的函数"。M1 的技能泛化实验应当以"agent 是否会主动打开 Recursive"作为一个可观测指标。

### 5.7.2 ⭐⭐ 完整 `-EXPORT` 旧命令表(补全了 §1 的残缺)

| # | 命令 | 语义 | 备注 |
|---|---|---|---|
| 1 | `-EXPORT <drm dir> <csv path> <fav path>` | 建模后按 Favorite 批量导出 | 默认 **Separate + csv** |
| 2 | `-EXPORT ... <csv path>\<name>.csv` | 可指定输出文件名 | 🔴 **多个 log 时输出文件按序互相覆盖** |
| 3 | `-MB` | 导出 Message viewer(All msg) | |
| 4 | `-TXT` | 输出格式改 TXT | |
| 5 | `-all` | Sampling interval = All | 🔴 **不设时恒为 1 sec** |
| 6 | `-L3` / `-D` / `-H` | 仅 L3 / Detail Message / Hex Data | 对应 GUI 的两个勾选框 |
| 7 | `-mg` | 合并 log 后再导出 | **第 2 行覆盖问题的官方解法** |
| 8 | `-BX` | 在 XCAL 中执行 XCAP-C | 另一个二进制 `XCAP-C.exe` |
| 9 | `-?` | 打印帮助 | 例子写作 `"OPTis Analyzer.exe"` |

两个直接结论:

1. 🔴 **第 2 行是又一个"静默数据丢失"**:多 log 批量导出时后写的覆盖先写的,不报错。与 Area Binning 静默覆盖同类 —— **这类"沉默的破坏性默认值"已经攒到 3 个**(Area Binning 覆盖、多 log 覆盖、采样率默认 1s),足以在论文里单列一小节:*商业软件的失败模式不是报错,是安静地给你一个不同的答案*。这是"为什么 software-use 比 tool-use 难"的最好实证。
2. **`-all` 与默认 1s 再次确认 T5**:同一条 `-EXPORT`,加不加 `-all`,导出行数和任何时间窗聚合的结果都不同,而命令行看起来只差 5 个字符。

### 5.7.3 🔴 Multi Process —— 新威胁 T8 的来源

> *"Divides log file to 2 groups and then performs parsing by groups. After parsing, merges the created models."*
> *"**sub model concept is added**. If 2 DRM files are merged, two sub models are existed in subfolder of Model."*
> *"If the process is not operated for special time, it will be **decided to Failure** and processes next configured job."*

三个后果:

| 后果 | 说明 |
|---|---|
| **Model 的磁盘结构随开关而变** | 单进程 = 一个 Model;多进程 = Model 下含 sub-model 子目录。**任何直读 Model 目录的离线管线都会因这个开关而崩** |
| **静默降级** | 混合格式(`drm+scn`、`drm+drmw`)、acu/ecu、选了 `PAF`,都会**自动退回单进程** —— 不通知 |
| **超时即失败,继续下一个 job** | 无人值守批处理中**部分任务会静默缺失结果**,Scheduler 不停 |

> ✅ **不推翻 capacity=1**(§8 曾列为待查):这是**单实例内部的并行解析**,不是多实例并发。许可证瓶颈仍在。
> ⚠️ **但它必须被钉进环境规格**:`tools > options > General > Common` 的这个开关要写进 `analysis_config`,并在 W01 记录其默认值。

### 5.7.4 🔴🔴 观测面在**建模时**被冻结,不是查询时

Selective Parsing 的两句:

> *"Select **Log viewer** option before creating the model **to avoid creating Message viewer**"*(为了更快)
> *"**Reparsing** can be executed when further analysis is needed using Message viewer."*

即:**建模阶段的一个"提速"选项,会让后续所有消息级问题无法回答,除非重新解析(分钟级)。**

这是迄今最贴合 M4(检查点)论证的素材 —— 它不是"不可逆",而是**"可逆但代价高,且代价在做决定时不可见"**。对 agent 的含义:

- 一个只关心"快点建完模"的 agent 会在第一步就把自己的观测面砍掉,而这个损失要到第 5 步才暴露;
- 正确行为是**建模前先判断任务需不需要消息级证据** —— 这是一个真正的前瞻性决策,恰恰是当前 GUI agent 最缺的能力。
- **可直接设计成一个诊断性任务对**:同一份 log、同一个问题,一条要求 L3 消息证据、一条不要求。看 agent 会不会在两种情况下做出不同的建模选择。→ 建议列入 E3。

另注:**Selective Parsing 仅支持 Qualcomm LTE 参数**(*"currently only supported for Qualcomm LTE Parameter"*)。GT schema 里的 `selective_parsing` 字段因此**不是全局可用的**,5G-NR / 非高通 trace 上不成立 —— 任务生成器必须按 (chipset, RAT) 门控。

### 5.7.5 ⭐ Filtering 是**全局**的,且是结构化可序列化对象

> *"Filter settings are applied to **all analysis types**; Graph, Map, Table, CDF, PDF."*

Filter 的文法很干净:`(Parameter, Operator ∈ {=,<>,>,>=,<,<=}, Value, Linker ∈ {AND,OR,None})` 的有序列表,有名字,存在 Filter List 里。

| 影响 | 说明 |
|---|---|
| **M2 的类型化动作空间** | filter 不必用像素操作表达,可以是一个**声明式结构** —— 可生成、可校验、可 diff。这是 GUI 动作空间里少有的"像 API"的部分,应作为 typed action 的第一等公民 |
| 🔴 **全局性 = 跨任务污染** | 任务 A 设的 filter 会改变任务 B 里所有窗口的读数。`ckpt.workspace()` 里"清 filter"这一步**从保守设计升级为必需**,且**清除范围已确认为全局一次即可** |
| **T5 再添一条** | 同一个参数窗口,filter 不同 → 数值不同 → GT 不同。`analysis_config.filters: []` 必须显式声明为空,不能省略 |

> 💡 顺带给了一个 map 上的坑:*"the symbol of map data represents first letter of the filtered data file"* —— 地图符号会因 filter 而改变含义。视觉任务需注意。

---

## 5.8 ⭐⭐⭐ `pgDatabase Usage`(13 页,2026-09-02 到手)—— 关键路径上最后一份

> §7 曾列它为"唯一还挡在关键路径上的一份"。**已到手,离线 GT 管线的通路问题定案。**

### 5.8.1 它是什么:**建模时的一次性推送,不是查询接口**

流程原文:`File – New` → 选文件 → **Options** → `pgDatabase` 按钮 → 配置 → `Open` →
**"Postprocessing - pgDatabase will proceed."**

⇒ **pgDatabase 是 XCAP 在建模后处理阶段把数据 push 到外部 PostgreSQL,不是一个可随时读的 API。**

三条直接后果:

| 后果 | 说明 |
|---|---|
| ⭐ **许可证瓶颈可被绕开一大半** | **建模时占一次 license 把数据推进库,之后全部 GT 工作在 SQL 里做,不再碰 XCAP。** 这是 S0.2-1 的答案 |
| 🔴 **上传范围必须在建模前决定** | 与 `viewer_mode`(T8b)同类 —— **又一个"在建模那一刻冻结、事后要重来"的决定** |
| **配置位置与 Selective Parsing 是同一个 Options 对话框** | ⇒ `analysis_config` 要新增 `pg_upload` 与 `pg_db_option` 两组字段 |

### 5.8.2 🔴🔴 一个**真正不可逆**的破坏性操作,藏在 checkbox 里

`[Option]` 页有这一项(原文):

> *"If the UpdateTime of the FILEINFO_TABLE is **Null**, File Index will delete the corresponding number from the DB."*
> *"Checking the option **deletes all DB data** that has a File Index with UpdateTime 'Null'."*
> *"(This option is a **one-time option**. Default = Uncheck)"*

三个性质叠在一起,构成迄今**最危险的一个 affordance**:

| 性质 | 含义 |
|---|---|
| **勾一下就删库** | 一个 checkbox,删掉一整类 DB 记录 |
| **不可逆** | 删掉就没了,没有回收站 |
| ⭐ **"一次性"选项 —— 用完自动复位** | **事后从界面上看不出它被用过。** 状态与后果脱钩 |

⭐ **这比 DCI Analysis 那条更强。** DCI 是"结果丢失",这条是"**数据被删除,且没有任何痕迹**"。

**对 M4 的意义**:可逆性模型必须知道这类控件的存在,而**它与旁边那些无害的勾选框在视觉上毫无区别**。
一个在 Options 对话框里"探索一下都有什么选项"的 agent,勾中它就毁掉数据 ——
**而这正是 coding agent 里 `git checkout` 让你敢做的那种探索。**
⇒ **论文里直接引用这三句,作为"GUI 探索为什么不能免费"的最强实证。**

### 5.8.3 🔴 两个新的"沉默的破坏性默认值"(总数升至 5)

| # | 默认 | 后果 |
|---|---|---|
| 🆕 **4** | `[DB Option]` **默认只建 4 类表**:`5G-NR QC / 5G-NR SS / LTE QC / LTE SS` | **其它 chipset / RAT 的数据静默不上传** —— 库里就是没有,不报错 |
| 🆕 **5** | `Keep the created model in the Model Manager` **默认 = Uncheck** | **上传完 Model 就不保留** —— 拿到 DB 行,却丢了 Model |

⚠️ 第 5 条还带一个交互:原文注明 **"Single Process only"** ——
⇒ **`Multi Process` 开着时这个选项不生效**。T8a 与 pgDatabase 通路**互相影响**,
要同时拿到 Model 和 DB 行,**必须在单进程模式下建模**。

⇒ **§4《沉默的破坏性默认值》那张表现在有 5 行**,来源覆盖 4 份互相独立的官方文档。

### 5.8.4 ⭐ `FILEINFO_TABLE` 的 Area 1/2/3 = 现成的会话标记位

> *"Area 1, 2 and 3 … The values recorded in Area 1, 2, 3 are recorded in Area 1, 2, and 3 of the File Info Table in DB."*

**三个用户自定义字段,随每个文件写进库。** 直接用途:

```
Area1 = session_id     (如 20260915-lab-A-s03)
Area2 = condition      (sham | A_only | B_only | A_plus_B)
Area3 = pair_id / seq
```

⇒ **注入元数据可以在建模时就写进数据库,与测量数据同表关联。**
GT 管线因此可以纯 SQL 地把"注入日志"和"测量结果"join 起来,不必额外维护映射文件。
**这条要写进 [[DATA-COLLECTION-SPEC]] 的归档规范。**

### 5.8.5 落地要求(A6③ 的具体步骤)

| 项 | 内容 |
|---|---|
| **服务端** | **XCAP 不自带 PostgreSQL**(A6 已证:安装目录零个 `postgres*`,只有 `libpq.dll`),需自建。文档路径写的是 `C:\Program Files\PostgreSQL\**10**\data` ⇒ 官方验证过的是 **PG 10** |
| **`pg_hba.conf`** | 需设 `host  all  all  all  md5`。⚠️ **文档全篇把文件名拼成 `pg_hpa.conf`**(真实文件名是 `pg_hba.conf`)—— 又一处文档与现实不符 |
| **防火墙** | Windows Defender 入站规则,TCP **5432** |
| **失败重传** | 模型右键 → `Re Upload Postgres DB`;`Info` 状态变 `Complete` |
| ⭐ **可校验的后置条件** | **`Info` 状态字段可直接作为 M3 的 postcondition** —— 上传成不成功有明确信号,不必猜。<br>⚠️ 但"需要重传功能"本身说明**上传会失败**,执行器必须检查该状态 |

---

## 5.9 ⭐⭐⭐ 真机实测所得(W01 台账 v2,2026-09-03/05)

> 前面 §5.5–§5.8 是**读文档**得到的;这一节是**在真机与全库数据上跑出来的**,
> 其中三条**推翻或限定了文档给出的图景**。

### 5.9.1 🔴🔴 XCAP 的组织轴是 **RAT × 功能块**,不是协议七层

pgDatabase 的 **308 张表**逐张归类:

| 主轴 = RAT | 表数 | | 次轴 = 功能块 | 表数 |
|---|---:|---|---|---:|
| LTE | 141 | | KPI | 134 |
| 5G-NR | 46 | | RF | 77 |
| 5G | 23 | | **Layer1** | 41 |
| WCDMA | 17 | | Tput | 30 |
| GSM | 14 | | Signaling | 19 |
| | | | 🔴 **`Layer2`** | **0** |

⇒ **XCAP 的组织轴里根本没有 MAC / RLC / PDCP / NAS,连 `Layer2` 都一张表都没有。**

**后果(重要)**:

| # | 后果 |
|---|---|
| 1 | 🔴 **"按表名分层"是把一个轴投影到另一个轴,投影没有唯一解** —— 换一套正则,RF/PHY 从 **107 张变 435 张**(4 倍),NAS 从 10 变 **0** |
| 2 | 🔴 **论文里不得单独出现"某层多少表 / 多少记录"**,除非同时给出投影规则 |
| 3 | ✅ 但**"≥3 层非空"这个判定是稳健的**(怎么投影都 ≥6 层)⇒ 跨层任务成立 |
| 4 | ⭐ 更诚实的写法:用 **XCAP 自己的轴**描述数据组织,只在对齐协议栈时做一次**显式声明的投影** |
| 5 | 🔴 **N1 混淆矩阵的"层"需要重新定义** —— 见 [[MASTER-STATUS]] 风险登记 |

⚠️ 试过拿 pgDatabase 的 308 张表当权威字典 —— **行不通,308 张里只有 1 张能匹上 Model 里的表名**。

### 5.9.2 🔴🔴 CLI `-CM` 建的 Model **缺一整类后处理产物** —— T8 的新实例

| 产物 | GUI 全流程建的 Model | CLI `-CM` 建的 Model |
|---|---:|---:|
| `MSG_FILTER_L*.dat` | **337 张** | **2 张** |
| `xCOMMON_BMF_*.dat` | 86 / 78 张 | **0 张** |
| **`xCOMMON_L3_Failure_Reason`** | ✅ **132 条** | 🔴 **不存在** |

⇒ **不是数据的差别,是建模方式的差别。** 三个代价:

1. 做不了逐 code 对账 ⇒ 全量验证只在 GUI 建的那条 trace 上成立
2. 没有 BMF 镜像表 ⇒ L2 的映射方法在新数据上要跑全库
3. 🔴 **没有 XCAP 自带的根因标注**

⭐ **这是 T8(建模时冻结观测面)的第二个实例,而且比 `viewer_mode` 那条更隐蔽** ——
`viewer_mode` 至少是个可见的勾选框,而**"用 CLI 还是 GUI 建模"看起来完全是个效率选择**。

### 5.9.3 ⭐⭐ `xCOMMON_L3_Failure_Reason` —— XCAP **自带的根因标注**

**7 类原因码**,132 条。形态上最接近 L5b 需要的 root-cause label。

⇒ **可作 T-13 专家试标的预标注模板**,大幅降低专家工作量。
⚠️ **前置**:它只在 **GUI 全流程**建的 Model 里存在(见 §5.9.2)。

### 5.9.4 ⚠️ `xCOMMON_BMF_*` 是导出 CSV 的**后台镜像**,不是独立通路

做双通路交叉验证时,若把 `xCOMMON_BMF_*` 当"第二通路",**与通路 A 同源,是自欺**。
排除后剩 **18 个参数**有真正独立的第二通路,逐点 **100% 一致**。

⭐ **一条可推广的方法论**:*交叉验证前必须先证明两条通路真的独立。*

### 5.9.5 🔴 列名里有**字面换行符**

```
"LTE KPI\nPCell Serving PCI"
```

⇒ 早先"按名匹配全部落空"的根因就是它。**任何按参数名匹配的 oracle 都必须处理列名内的换行。**
(与 §5.7 的 `Parameter Caption Short/Long` 开关叠加,按名匹配的脆弱性再加一层。)

### 5.9.6 ⭐ P32:**展开树节点会顺带选中它,收起不还原**

序列化器五条判据里:稳定 ✅ / 敏感 10/10 ✅ / 精确 10/10 ✅ / 还原 10/10 ✅ /
**全快照回基线 🔴 0/10**。

根因不是序列化器,是 XCAP 的真实副作用:**对展开状态幂等,对选中项不幂等。**

⭐ **如果序列化器只记"主语义",这个副作用永远不会被发现** —— 是那条更严的判据把它抓出来的。
⇒ 直接影响检查点设计(G0.1 / G0.4):`ckpt.workspace()` 必须显式复位选中项。

### 5.9.7 CLI 行为的三条实测补充

| 发现 | 说明 |
|---|---|
| 🔴 **`-?` 的解释已撤回(09-09)** | ~~"`-?` 无效,因为文档例子写的是 `OPTis Analyzer.exe`"~~ **证否**。`XCAP-M.exe -?` **有专用分支**:弹模态框(标题 `XCAP-M`/警告图标/单 OK),正文只有一行 `-EXPORT` 用法;stdout 仅 **44 B**;点 OK 后进程退出、主界面从不显示。三路参数(`-?` / 不存在的开关 / 缺参数的真开关)**行为互不相同** ⇒ 不是被通用报错兜底。<br>⭐ **观测对、解释错**。根因:**一次测 stdout、一次测 GUI,两份记录都不说明自己测的是哪个通道** |
| ⭐ **stdout 有完整状态机** | `Start XCAP → Initailization → LogParsing → SaveResult → Postprocessing → Exporting → Terminate XCAP`(厂商原文拼写)。**末行可当完成判据**,不必轮询文件大小 |
| ⭐ **`CLI_Log\<时间戳>.txt`** | 每次调用新建一份,内容是完整命令行 ⇒ **现成的调用审计日志** |
| **建模耗时(真实体量)** | 65.5 MB → **20.2 s**;400.6 MB → **33.4 s** |
| 🔴🔴 **`.fav` 形态的真相(09-09,A 级证据)** | ~~"`.fav` 那条调用形态卡死"~~ **推翻**。七种形态各跑一次:r1 = 1,246,002 B、r2(两 log)≈ 2×r1、**r3–r7 = 0 B**,进程正常退出、Model 被重建 ⇒ **跑完了但导出零行**。<br>🔴 **受控复现 r8 更严重**:带 `.fav` 的 `-EXPORT` **即使导出零行、连输出文件都不创建,仍会重建并覆盖 `Model\<输入名>_M<n>`**(运行前后目录与内部每个文件 mtime 全变),**全程无弹框,`CLI_Log` 只写 `Start -`**。命名确定性 ⇒ 同一输入**必然撞名** ⇒ **覆盖是必然,不是偶发**。<br>⭐⭐ **三个可观察通道(退出码 / 输出文件 / 日志)没有一个说实话,真相在第四个地方(Model 目录的 mtime)** ⇒ **这软件上没有任何一个便宜的、可观察的成败判据** |
| 🔴 **多 log 导出"互相覆盖"被推翻(09-09)** | CLI + 文件夹 + `-MB` 时**两个 log 都进同一个 CSV,没有任何东西被覆盖**(r2 ≈ 2×r1)。⚠️ 原结论来自手册表述,**未实测** |
| ⭐ **`CLI_Log` 的语义是三分的,不是二分** | 参数被接受 → `Start - <完整命令行>` ｜ 参数预检失败 → **只写错误码,不含命令行** ｜ 未识别开关 / `-?` → **完全不写**。<br>⇒ ⭐ **不是"什么都分不出",是"选择性沉默"** —— 软件**分得出一种失败,分不出另一种**,而失败那份**永远不知道是哪条命令**。<br>⭐ 且 **`CLI_Log` 不保留引号** ⇒ 日志无法区分"一个含空格的路径"与"多个参数" |
| **`-EXPORT … -MB` 逐字节可复现** | 三次 SHA-256 全等(38,853,169 B / 62–65 s)<br>⚠️ **只覆盖这一条形态;`.fav` 形态卡死无法测,而 L2 的 oracle 正是用 `.fav` 导出的 CSV** ⇒ **不可外推** |

### 5.9.9 🔴🔴 **不存在"只读地跑一次 XCAP"**(09-09 快照实测)

连拍 4 次安装目录 + Settings 目录快照,**即使跑的是 `-zzzz`(不存在的开关)**:

| 位置 | 被改写的内容 |
|---|---|
| `Program Files\…\XCAP-M\` | `CONFIG` / `reg` / `sym` / `ver`、`MapXtremeDesktop.lic`、**外加一个每次换名字的 949 B DLL `LibLogx<8位十六进制>.dll`** |
| `~\Documents\Accuver\XCAP-M\Settings\` | **约 20 个文件 / >1.6 MB** |

⚠️ **写入分两阶段**:启动时写那 20 个,**退出时**才写 `EtcOption.ii` 等 8 个。

⇒ 🔴 **"环境有没有变"不能作为 checkpoint 判据** —— 它恒为"变了"。
**M4 的检查点必须基于语义状态,不能基于文件系统 diff。**

### 5.9.10 ⭐ 上限类文案:**参数化 10 : 写死 24**(A7′ 全库普查)

对安装目录全部 `*Resource*.ini` 做互补关键词普查,只数"容量/上限"类:**写死占 70.6%**。

⭐ **比比例更要紧的是这组对照**:

| 文案 | 写法 | 用户能改吗 |
|---|---|---|
| 能开几个**窗口**(`0575`/`0642`)、几个 **Map 窗口**(`2136`)、选几个**模型**(`2228`) | 全是 `%d` | ❌ **一个都改不了** |
| 能开几个**实例**(`0010`) | **写死 `two`** | ✅ **唯一能改的那条** |

⇒ ⭐⭐ **从文案本身看不出你读到的数字是活的还是死的,而这软件里两种都有,比例约 3:7。**

顺带:「模型数量」上限**三处文案、三种写法、两个数**(`2228` `%d` / `DRMAPCOMMCLS_0241` 写死 20 / `1476` 写死 14)。

### 5.9.11 🆕 第三种 record_type `0a 14 07`

合成 `.drm` 走链时发现 `drm_scan.scan()` 的分支**没覆盖到** `0a 14 07`,而它带 **73 个 log code** ——
**比 `14 01 01` 的 59 个还多**。

⇒ 🔴 **现有所有 log code 覆盖率数字全部偏低**(T-14 的判定不变,但计数要重算)。
⬜ 其布局未解。

### 5.9.12 ⭐ 时间戳公式在**写方向**被被测软件确认

合成 `.drm` 时把每条记录的**内外两个 QC 时间戳各 +1.25 ms**,
再用 XCAP 自己的 `-EXPORT -MB` 导出读回:`.207464 → .208714 → .209964 …` **逐条差 1.25 ms,分毫不差**。

⇒ ⭐ **新的证据类型:被测软件自己确认了我们的公式。** v2 只验过读方向。

---

### 5.9.8 授权与文档的两个空洞

| 项 | 说明 |
|---|---|
| **授权 Call Type** | 有 `Youtube`,**没有通用 Video Streaming,更没有 WebRTC** ⇒ XCAP 侧 App 层表几乎全空,**App 层真值必须来自外部日志** |
| 🔴 **`Delay` 列的定义** | **在任何文档里都找不到** ⇒ **定下来之前不能当 oracle** |
| ⬜ 仍缺 | `Tools_Analysis`;`User Defined KPI Parameter`(可能解释 `[Sum]`/`[Avg]`/`[Mode]` 后缀 = A4 的遗留) |

---

## 6.5 ⭐⭐ Release Note 富矿已确认（KB 内可直接取）

搜索 `analyzer release note` / `release note` 找到密集的版本记录：

**XCAP (Analyzer) 线**：`20250828 (O5.33.7.18)` · `20250724 (O5.33.6.17)` · `20250430 (O5.33.3.20)` · `20250327 (O5.33.2.17)` · `11292024 (05.32.10.19)` · `11122024 (D5.32.10.6)` · `09302024 (05.32.8.17)`

**XCAL5 线**：`20250929 (Rev55, O5.0.50.0)` · `20250707 (Rev52, O5.0.44.0)` · `20250512` · `20250407` · `20250304` · `02102025` · `01132025` · `12022024` · `10282024` · `10072024`

> **一年内 7+ 个 XCAP 版本，且每个都有 release note。**
> → **M6 的漂移修复实验可以用真实版本演化**，漂移类别（Cosmetic / Layout / Naming / Structural / Semantic）可以从 release note 里**读出来并分类**，而不是人工注入。
> → 这比在 Wireshark 上合成漂移强得多，而且就在 XCAP 上做。
> **W01 追加任务**：把 XCAP 线的 release note 全部取回，按漂移类别编码，看能否构造出一条真实的 UI 演化时间线。

---

## 7. 尚未取回的文档（KB 里没有，需另找）

搜索确认：**`Multi Process` / `pgDatabase Usage` / `Selective Parsing` / `Filtering` / `Usage of Favorite Info` 在 KB 中没有独立文章**，只在 `Detailed Guides` 的话题列表里被提及，但**没有对应附件**。

> ✅ **2026-09-01 更新：五份中的四份已到手**（从 `Analyzer Help.chm` 抽出，见 `xcap_docs/*.md`）：
> `Filtering` · `Multi Process` · `Selective Parsing` · `Usage of Favorite Info` —— 精读结论见 **§5.7**（其中两条改变了设计：Favorite 跨 Model 可迁移 → M1 的技能格式；观测面在建模时冻结 → 新威胁 T8）。
> ~~**仍缺：`pgDatabase Usage`**~~ → ✅ **2026-09-02 到手**（`Chapter 4_pgDatabase Usage.pdf`，13 页），见 **§5.8**。
> **关键路径上已无缺失文档。**

> 💡 **它们很可能随软件一起安装。** User Guide 全文反复用 *"that comes with it"* 指代这些分册 —— 意味着 PDF 就在 **XCAP 安装目录**里。
> **W01 行动**：装好 XCAP 后**第一件事是翻安装目录找文档文件夹**，比走 portal 快得多。这条已加进 A1。

~~其中最关键的仍是 `Usage of Favorite Info`~~ → **已读**。它没有给出 `.fav` 的**文件格式**（M6 程序化生成的问题仍未答，需 W01-X3 直接十六进制/文本查看 `.fav`），但给出了比格式更重要的东西：**Favorite 的语义与迁移边界**（§5.7.1）。

---

## 7.5 已取回的文档（`xcap_docs/`，17 份）

`File Export using Command Prompt` · `Exporting Batch using Batch Export Feature` · `Model Creation Plan (Scheduler)` · `LTE DCI Analysis` · `LTE RB Scheduling Map Usage` · `Message Duration Time Analysis` · `Area Binning` · `Add User Column Usage` · `Event Call Extract from Statistics Window` · `Finding Missing Neighbors` · `DSDS Feature` · `Compare before and after Delta Map Optimization` · `Auto Reporting with Map Legend change` · `Create Virtual GPS Function` · `Editing GSP Coordination` · `Map Display of Inbuilding Data` · `LTE PDSCH Throughput Calculation`

---

## 8. 尚未取回、但已确认存在的文档

| 文档 | 用途 |
|---|---|
| ~~`Usage of Favorite Info`~~ | ✅ 已读 —— 语义拿到，**文件格式仍未知**（转 W01-X3） |
| ~~`Multi Process`~~ | ✅ 已读 —— **不推翻 capacity=1**（单实例内并行解析）；但引入 T8，见 §5.7.3 |
| ~~`pgDatabase Usage`~~ | ✅ 已读 —— **建模时一次性 push，不是查询接口**；含一个不可逆删库 checkbox（§5.8） |
| ~~`Selective Parsing`~~ | ✅ 已读 —— 观测面在建模时冻结（§5.7.4）；仅支持 Qualcomm LTE |
| ~~`Filtering`~~ | ✅ 已读 —— 全局、结构化、可序列化（§5.7.5） |
| `User Defined KPI Parameter` | 自定义指标 |
| `Main Window Tools Setting Options` | T5 配置项全集 |
| `Table Mouse Menu` / `Tree Mouse Menu` | M1 affordance 清单 |
| `5G-NR Slot per Average Function` | 聚合语义 |
| `Quick Guide to enable per slot KPI by default` | 默认配置改变读数 |
| `Export Voice Call Statistics with CLI Command` | CLI 用例 |
| `LTE PDSCH Throughput Calculation`（PDF，已下载） | T5 |

多数在 `[XCAP-M] Detailed Guides` 一文的附件里（~20 个 Chapter 4 PDF）。

---

## 8. 对计划的影响（需回写）

| 影响 | 说明 |
|---|---|
| **U6 已答** | CLI 完备，W01 的 A6 从"探索"变为"验证语法 + 测批量导出的路径限制" |
| **M6 论证强度大幅提升** | 从"我们的提议"变为"厂商既有架构"。§5 要重写这一节 |
| **T5 有官方解药** | S0.1-6 采样标定从反向工程变为照文档实现；5G-NR PDSCH 推导链可直接作为论文例证 |
| **T7 被官方证实** | DL Dropped PDUs = 计数器 1 秒差分 |
| **L1–L4 的 oracle 更便宜** | CLI + UDR 阈值判定 → 大批任务可自动验证，不占 GUI |
| **E0.0 有了具体检查项** | CSV 导入的前 4 列硬性要求（Day/Time Stamp/Lat/Lon） |
| **漂移实验可用真实版本** | Release notes + 9 个历史版本 |
| **M2 必要性增强** | 单个 BMT 报告即 1727 个 KPI，参数空间极大 |
| ⭐ **M1 有了原生技能格式** | Favorite 跨 Model 可迁移、跨 analysis type 不迁移 → 技能库按 `(params, type)` 二元索引；Recursive 是唯一的参数化手段 |
| ⭐ **M2 的第一等 typed action** | Filtering 是声明式四元组列表，不必走像素 → 类型化动作空间有了真实的锚点 |
| 🔴 **新威胁 T8** | Multi Process 改变 Model 磁盘结构 + 静默降级 + 超时静默失败 → 必须钉进环境规格 |
| 🔴 **M4 拿到最好的素材** | Log-viewer-only 建模会砍掉消息级观测面，代价在决策时不可见、在 5 步后才暴露 → E3 的诊断任务对 |
| ⭐ **"沉默的破坏性默认值"凑够 5 个** | Area Binning 覆盖 / 多 log 导出覆盖 / 采样率默认 1s / **DB Option 默认只建 4 类表** / **上传后默认不保留 Model** → 论文 §4 单列一节 |
| 🔴🔴 **M4 拿到最强实证** | pgDatabase 的"一次性删库 checkbox" —— 不可逆、无痕迹、用完自动复位，与无害勾选框视觉上无区别（§5.8.2） |
| ⭐ **许可证瓶颈可绕开** | 建模时 push 进 PG，之后 GT 全部走 SQL，不再碰 XCAP（§5.8.1） |
| ⭐ **注入元数据有了原生落点** | `FILEINFO_TABLE` 的 Area 1/2/3 三个自定义字段（§5.8.4） |
| 🔴🔴 **"层"不是 XCAP 的一等公民** | RAT × 功能块才是;`Layer2` 零表 → N1 的层定义需重做（§5.9.1） |
| 🔴 **T8 第二实例** | CLI `-CM` vs GUI 建模，产物差一整类（§5.9.2） |
| ⭐ **D3 有了预标注来源** | `xCOMMON_L3_Failure_Reason` 7 类原因码（§5.9.3） |
| ⭐ **检查点必须复位选中项** | P32：展开幂等、选中不幂等（§5.9.6） |

> ⚠️ **一个新的张力**：CLI 越强，"为什么 agent 需要 GUI"的质疑越尖锐。
> 但注意 —— **CLI 需要先有 `.fav` / `.mes` / UDR 模板，而模板只能在 GUI 里创建**。
> 这恰恰是最好的回答：**GUI 是"编写分析意图"的地方，CLI 是执行的地方。agent 的价值在前者。**
> 这条要写进 [[PAPER-STRUCTURE-AND-STORYLINE]] 对"为什么要开 GUI"的回答里 —— 比原先的实用论据强得多。
