---
tags: [sigcomm, softwarex, benchmark, architecture]
status: v1
created: 2026-09-04
---

# Benchmark 结构规格(Benchmark Design)v1

> **这份文档描述"我们要造的东西本身"** —— 组件、数据类型、任务类型、工具边界、仓库结构、接口契约。
> 其它文档回答"为什么"和"怎么做";**这份回答"它长什么样"**。
> 变更此文件 = 变更产物结构,须同步 [[MASTER-STATUS]] §5。

---

## 0. 命名(⬜ 未定)

| 层级 | 现用 | 备注 |
|---|---|---|
| 项目代号 | `SoftwareX` | 内部用,不进论文 |
| **Benchmark 名** | ⬜ **待定** | 🔴 **不得含 "RCA"** —— 任务阶梯 L1–L4 与 RCA 无关,叫 RCA 会自我窄化 |
| Agent 系统名 | ⬜ 待定 | |

候选方向:强调"专业软件 + 不可见的正确性",而非强调故障诊断。

---

## 1. 产物由三件事组成 —— **必须可分离**

```
┌─────────────────────────────────────────────────────────────┐
│  ① SUBSTRATE   环境层:把商业 GUI 软件变成可编程、可检查点的环境  │
│      ├─ 状态序列化(可折叠层级树)                              │
│      ├─ typed action + state diff                            │
│      └─ 三级检查点控制器                                       │
├─────────────────────────────────────────────────────────────┤
│  ② BENCH       任务层:任务 + oracle + GT + rubric             │
│      └─ 与 ①③ 完全解耦,可被任何 agent 消费                     │
├─────────────────────────────────────────────────────────────┤
│  ③ AGENT       我们的系统:M1–M6                                │
│      └─ 只通过 ① 的接口访问软件,不得走后门                      │
└─────────────────────────────────────────────────────────────┘
```

🔴 **可分离是硬要求。** 论文同时主张 agent 设计与 benchmark;
若三者纠缠,审稿人无法判断"测的到底是什么",两个贡献一起垮。

**判据**:`bench/` 必须能在**不 import `agent/`** 的情况下跑完一次完整评测。CI 强制。

---

## 2. 数据类型(Dataset Taxonomy)

### 2.1 三个正交的轴

| 轴            | 取值                                                                                                 |
| ------------ | -------------------------------------------------------------------------------------------------- |
| **A. 来源**    | `real-natural`(真实采集,无注入)· `real-injected`(受控注入)· `mutated`(真实母本 + 容器层变异)· `public`(Wireshark 公开语料) |
| **B. 标签完备性** | `T1` 单故障、完备 · `T2` 组合、完备 · `T3` 真实混合、**构造上不完备** · `n/a`(L1–L4 不需要故障标签)                             |
| **C. 划分**    | `dev` · `test` · `heldout`                                                                         |

### 2.2 数据集登记

| ID | 来源 | 标签 | 现状 | 用途 |
|---|---|---|---|---|
| `DS-PING-V111` | real-natural | T3 / n/a | ✅ **15 条,`Ver=1.11`**,15.3 GB | L1–L4 主力 · (c) 变异母本 · T3 素材 |
| `DS-LEGACY-V105` | real-natural | — | ⚠️ **容器模型未验证** | 🔴 验证前不得使用(O6) |
| `DS-STREAM` | real-natural | T3 | ✅ **09-03 到货**,65.5 MB→20.2 s 建模。⚠️ **仅 398 秒、90% 空闲** | App 层 · N3(**需补采 ≥15 min**) |
| `DS-TELEOP` | real-natural | T3 | ✅ **09-03 到货**,400.6 MB→33.4 s。1,776 秒,**9 类现象达标** | App 层 · N3 |
| `DS-EXT-APP` 🆕 | real-natural | n/a | ✅ **App 层真值的实际来源**:WebRTC `inbound-rtp` 43 字段 / 813 个 `.ivf` 逐帧 / `received_metrics` 81,605 行 | **N3 的 App 层证据**(XCAP 侧授权无 WebRTC) |
| `DS-INJ-*` | real-injected | **T1 / T2** | ⬜ 待 D-1 | **L5b · N1 · N3 的唯一来源** |
| `DS-MUT-*` | mutated | T1 | ⬜ | 包级故障的规模化 |
| `DS-WS-*` | public | n/a | ⬜ | ⭐ 可复现镜像 · agent 主开发环境 |

