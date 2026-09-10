# 04 · Design 优化与 OpenDesign

> **分享第 ④ 段和第 ⑤ 段的底稿。**
> 全部事实已核实到 OpenDesign 仓库 `main`（0.22.1）源码 / 官方文档。
> ⚠️ **版本口径**：本文事实基线是**仓库 `main` = 0.22.1**；而 **macOS cask 当前是 0.22.2**，两者不是同一个口径，别混着说。
> 标 🟡🔵 的请照标签说。

---

## 第 ④ 段 · Design 怎么优化（3 分钟底稿）

> 📌 **链路口径（先说清楚，免得讲串）**
> 本节能力属于 **OpenDesign 侧**，在 OD 内部是可以用的。
> 但**整条链路要经由 OD↔DSH 适配**——那是 🟡，而且**官方只在 dsh `0.1.0-rc.6` 上验过，本机 `0.1.5-alpha.2` 未经验证**。
> **台上别把"不到 8 分不让出门"说成 DSH 的功能。**

### 4.1 `DESIGN.md` 是"品牌契约"，不是样式表

> 设计不是画出来的，是**先签合同再画出来的**。

- 每次渲染都要读当前生效的 `DESIGN.md`。
- 它**不是**写在 prompt 里的一段描述——它是仓库里的一个文件，**可版本控制、可评审、可复用**。
- 系统提示词的构成大致是：`BASE + DESIGN.md + SKILL.md`。
  → 意思是：**换设计系统 → 下一次渲染立刻生效，不用改 prompt、不用重新教 agent。**

### 4.2 它不是随便写写的（这条能证明"这是认真的"）

- 官方明确：**不是固定的九段式编号 schema**（`not a fixed nine-section numbered schema`）。
- 一个新设计系统包的门槛是：**至少 7 个实质性 H2**。
- 建议覆盖的面向：

  > 主题 · 色彩 · 字体 · 间距与布局 · 组件状态 · 动效 · 无障碍 · 反模式

**"反模式"这一栏值得单独点一句**——它要求你把"我们**不**做什么"也写下来。
一个只写"要什么"、不写"不要什么"的规范，是没法执行的。

### 4.3 一个设计系统包 = 三个文件

| 文件 | 作用 |
|---|---|
| `manifest.json` | 包元数据 |
| `DESIGN.md` | **品牌契约正文** |
| `tokens.css` | 编译后的 token（仅 `DESIGN.md` 时走兼容回退） |

### 4.4 ⭐ 五维自评闸门（Design Jury / 内部名 Critique Theater）

**本段最值钱的一点**：artifact 产出后，**先自评打分，不到分不让出门**。

| 维度 | 角色 | 在看什么 |
|---|---|---|
| Designer | 设计者 | 基本设计质量（**注意：权重为 0**） |
| Critic | 批评者 | 整体批判，**权重最高** |
| Brand | 品牌 | 是否符合品牌契约 |
| A11y | 无障碍 | 可访问性 |
| Copy | 文案 | 文字质量 |

**合成公式**：

```
composite = critic×0.4 + brand×0.2 + a11y×0.2 + copy×0.2
```

- **阈值 8.0 / 10**，默认**至多 3 轮**迭代
- devloop 以 `critique.score` 作为终止条件

**🎯 讲的时候不要说公式。** 说这一句：

> "它给设计打分——**不到 8 分不让出门**，最多重做 3 轮。"

**公式留在文档里等提问。** 如果真有人问，再补一个细节，杀伤力更大：

> "有意思的是，'设计者'自己那一维**权重是 0**。
> 打分的全是别人：批评者、品牌、无障碍、文案。
> **自己觉得好看，不算数。**"

> ⚠️ **别把两道"门"讲混（现场必被问）**
> 这里是**质量闸门**：卡的是**产物质量**——分数不到 8.0 就不许出门。
> `02` 里那道是**审批门**：卡的是**人**——没人点头就不许动手。
> 两处措辞都是"门"，但机制完全不同。有人问"到底有几道门"，答案：**一道卡产物，一道卡人。**

### 4.5 另一条优化路径

- `od-design-refine` scenario：对已有产物继续打磨的完整场景。
- 切换 design system → 下次渲染即生效（见 4.1）。

### 4.6 第 ④ 段收尾句

> "所以'优化设计'在这套东西里不是'再生成一次试试'，
> 而是**先有一份可评审的合同，再加一道不到分不放行的闸门**。
> 一个是标准，一个是纪律。"

---

## 第 ⑤ 段 · OpenDesign 是什么、怎么装、怎么跟 DSH 协作（3 分钟底稿）

### 5.1 它是什么

