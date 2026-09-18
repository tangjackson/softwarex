P1P4_flow.md
100%
# P1–P4 测试流程

被测模型：`gpt-6-astra`（Responses API）
被测软件：`XCAP-M O5.33.0.15 25-01-23`，VM `DESKTOP-CC4OI10`，requireAdministrator
要回答的：Q1 —— agent 能不能操作 XCAP 完成任务（目前 0 个数据点）

---

## 0. 评审结论（2026-09-13）

**结构可用,可以按此跑。** preflight 七条、ref 白名单、L3 用程序化状态谓词而不读模型的话 —— 这三条都对。
下面四处 🔴 **必须改了再开跑**,三处 ⚠️ 建议改。**已直接改在本文对应位置,均标 `【改】`。**

| # | 问题 | 位置 | 不改的后果 |
|---|---|---|---|
| 🔴 1 | **写动作求差的作用域没写**,而 Filtering 是全局的 | §3 | 模型设一个 filter 改变全部窗口读数,diff 只看当前窗口 → **报 `ok`,副作用整个丢失** |
| 🔴 2 | **`no_effect` 二义** | §5 | 分不出"动作真的失败了"和"序列化器没记这个字段"(P32 就是后者)→ **A9 的静默失败率永远算不准** |
| 🔴 3 | **P0 只在开跑前跑一次** | §4 / §5 | 每次启动 XCAP 重写 >1.6 MB;冷启动有窗口期输入进不去。**9 点通过不代表第 47 次运行有效** |
| 🔴 4 | **七档 rubric 没落地** | §5 | 判分是二值,**`Confident-Wrong` 丢掉** —— 而追问已经在流程里了,白问 |
| ⚠️ 5 | T5 陷阱题"补还是替"未定 | §4 | 替掉就失去 P1–P4 里唯一测「会不会声明配置依赖」的题 |
| ⚠️ 6 | B 条件铺满 33 次 | §4 | P1–P4 考操作软件,联网帮不上;**先跑子集可省 21 次** |
| ⚠️ 7 | 环境指纹未列具体项 | §5 | **版本号不唯一确定二进制集合**(B6 已证),只记 `5.33.0.15` 不够 |

⭐ **另一条不是问题,是机会**:`driver_uia.py` / `serializer.py` / `actions.py` 这三件
**就是 W03 计划里的 S1 + S2**。建议现在就落到 `environments/common/`(序列化 + 求差)与
`environments/xcap/`(driver + actions),**不要放在评测目录下** ——
否则到 P4 建 agent 时要重写一遍。

---

## 1. 组件

| 名字 | 实际是什么 | 代码量 | 跑在哪 |
|---|---|---|---|
| `XCAP-M` | 被测软件 | — | VM |
| `ui_inspect.py` | 找窗口、包 pywinauto（W01 就有） | 已有 | VM |
| `driver_uia.py` | **唯一碰机器的一层**：走一遍界面得到原始控件树；发一次输入 | ~1000 行 | VM |
| `serializer.py` | 原始树 → 归一化状态；两个状态求差 | ~500 行 | VM |
| `actions.py` | **11 个动作**。每个写动作 = 看一眼 → 发 → 再看一眼 → 求差 → 判 `ok`/`no_effect` | ~600 行 | VM |
| `worker.py` | TCP 服务端，**提权**。学长指定的唯一入口 | 学长的 | VM |
| `controller.py` | TCP 客户端。学长钉死的边界 | 学长的 | 联网侧 |
| `xcap-adapter` | 把 MCP 的 JSON-RPC 翻成 `Controller.call()`。纯管道，零逻辑 | **~200 行** | 联网侧 |
| `harness` | Astra 的自主循环：问模型 → 模型要调工具 → 调 → 结果回灌 → 再问 | 已有（`run_p5a_mcp.js`） | 联网侧 |
| `gpt-6-astra` | **被测模型** | — | OpenAI |

**改名**：协议适配层原名 `mcp-xcap`，改为 `xcap-adapter`。
理由：`Windows-MCP` / `mcp-windows` 那类是完整的桌面自动化产品；这个是 200 行管道，不是同类东西，不该用同一种命名。
旧名字出现在 `S2_and_mcp_xcap_0913.md`、`p04_export_path_0913.md`、`P0_closed_4of4_0913.md`。

---

## 1.1 备注：这里说的 "MCP" 指什么

**MCP 是协议名，不是某个软件。** 链路图里 `MCP stdio JSON-RPC` 那一段，意思是"我们用这个协议说话"，不是"我们用了一个叫 MCP 的产品"。

四个容易混的东西：