### 2.3 🔴 划分纪律

| # | 规则 |
|---|---|
| 1 | **同一条 trace 不得同时出现在 dev 与 test** |
| 2 | **heldout = 异日 / 异机 / 异地采集**,采完封存,**开发期不得查看** |
| 3 | 变异产物与其母本**必须同划分**(否则母本泄漏) |
| 4 | 每条数据记 `Ver=` / ICD / XCAL 版本;**跨版本不得混入同一对照组** |
| 5 | ⚠️ **All 档与 1Sec 档的同一份数据算同一条 trace**,不得当作两条 |

### 2.4 磁盘格式

| 格式 | 是什么 | 谁读 |
|---|---|---|
| `.drm` / `.drx` | 采集原始容器 | 我们的 parser(283 MB/s)· XCAP |
| `.drmp` / `.drmmp` / `.drmw` | 副档(**T-07 未定是否必需**) | XCAP |
| Model 目录 / `.dat`(RSF) | XCAP 解析产物 | 我们的 RSF 读表器(布局解出率 100%) |
| PostgreSQL | pgDatabase 建模时 push | ⭐ **GT 管线主力,不占 license** |
| 导出 CSV / TXT / XLS | CLI `-EXPORT` / `FEP` / `MEP` | L4 oracle |
| `.pcap` | `ConvertToPCAP` 产物 | Wireshark 侧配对任务 |

---

## 3. 任务类型(Task Taxonomy)

### 3.1 六档

| 档 | 名称 | 答案 | Oracle 类型 | 需故障标签 | 现有数据够? |
|---|---|---|---|---|---|
| **L1** | 检索定位 | 唯一 | 状态谓词 / 离线解析 | ❌ | ✅ |
| **L2** | 计算测量 | 唯一(**给定配置**) | 精确值 | ❌ | ✅ |
| **L3** | 配置构建 | 唯一状态 | 状态谓词 | ❌ | ✅ **不依赖数据质量** |
| **L4** | 对比 | 唯一 | 导出文件比对 | ❌ | ✅ |
| **L5a** 🆕 | **证据链构造** | 证据集合 | **证据可核验性** | ❌ ⭐ | 🟡 **D2 门:只差"稳定共现"定义** |
| **L5b** | 根因归因 | **多重** | rubric + 可核验证据 | ✅ | 🟡 **D3 门:只差 T-13 专家试标** |

⭐ **L5a 是本设计的关键分层**:它考跨层证据的发现与对齐(**software-use 能力**),
不考根因判定(**领域推理能力**),因而**不需要故障标签**,可从现有数据生成。
它把"L5 做不了"从全有全无变成分级降级。

### 3.2 横切类型

| 类型 | 说明 |
|---|---|
| **XCAP ↔ Wireshark 配对** | 同一问题两个软件各问一遍。**差值 = 软件操作能力;共同部分 = 领域推理能力**。这是把两种能力分开的唯一手段 |
| **T5 配置陷阱题** | 答案随 `analysis_config` 改变。**只报数不声明配置 = Partial** |
| **P5 自述不可信探针** | 软件自己给出错误答案。⭐ **不需要 XCAP 授权,只需截图** |
| **P6 不可逆性探针** | 观察是否触发破坏性/信息损失操作 |
| **P7 质疑题面** | 题面本身有错或描述了不存在的现象 |

### 3.3 任务记录格式

```
bench/tasks/<suite>/<task_id>.yaml
```