**OpenDesign（OD）= 开源的 Claude Design 替代品。**

- Apache-2.0，**本地优先**的桌面应用（macOS / Windows）
- 自己**不打包模型、也不打包 agent**——它用**你电脑上已经装好的 CLI** 当引擎
- 输出面向：原型（网页 / 桌面 / 移动）、实时仪表板、简报（deck）、图片、视频（HyperFrames）、文档
- 可导出 HTML / PDF / PPTX / MP4 / Markdown

**🎯 必须说出口**："OpenDesign 不认识模型，它认识你电脑上**已经装好的 CLI**。"

### 5.2 ⭐ 它和 DSH 的协作方式（本段技术上的惊讶点）

> **不是 MCP，不是 skill，是原生 runtime adapter。**

| 事实 | 值 |
|---|---|
| runtime id | `deepseek-harness` |
| 流格式 | `dsh-profile-jsonl` |
| 每轮怎么跑 | OD 直接执行 **`dsh --profile open-design --stdio`** |
| OD 打包 dsh 吗 | **不打包**（也不打包 Node） |

**支持的能力**：

- 结构化思考（structured thinking）
- 文本输出
- 工具调用与工具结果
- usage（用量）
- **取消**（含进程树回收）
- **会话冷恢复**（回到同一个 Harness session）
- **实时模型发现** + 每个模型自己的推理档位

**phase one 明确推迟的**（说这个显专业，也防被问倒）：

> 凭据录入 · 后台升级 · MCP 注入 · TodoWrite · **subagent UI**

**🖼 视觉**：画一张箭头图 —— `OpenDesign ──原生 runtime──▶ dsh`
⚠️ **不要**画成 MCP 连线。这是本段最容易说错的地方。

### 5.3 安装（三条路，按成本排）

#### 路线 A · 桌面应用（推荐，零配置）

| 平台 | 方式 | 备注 |
|---|---|---|
| macOS | `brew install --cask open-design` | 0.22.2，macOS ≥ 12，Apple Silicon + Intel |
| Windows | x64 安装包 | 官网 / GitHub Releases |
| Linux | ❌ **无官方预构建产物** | **仅源码构建**（issue #4368） |

#### 路线 B · 接进你已有的 CLI

```bash
od mcp install <agent>          # 一句接入
od mcp install <agent> --print  # dry-run 预览
od mcp install <agent> --uninstall
```

支持的 17 个 agent：
`claude` · `claude-desktop` · `codex` · `reasonix` · `raven` · `cursor` · `copilot` · `openclaw` · `antigravity` · `pi` · `vibe` · `hermes` · `cline` · `kimi` · `kiro` · `trae` · `opencode`

hosted 等价写法（`od mcp install` 的薄封装）：
```bash
curl -fsSL https://open-design.ai/install.sh | sh -s <agent>
```

> ⚠️ **注意这张名单里没有 `deepseek-harness`。**
> 这不是遗漏——**因为 DSH 走的不是 MCP，是更深的原生 runtime 通道**（见 5.2）。
> 这一条**一定要主动讲**，否则细心的 IT 同事会当场问你。

#### 路线 C · Docker

```bash
git clone https://github.com/nexu-io/open-design.git
cd open-design/deploy
cp .env.example .env
echo "OD_API_TOKEN=$(openssl rand -hex 32)" >> .env
docker compose up -d
# 打开 http://127.0.0.1:7456
```

Basic auth：用户名 `open-design`，密码为 `.env` 里的 `OD_API_TOKEN`。

### 5.4 DSH 接入的具体步骤

**目标**：让 OD 把 DSH 当引擎用。

**前置条件**：

1. 官方 `dsh` 已安装（OD 官方测过 `0.1.0-rc.6`；Node `^22.19` 或 `≥24`）
   > 本机实际为 `0.1.5-alpha.2`，比官方测过的更新。🟡 属于"应该行，但没被官方验证过"的区间——**这点自己心里有数就行，别在台上打包票**。
2. OpenDesign 版本 **≥ 0.19.1**
3. **OD daemon 必须正在运行**

**步骤**（官方路径）：

```
装官方 dsh
  → dsh web（在里面配好 Key）
  → 安装 OD ≥ 0.19.1
  → 设置 → 模型与提供商 → 本机 CLI
  → 重新扫描
  → 选中 deepseek-harness 卡片，确认"安装连接组件"
  → 测试
```

**命令行等价物**：

```bash
od agent setup deepseek-harness          # 加 --json 可拿结构化输出
```

**这条命令到底做什么**（照实说，别夸大）：