| 名字 | 是什么 | 本项目 |
|---|---|---|
| **MCP** | **协议**。规定 `initialize` / `tools/list` / `tools/call` 这些消息长什么样 | ✅ 我们说这个协议 |
| `Windows-MCP` · `mcp-windows` · `desktop-pilot-mcp` | **现成的桌面自动化产品**，别人写的，装上就能让 agent 点 Windows 界面 | ❌ **没用**。候选给 B1/B2 对照组 |
| `mcp-official-fs` | 官方 filesystem server。P5-A 的 mcp 臂用的就是它，**只能读文件** | ✅ 用过（P5-A，规格外，已结束） |
| **`xcap-adapter`**（原 `mcp-xcap`） | **我们写的 200 行管道**。说 MCP 协议，把工具调用转成 `Controller.call()` | ✅ **这次用它** |

⚠️ `xcap-adapter` 和 `Windows-MCP` 不是同类东西：后者是完整产品（自带截图、坐标点击、PowerShell、注册表），前者只是协议转换，真正的能力全在它下面的 `actions.py` / `serializer.py` / `driver_uia.py` 里。

---

## 2. 链路

```
gpt-6-astra
   │  MCP stdio JSON-RPC        ← 协议，不是产品名
harness
   │
xcap-adapter
   │  Controller.call() → TCP        ← 学长钉死的边界
worker.py  handle_command()          ← 学长指定的唯一入口（提权）
   │
actions.py (11 个动作)
   │
serializer.py (状态 + 求差)
   │
driver_uia.py (UIA)
   │
XCAP-M
```

---

## 3. 动作面（11 个，规格 §2.2 逐字）

```
screenshot(path)            截屏到文件
read_tree(subtree?)         观察界面。返回归一化状态 + ref 清单
read_window_list()          列出打开的窗口，各带一个 ref
read_file(path)             读一个文件
click(target)               点一个 ref。返回状态差异
drag(from_ref, to_ref)      拖。返回状态差异
set_field(target, value)    设一个 ref 的值。返回状态差异
type(text)                  在焦点处打字。返回状态差异
key(combo)                  发一个组合键。返回状态差异
scroll(target, delta)       滚动。滚动位置不在状态里，通常报 no_semantic_change
export(view, path)          把一个 ref 的内容写到文件。报状态差异 + 文件是否非空
```

target 只能是 `read_tree` / `read_window_list` 发出的 ref，编一个名字会被拒。

### 【改】🔴 写动作求差的作用域 = **全工作区**

读动作可以用 `read_tree(subtree)` 限定范围；**写动作的前后快照不可以**。

> **Filtering 是全局的**（跨 Graph / Map / Table / CDF / PDF）。
> 模型通过点击设一个 filter → 改变**全部窗口**的读数 → 若求差只看当前窗口 → **报 `ok`，副作用整个丢失**。

⇒ `actions.py` 的写动作：**前后两次快照一律取全工作区**，即使动作本身只作用于一个控件。
同类全局项还有 Area / Route Binning —— 它们会**产生持久化派生状态**（挂进参数树，甚至生成新 Model），
状态空间可递归增长，求差必须能看到新出现的子树。

---

## 4. 流程 A —— 到开跑之间

```
1  跑 P0 over MCP                          你，VM        通过 = 四题全绿
2  动作表 manifest 落盘                    我，10 分钟   产物 = manifest.json
3  L4.1 复核，补第 11 题                   我（GUI）     产物 = 两个计数
4  T5 陷阱题：【改】⚠️ 补进去，不替换        我            产物 = 第 12 题
      题：「这条 trace 最差的 SINR 是多少」
      不需要新数据，A4 已实测：All 档 −20.0 / 1Sec 档 −12.9
      判分：只报一个数而不声明采样档 = Partial，不是 Full
5  harness 改三处                          我
      server      → xcap-adapter
      工具拦截    → 白名单恰好 11 个
      前置检查    → 加 §5 的 7 条
6  【改】⚠️ 分两批跑                       你 + 我
      批一  A 闭卷全量    12 题 × 3 seed        = 36 次   ← 主结果
      批二  B 开卷子集    每档 1 题 × 3 seed    = 12 次   ← 先看有没有差
            B 若与 A 无差异 → 不铺开，省 21 次
            B 若有差异     → 补齐剩余 24 次
7  判分                                    我 + 你       程序化 oracle + 两人独立
8  【改】🔴 P0 复测                         你，VM        与开跑前首尾对照
```

---

## 5. 流程 B —— 一次运行内部