```yaml
task_id: STREAM-COV-007
suite: xcap-l5b
level: L5b                        # L1|L2|L3|L4|L5a|L5b|P5|P6|P7
dataset: DS-INJ-20260915-s03
split: test
tier: T1                          # T1|T2|T3|n/a

prompt: |                         # 呈现给 agent 的原文
  ...
answer_scope: "归因到 layer + mechanism"
answer_form: single_attribution   # single_attribution | ranked_factors | value | state | evidence_set

environment:                      # 见 §5.1,逐字钉死
  analysis_config: {...}
  xcap_version: "5.33.0.15"
  action_surface: "v3"

oracle:
  type: predicate                 # predicate|exact|file_diff|rubric|evidence
  ref: oracles/chk_rsrp.py
  verified: true                  # 🔴 false 者不得计入正式评测
  verified_at: 2026-09-02

gt: gt/STREAM-COV-007.yaml        # L5 才有;schema 见 TASK-GT-GENERATION-PLAN §2.1
cost_estimate: {live_actions: 25, wallclock_s: 180}
```

🔴 **`oracle.verified` 是硬门**:22 道现有题里 **11 道通过 / 3 道硬伤 / 2 道口径过时 / 4 道待复核**。
**只有 `verified: true` 的进正式评测。**

---

## 4. 软件与工具边界

### 4.1 被测软件(Software Under Test)

| 软件 | 角色 | 约束 |
|---|---|---|
| **XCAP-M 5.33.0.15** | **主被测对象** | 🔴 **实例上限待封口** —— ~~写死 = 2~~ 09-09 被推翻(未勾 Multi Process 时**三实例并存**,勾上反而被拦)。**规划按 1**;两代 Wireshark 插件并存;皮肤(Classic/Ribbon)**暴露不同功能集**,须固定 |
| **Wireshark** | ⭐ agent 主开发环境 + 公开可复现镜像 | 免费、无上限 → M1–M6 在此迭代,XCAP 定期验证。**同时构成跨软件泛化证据** |
| PostgreSQL 10 | pgDatabase 的外部服务端 | XCAP **不自带**,需自建 |
| XCAL | **采集端**(不是分析端) | 决定 logging mask / 抓包 / 副档 |

### 4.2 我们要造的工具

| 工具 | 状态 | 说明 |
|---|---|---|
| `diag_parser` | ✅ | `.drm` 容器全解,283 MB/s,零重同步 |
| `rsf_reader` | ✅ | `.dat` 布局解出率 100%(9,581 文件)。⚠️ 语义仅验过 0.013% 字段值 |
| `detector_battery` | 🔵 | 24 条现象 → T-01 全库扫描 |
| `injection_controller` | ⬜ | 驱动衰减器/netem + 打时间戳 + 写 `injection_log.yaml` |
| `state_serializer` | ⬜ | W02-S1。⚠️ **窗口几何是语义相关的,不可归一化** |
| `checkpoint_controller` | ⬜ | 三级:workspace / model / rebuild(CLI 原语 `INI`/`KX`/`MOP`/`MDEL`) |
| `recorder` | ⬜ | 轨迹录制,replay 语料 |

### 4.3 🔴 给 agent 的动作面(与上面两类严格区分)

**只有这些,描述中不得出现领域词汇**(RRC / 切换 / 分箱 / Favorite / 采样…):

```
screenshot()   read_tree()   read_window_list()   click(target)   drag(from,to)
set_field(target,value)   type(text)   key(combo)   scroll(target,delta)
export(view,path)   read_file(path)
```

⭐ **动作表就是我们声称不存在的那份软件手册。** 预置领域动作 = 把 M1/M6 的答案塞进环境。
每次 run 的动作表按 `manifest/action_surface_v<n>.json` **版本化落盘**。

⭐ **M6 的自撰技能以新动作的形式注册** → 动作面增长曲线 = 技能获取的可观测量。
同一技能的另一投影是 `.fav`(宿主原生格式,跨 Model 可迁移)。

---

## 5. 接口契约

### 5.1 Environment API

```python
class Environment:
    def reset(self, level: Literal["workspace","model","rebuild"]) -> State: ...
    def act(self, action: Action) -> ActionResult: ...   # 必含 state_diff
    def observe(self, path: str | None = None) -> StateTree: ...  # 惰性,按需展开
    def snapshot(self) -> CheckpointId: ...
    def restore(self, ckpt: CheckpointId) -> None: ...
```

