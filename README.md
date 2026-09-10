# AI 分享项目 · 一个 Bug 的一生

> 面向 IT 部门（其他部门可能在场）的 30 分钟技术分享材料。
> 主题：**AI 第一次可以拥有编制**——用 DeepSeek Harness 组织一支三人 Agent 团队，走完「接单 → 分析 → 计划 → 人工确认 → 实现 → 验收 → 留痕」的完整闭环。

---

## 这套文档给谁用

给你自己。**读者只有一个人：上台的那个人。**
所以这里没有演讲稿全文，只有：讲什么、什么时候讲、必须说出口的那一句、以及讲砸了怎么办。

---

## 分享定位（先看这个，再读别的）

| 维度 | 结论 | 影响 |
|---|---|---|
| 主要目标 | **可复制**（听众回去敢自己试） | 方法论必须落到"你可以怎么设岗定权" |
| 次要目标 | **认知冲击**（好看、印象深） | 靠"它停下来了，它在等我点头"这一秒 |
| 不追求 | ROI 汇报、立项、承诺落地 | 全程推演语气，不给数字承诺 |
| 时长 | 30 分钟以内 | 高密度版本，**必须有砍单预案** |
| 演示 | 一个"修小 bug"的轻量演示 | 不依赖内网权限，CSV 兜底 |
| 性质 | **纯畅想 + 能力展示** | 全程区分"现在能用"和"畅想中" |

---

## 三条纪律（每次改文档都要回来检查）

### 纪律一：每个能力都贴标签

| 标签 | 含义 | 例子 |
|---|---|---|
| 🟢 | 现在就能用（**= 在本文核验的 `0.1.5-alpha.2` 上默认可用**） | `dsh-subagent`、`dsh-plan-mode`、`dsh-user-approval` |
| 🟡 | 实验性 / alpha | DeepSeek Harness 本体（当前 `0.1.5-alpha.2`）；OpenDesign 的 DSH 集成（官方测过 dsh `0.1.0-rc.6`） |
| 🔵 | 畅想 / 需自己接 | 反向把 OpenDesign 当 MCP server 接进 DSH（**官方未打通**） |

**坦白 alpha 不是减分项。** 台上说一句"这东西现在是 alpha，我是拿它做推演，不是推荐你们上生产"，可信度立刻上去——因为听众知道你没在卖东西。

### 纪律二：不说"agent team 是 DSH 的功能"

DSH 里**没有**一个叫 agent team 的开关。
`dsh-experimental-agent-team` 只存在于 DSH 的 `devDependencies`，**没有随 npm 包 `0.1.5-alpha.2` 发布**（已核实：安装目录下不存在该包）。

真相是：**团队是用原语拼出来的模式**——`subagent` + `workflow` + `plan-mode` + `user-approval`。

> ⚠️ 接单用的 `webhook` **不算在内**：它是 🟡 可选 overlay，**默认 profile 里没有**。所以"团队怎么组"和"单子怎么进来"要分开讲，否则 IT 同事一装发现没这条链，你就被动了。

> 这个真相**本身就是最好的"可复制"论据**：因为听众学到的是方法，不是某个产品的按钮。

### 纪律三：不承诺落地

用"如果这样做会怎样"的推演语气。
结尾只给一个**最小可试动作**，不说"我们部门将如何如何"。

---

## 30 分钟骨架（完整讲稿与砍单见 `演讲稿.md`）

| 部分 | 时长 | 一句话目的 |
|---|---|---|
| 开场 | 2 min | 把"工具"换成"编制" |
| ① 介绍 | 3 min | DSH 是运行时不是聊天框；OD 认识你装好的 CLI |
| ② 安装命令 | 2 min | 让人真能装起来（细节留文档） |
| ③ 用什么插件 | 2 min | 最小五件套 |
| **④ team member 怎么创建 + 职责** | **6 min** | **核心**：三层机制 + 岗位卡 |
| ⑤ 怎么做 workflow | 3 min | 脚本只协调；**门在运行时** |
| **★ 现场演示** | **4 min** | ⭐ "它停下来了，它在等我说可以" |
| ⑥ agent 独有 skill | 1.5 min | 技能清单本身就是边界设计 |
| **⑦ 动态 Teams** | **3 min** | **核心**：岗位可以动态选，权限不能动态长 |
| ⑧ 项目背景故事 | 1.5 min | 五句话模板，主角是人不是 AI |
| 收尾 | 2 min | 三个判断 + 最小可试动作 |