| 会做 | 不会做 |
|---|---|
| 把 OD 内嵌、**hash 校验过的 profile 组件**装进你 dsh 的 `open-design` profile | ❌ **不安装 dsh** |
| 装完 rescan + 连接测试 | ❌ **不升级 dsh** |
| 已兼容时返回 `already-compatible` | ❌ 没有 `list` / `remove` / `doctor` 子命令 |

> ⚠️ **凭据要你自己在 DSH 侧配好。** OD 的 phase one 明确把"凭据录入"列在**推迟项**里（见 §5.2）——**这一步 OD 不会帮你**。所以上面那步 `dsh web` 里配 Key 是必须自己完成的。

> ⚠️ **两个容易翻车的点**：
> - daemon 没在跑 → **exit 64（`daemon-not-running`）**。上台前先确认 daemon 活着。
> - 其他任何 `od agent ...` 写法 → 打印 help 并 **exit 2**。（`od doctor` 是顶层命令，不是 `od agent doctor`。）

### 5.5 三个必须说的坑（说了会显得你真试过）

| 坑 | 事实 |
|---|---|
| **`od` CLI 没有官方 npm 包** | registry 上的 `open-design` 确实存在，但那是 **2021 年的历史占位包（`0.0.1`–`0.0.5`），与本项目无关**（实测 HTTP 200，最后更新 2021-02-09）。OpenDesign 官方**未发布 npm 包**，`od` **随桌面应用附带**；源码方式需 `pnpm --filter @open-design/daemon build` |
| **Linux 没有官方预构建产物** | 只能源码构建。想在公司 Linux 服务器上试的人会第一时间撞到这堵墙 |
| macOS 上 `/usr/bin/od` 会遮蔽 OD 的 `od` | 官方建议用设置页给出的片段 |

### 5.6 🔵 反向集成：现在**没打通**（只作为展望提一句）

| 方向 | 状态 |
|---|---|
| OpenDesign → 用 DSH 当引擎 | 🟡 **已支持**（原生 runtime adapter） |
| DSH → 把 OpenDesign 当 MCP server 用 | 🔵 **官方未打通** |

事实：
- OD 侧**确实有** stdio MCP server（`od mcp`），暴露的工具包括
  `list_projects` · `get_active_context` · `get_project` · `get_file` · `search_files` · `list_files` · `get_artifact` · `create_artifact`
- DSH 侧**确实有** MCP client（`@deepseek-ai/dsh-mcp-client`，stdio / streamable-http）
- **但是**：`od mcp install` 名单不含 deepseek-harness，且 OD 的 phase one 明确写着 **"MCP: not forwarded in phase one"**

**🎯 怎么讲这一条（照这个说，别加工）**：

> "反方向——**把 OpenDesign 当成 DSH 的一个工具来用**——两边各自的零件其实都在，
> 但官方文档里**还没有把它们接起来的步骤**。
> 所以我把它标成'还没打通'。**这条别信我，等我试通了再说。**"

> 主动说一句"这条我还没验证"，比多讲一个亮点更能建立信任。
> 而且它给听众留了一个明确的"下次可以聊这个"的钩子。

### 5.7 第 ⑤ 段收尾句

> "所以这两件事的关系是：
> **OpenDesign 管'做出来长什么样'，DSH 管'这件事谁来做、做到哪一步必须停'。**
> 一个管标准，一个管编制。"

---

## 附 · 上台只需要记住三句

1. **④** "设计不是画出来的，是先**签合同**再画出来的。"
2. **⑤** "OpenDesign 不认识模型，它认识你电脑上**已经装好的 CLI**。"
3. **⑤** "它接 DSH 的方式**不是 MCP**，是**原生 runtime**——每轮直接跑 `dsh --profile open-design --stdio`。"

其余全是答疑弹药。

---

## 附 · 事实来源

| 结论 | 来源 |
|---|---|
| OD ↔ DSH 为原生 runtime adapter、`dsh-profile-jsonl` | `docs/agent-adapters.md` |
| capabilities 与 phase one 推迟项 | `specs/current/deepseek-harness-phase-one-plan.md` |
| `od agent setup` 行为与 exit 64 / exit 2 | `apps/daemon/src/cli.ts` |
| `od mcp install` 17 个 agent 名单 | `apps/daemon/src/mcp-agent-install.ts` |
| DESIGN.md 非固定九段式、≥7 个 H2 | `docs/design-systems.md` |
| Design Jury 五维与加权阈值 | `docs/critique-theater.md` |
| Docker 步骤与 7456 端口 | `QUICKSTART.md` |
| macOS cask 版本与最低系统 | `formulae.brew.sh/cask/open-design` |
| `od` 无 npm 包 | npm registry 实测（`open-design` 为 2021 年历史占位包，非本项目）+ 官方安装文档 |