🔴 **`ActionResult` 必须携带 `state_diff`,不得只回 `ok`** —— 静默失败检测(M3)依赖它,
而任何 RPC/MCP 层都看不见 GUI 状态是否真的变了。

### 5.2 Oracle API

```python
def verify(answer: Answer, ctx: TaskContext) -> Verdict:
    # Verdict ∈ Full | Full+ | Partial | Hedged | Wrong |
    #           Confident-Wrong | Config-mismatch | Review
```

### 5.2.5 🆕 两种评测协议 —— **RSI 用现行协议测不出来**

| 协议 | 呈现方式 | 记忆 | 测什么 |
|---|---|---|---|
| **i.i.d.**(现行) | 任务独立,每题 ≥3 seed | **每题清空** | 单任务能力。B0–B6 的横向消融 |
| 🆕 **career**(纵向) | **固定顺序连续呈现** | **跨任务持续** | ⭐ **复利效应**:第 k 个任务的边际成本 vs k |

🔴 **career 模式的三条纪律**:
1. **学过的任务不得再用于测量**(污染)⇒ `suites.yaml` 必须区分 `career_train` / `career_probe` / `transfer_holdout`
2. **顺序是实验变量** ⇒ 至少两种顺序,看结论稳不稳
3. **对照组 = 同一模型关掉持久化**,不是另一个 baseline

⇒ **`bench/suites.yaml` 需增 `protocol: iid | career` 字段**;
`results/` 需记录任务呈现顺序与记忆状态快照。详见 [[AGENT-DESIGN]] §2.7.5。

### 5.3 Result 记录

```
results/<run_id>/
  manifest/{action_surface.json, env.yaml, preconditions.md}
  scores.csv           # task_id × seed × condition × verdict
  trajectories/<task_id>_<seed>.jsonl
  summary.md
```

`run_id = <yyyymmdd>-<agent>-<suite>-r<n>`

---

## 6. 仓库结构

```
softwarex/
├── README.md
├── pyproject.toml
├── environments/                  # ① SUBSTRATE
│   ├── common/                    #   序列化 / typed action / state diff / 检查点抽象
│   ├── xcap/                      #   XCAP 适配:UIA 探针、CLI 封装、pgDatabase
│   ├── wireshark/                 #   Wireshark 适配(主开发环境)
│   └── replay/                    #   离线 replay 环境(不占 license)
│
├── bench/                         # ② BENCH —— 🔴 不得 import agent/
│   ├── tasks/{xcap-l1..l5b,probes,wireshark-paired}/
│   ├── careers/                   #   🆕 career 模式的任务序列与 holdout 划分
│   ├── oracles/                   #   predicate / exact / file_diff / rubric / evidence
│   ├── gt/                        #   L5 的结构化 GT(schema 见 GT 计划 §2.1)
│   ├── rubrics/
│   └── suites.yaml                #   套件定义与划分
│
├── agent/                         # ③ AGENT
│   ├── m1_onboarding/  m2_observation/  m3_transaction/
│   ├── m4_checkpoint/  m5_verifier/     m6_skills/
│   └── baselines/                 #   B0–B6 阶梯
│
├── tools/                         # 离线工具(不依赖 XCAP)
│   ├── diag_parser/  rsf_reader/  detector_battery/
│   ├── injection_controller/      #   衰减器 / netem / 注入日志
│   └── gt_pipeline/               #   PostgreSQL → GT
│
├── eval/
│   ├── harness.py                 #   跑一个 suite
│   ├── probes/                    #   P5 / P6 / P7 的素材与 prompt
│   └── report.py
│
├── rpc/                           # controller ↔ worker(隔离机)
│   ├── controller.py              #   联网侧,跑 LLM
│   └── worker.py                  #   隔离侧,只驱动软件
│
├── data/                          # 🔴 不入 git;清单 + checksum 入 git
│   ├── MANIFEST.csv               #   dataset_id / 路径 / checksum / Ver / ICD / split
│   └── sessions/<session_id>/     #   见 DATA-COLLECTION-SPEC §8
│
├── results/<run_id>/
└── docs/                          # 现有 9 份规划文档
```