⚠️ **8 个点全讲 + 演示，30 分钟是紧的。** 标准版 22 分钟、保命版 16 分钟的压缩方案与**砍单决策表**见 `演讲稿.md` 附录 B。

---

## 文档地图

| 文件 | 什么时候读 |
|---|---|
| `README.md` | 现在。定位、纪律、事实基线 |
| **`演讲稿.md`** | ⭐ **主交付物**。八部分演讲稿 + 话术卡 + 时间分配 + Q&A 索引 |
| `02-三人团队设计.md` | 准备期。**创建岗位的三层机制**、权限矩阵、硬门、配置骨架 |
| `03-Workflow-与-DSH.md` | 准备期 + 课后答疑。DSH 能力速查 + workflow 钩子与脚本 |
| `04-Design-优化与-OpenDesign.md` | 准备期。DESIGN.md、五维自评、OD 安装、与 DSH 协作 |
| `05-演示与答疑.md` | **上台当天**。演示步骤、静态截图兜底、Q&A 应答 |
| `demo/issues-sample.csv` | 演示数据（UTF-8 BOM + CRLF，Excel 双击不乱码） |

### 演讲稿八部分 ↔ 你的需求对照

| 演讲稿章节 | 对应你的要求 |
|---|---|
| 第一部分 | **介绍**（DSH / OpenDesign / 两者关系） |
| 第二部分 | **安装命令**（含自查清单 + 两个坑） |
| 第三部分 | **用什么插件**（按环节排 + 最小五件套） |
| 第四部分 ★ | **team member 怎么创建 + 职责说明**（三层机制 + 三张岗位卡） |
| 第五部分 | **怎么做 workflow**（钩子、脚本、"门在哪"） |
| 第六部分 | **agent 独有 skill**（preset 自带技能目录 + 调用策略） |
| 第七部分 ★ | **能否按项目/功能点单独设立或动态创建 Teams** |
| 第八部分 | **项目背景故事怎么写**（五句话模板 + 三个纪律） |

---

## 事实核对基线

> 全部已核实到源码 / 官方文档层面。**改文档时如果和这张表冲突，以这张表为准，并重新核实。**

### DeepSeek Harness

| 事实 | 值 |
|---|---|
| 版本（本机核验） | `0.1.5-alpha.2`（🟡 alpha） |
| 版本（npm latest） | **`0.1.5-rc.1`** —— 已到 RC，比本机核验版本新 |
| 安装 | `npm install -g @deepseek-ai/dsh`（公开包）；需 Node `^22.19` 或 `>=24`（本机 v24.19.0） |
| 入口模式 | `dsh web` · `dsh --profile headless "job"` · `dsh --profile acp` · `dsh --profile sdk` / `sdk-minimal` · `dsh plugin --profile <name> <pnpm args>` |
| 配置模型 | profile = bundle + patch 层叠加；`--dump-config` 可不启动查看合成结果 |
| **不存在** | `dsh-experimental-agent-team` 未随 npm 包发布 |
| **创建岗位的三层** | **① `dsh-agent-presets`**（岗位是谁，带自己的 **skill 目录**）　**② `dsh-permission-presets`**（沙箱模式 + 审批策略**捆绑**成具名预设，`/permission` 切换）　**③ 具名 `dsh-tool-subagent` 实例**（`toolName` / `persona` / `toolFilter` / `agentOptions` / `maxDepth`） |
| ⚠️ 关键约束 | **子 agent 会加入其父方的组装** → **不能靠 preset 给单个 subagent 换岗**；**只有空会话能切 preset**；permission preset 在**创建会话时**读取，之后改动不影响运行中的会话 |
| agent preset 位置 | `<dshHome>/.agent-presets`，或配置 `roots`（每项带 `path` 与 `trust`）；创建方式=**复制现有 preset 目录**；id 规则 `[a-z0-9][a-z0-9-]*` |
| 动态团队 | 岗位**可**动态选（preset roots 按项目分目录 + workflow 脚本条件分支）；**权限不可运行时生成**（preset 是受信任配置） |
| skill 机制 | `dsh-skill`（合并注册表，本身不含内容）+ `dsh-skill-filesystem`（本地发现）+ `dsh-tool-skill`（模型访问）；每个 skill 有 `modelInvocable` / `userInvocable`；插件可用 `ctx.skills.register(...)` 内嵌 |