```
preflight   【改】🔴 断言 8 条，任一不满足 → 停，不跑
   1  worker 已提权
   2  XCAP 运行中且未最小化
   3  侧边栏展开（Graph/Map/Table/CDF/PDF/Event/Statistics 七个标签都在）
   4  无残留对话框 / 弹出菜单
   5  Sampling Rate = 1Sec
   6  环境指纹落盘（【改】⚠️ 具体项见下）
   7  模型可见目录无我们的临时文件
   8  【新】canary 动作 —— 做一次已知必定生效的操作，确认 diff 非空
        为什么：2 只能证明"进程在、窗口没最小化"，证明不了"输入进得去"。
        冷启动后 MapXtremeViewer730.exe 抢前台且不释放，
        期间自动化 100% 不可用，需人工点一下才解除。
        canary 失败 = 环境没就绪，不是模型不行。

发题面          独立会话，不带记忆，不给任何 XCAP 文档

循环
   模型          → 要调哪个工具
   harness       → 查白名单 → 调 xcap-adapter
   xcap-adapter  → Controller.call() → TCP → worker
   worker        → actions.py
        读动作:  driver 走一遍界面 → serializer 归一化 → 返回状态 + ref 清单
        写动作:  看一眼（全工作区）→ 发输入 → 再看一眼（全工作区）→ 求差
                 【改】🔴 三分，不是二分：
                   有差                         → ok
                   无差 且 在"无语义变化"白名单  → no_semantic_change
                   无差 且 不在白名单           → no_effect  ← 疑似静默失败
                 为什么要分：
                   "动作真的失败了" 和 "序列化器没记这个字段" 都表现为无差。
                   P32 就是后者 —— 展开树节点会顺带选中它，
                   若 serializer 不记选中项，一次成功的选中会被报成 no_effect。
                 白名单必须显式列出并随 manifest 落盘（当前已知：scroll）。
                 A9 的静默失败率只能从 no_effect 这一档算，不能从 no_effect+no_semantic_change 算。
   结果原样回灌给模型
直到模型给出答案

追问           逐字，不改写：
               你对这个答案有多确定？依据是什么？

落盘           req_NN.json（轨迹）+ 环境指纹

【改】⚠️ 环境指纹必须含这 10 项（只记版本号不够 —— B6 已证
              「版本号不唯一确定二进制集合」，本机两代 Wireshark 插件并存）：
   1  10 个主二进制的 size
   2  4 个 PDB 路径
   3  plugins/3.2 与 plugins/3.6 各自的文件数
   4  dongle Serial（本机 03AA19C0024）
   5  13 个影响解析的配置键
   6  EtcOption.ini 全文
   7  analysis_config 全字段（sampling_rate / filters / binning /
        canonical_clock=TIME_STAMP / verification_clock=chipset /
        viewer_mode / multi_process / process_as_file / build_path）
   8  动作表 manifest 版本
   9  皮肤（Classic / Ribbon）—— 两套暴露不同功能集，必须固定并记录
  10  Num_of_Process 实测值 + 标注「capacity 待封口」

判分        【改】🔴 两步，不是一步
   第一步  程序化 oracle，判对错（二值）
      L1 / L2     定值 / 数值 + 容差
      L3          状态谓词 —— 程序化读软件状态，不读模型说的话
      L4          比对导出文件
      L2-T5       只报一个数而不声明采样档 → Partial（不是 Full）

   第二步  用追问的回答判确定度，与第一步交叉 → 规格 §4.1 的七档
                        │ 高确定度        │ 存疑 / 说要实测
      ──────────────────┼─────────────────┼──────────────────
      oracle 判对       │ Full            │ Full（可另标）
      oracle 判错       │ Confident-Wrong │ Hedged
      未给结论          │ —               │ Hedged

      为什么：追问已经在流程里了，不用就白问。
      Confident-Wrong 是 P5 那边最有价值的一档（0/45），P1–P4 同样要统计。
```

---

## 6. 题目（【改】现 10 道可用 → 补两道后 12 道）

| 探针 | 档 | 题 | oracle | 状态 |
|---|---|---|---|---|
| P1 | L1 检索 | L1.1 · L1.3 · L1.4 | 定值 | ✅ 3 道 |
| P2 | L2 计算 | L2.1 · L2.2 · L2.4 | 数值 + 容差 | ✅ 3 道 |
| **P2** | **L2 计算（T5 陷阱）** | **【新】L2.T5「这条 trace 最差的 SINR 是多少」** | **数值 + 必须声明采样档** | 🆕 流程 A 第 4 步 |
| P3 | L3 配置 | L3.2 · L3.4 | 状态谓词 | ✅ 2 道 |
| P4 | L4 对比 | L4.4 · L4.5 | 导出文件 | ✅ 2 道 |
| P4 | L4 对比 | ⬜ 第 12 题（L4.1 复核后补） | 导出文件 | 流程 A 第 3 步 |

🔴 **只有 `oracle.verified == true` 的题进正式评测**（规格 R7）。
当前 22 道题里：**11 道重测通过 / 3 道硬伤（已回收成 P7）/ 2 道口径过时 / 4 道待 GUI 复核**。
本表用的是其中 10 道 + 待补 2 道。

⚠️ **`ASTRA-EVAL-SPEC` §3 写的是 18 道（P1=5/P2=5/P3=4/P4=4）** —— 那是当时的预期，
**以本表为准**，规格那边待同步。

两个条件：A 闭卷（不联网，主结果）· B 开卷（联网，差值 = 公开文档能帮多少）
每题 ≥3 seed，每题独立会话
【改】⚠️ B 条件先跑子集（见流程 A 第 6 步）

---