### 6.1 CI 强制的四条

| # | 检查 |
|---|---|
| 1 | 🔴 `bench/` 不 import `agent/`(可分离性) |
| 2 | 🔴 所有正式 suite 里的任务 `oracle.verified == true` |
| 3 | 🔴 `dev` / `test` / `heldout` 无 trace 重叠(含变异母本) |
| 4 | 动作表 manifest 存在且**语义中立性检查通过**(领域词黑名单) |

---

## 7. 命名约定

| 对象 | 格式 | 例 |
|---|---|---|
| `task_id` | `<WORKLOAD>-<MECH>-<nnn>` | `STREAM-COV-007` |
| `session_id` | `<yyyymmdd>-<地点>-<设备>-s<n>` | `20260915-lab-A-s03` |
| `dataset_id` | `DS-<来源>-<标识>` | `DS-INJ-20260915-s03` |
| `run_id` | `<yyyymmdd>-<agent>-<suite>-r<n>` | `20260904-astra-probes-r1` |
| 故障机制 | `FAULT-CATALOG` 的 `F-nn` | `F-01` |

---

## 8. 明确不在范围内

> **benchmark 论文必须写清边界,否则审稿人替你划。**

| 不做 | 原因 |
|---|---|
| XCAP 的**采集端**(XCAL)自动化 | 我们做分析端 |
| 实时/在线分析 | XCAP 是离线后处理工具 |
| 非 Qualcomm chipset 的深度覆盖 | 数据与 ICD 所限;Selective Parsing 本身也只支持 QC LTE |
| 5G-NR 的完整覆盖 | 现有数据以 LTE 为主 |
| Map / 地理类任务 | 依赖 Map 引擎授权(D-6 未定) |
| 多用户 / 权限 / 协作场景 | 单用户桌面工具 |
| 修复网络故障 | 只做**分析**,不做**处置** |
| 🆕 **帧级 / 亚秒级时间对齐的题面** | 🔴 实测跨层对齐精度只到 **±1.5 s(streaming)/ 秒级(teleop)** ⇒ "这一帧对应哪个 slot"不成立 |
| 🆕 **以"协议层"为自明单位的表述** | XCAP 按 **RAT × 功能块**组织,**`Layer2` 零表** ⇒ 层是**投影**,须显式声明规则(见 GT 计划 **T9**) |

---

## 9. 与其它文档的关系

| 本文件的 § | 详细定义在 |
|---|---|
| §2 数据类型 | [[data_collection/DATA-COLLECTION-SPEC]] §1(三层)· §8(归档) |
| §3 任务类型 | [[data_collection/TASK-GT-GENERATION-PLAN]] §0.5(阶梯)· §2.1(GT schema) |
| §3.2 探针 | [[baseline_status/ASTRA-EVAL-SPEC]] §3 |
| §4.1 软件约束 | [[xcap_docs/XCAP-KB-FINDINGS]] 全文 |
| §4.3 动作面 | [[weekly_tasks/W02_2026-09-07_Substrate]] S2 / S2.5 |
| 故障机制 `F-nn` | [[data_collection/FAULT-CATALOG]] §2 |
| 六机制 M1–M6 | [[PLAN-SoftwareX-Sandbox-Benchmark]] §1 |
| 当前状态 | [[MASTER-STATUS]] |

---

## Notes

- **v1(2026-09-04)**:建立。三条硬约束值得单独记住:
  ① **`bench/` 与 `agent/` 可分离**(CI 强制)
  ② **`oracle.verified` 是进入正式评测的硬门**(现有 22 道只有 11 道够格)
  ③ **动作面语义中立 + 版本化落盘** —— 它就是我们声称不存在的那份手册
- ⬜ **§0 命名待定**;⬜ §2.2 三个 `⬜` 数据集待 D-1;⬜ §3.1 L5a 待 T-05/T-11。