### OpenDesign（OD，`nexu-io/open-design`，仓库 main 0.22.1）

| 事实 | 值 |
|---|---|
| 与 DSH 的集成方式 | **原生 runtime adapter**（**不是** MCP、**不是** skill）：runtime id `deepseek-harness`，每轮执行 `dsh --profile open-design --stdio` |
| `od agent setup deepseek-harness` | 只把 OD 内嵌的 profile 组件装进 dsh 的 `open-design` profile，再 rescan + 连接测试。**不安装、不升级 dsh**。无 `list`/`remove`/`doctor` 子命令 |
| 前置条件 | 官方 dsh 已装 + **OD daemon 必须正在运行**（否则 exit 64 `daemon-not-running`） |
| `od mcp install <agent>` | 17 个 agent，**名单里不含 deepseek-harness**（DSH 走原生 runtime 通道） |
| `od` CLI 安装 | **没有官方 npm 包。** registry 上的 `open-design` 是 **2021 年的历史占位包**（不是本项目，实测 HTTP 200）。`od` 随桌面应用附带；源码方式用 pnpm |
| Linux | **无官方预构建产物**，仅源码构建（issue #4368） |
| macOS | `brew install --cask open-design`（macOS ≥ 12） |
| Docker | 端口 `7456`，Basic auth 用户名 `open-design`，密码为 `.env` 里的 `OD_API_TOKEN` |
| DESIGN.md | **不是固定 9 段式**；新包门槛 ≥7 个实质性 H2 |
| 五维自评 | Design Jury（内部名 Critique Theater）：Designer / Critic / Brand / A11y / Copy；composite = critic×0.4 + brand×0.2 + a11y×0.2 + copy×0.2；阈值 8.0/10，至多 3 轮 |
| 反向集成 | OD 有 stdio MCP server（`od mcp`），DSH 有 MCP client，**但官方未给出互接步骤**；OD phase one 明确 "MCP: not forwarded in phase one" → 🔵 |

### 本部门环境

| 事实 | 值 |
|---|---|
| 缺陷 / 任务跟踪 | **Azure DevOps**（Work Item / Board / Repo / PR 审批 / Pipeline） |
| 演示兜底 | **CSV** 代替 ADO，不依赖内网权限 |

---

## 准备顺序建议

1. **读 `演讲稿.md`** —— 这是主体，八部分按你的需求排好了
2. 掐表把时间轴念一遍（**真的念，你会删掉三分之一**），按 `演讲稿.md` 附录 B 决定砍哪几段
3. 读 `02` 的 §5.1（三层机制），确认你**不看稿也能说清"岗位怎么创建"**
4. 读 `03` / `04` 只记结论，细节留给 Q&A
5. 按 `05` 把演示**完整跑通两遍**，并**截好静态图**（兜底用）
6. 上台前只带 `演讲稿.md` 和 `05-演示与答疑.md`

---

## 一条元规则

这套文档里所有的技术断言都有来源。
**如果上台前你发现任何一条对不上，删掉它，不要临场圆。**
30 分钟的分享，讲错一条技术细节的代价，远大于少讲一条。
