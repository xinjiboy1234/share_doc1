# 03 · Workflow 与 DeepSeek Harness

> **分享第 ① 段和第 ③ 段的底稿，同时兼作课后答疑附录。**
> 所有事实已核实到 DSH `0.1.5-alpha.2` 的包内文档；标 🟡 的是可选 overlay 或 alpha 特性。
>
> 📌 **标签约定**：§2.1 的原语表已**逐项标注**；本文其余正文段落若未标注，**默认 🟢**。
> ⚠️ 但请记住 README 的前提——**🟢 指"在本文核验的 `0.1.5-alpha.2` 上默认可用"**，
> 而 DSH 本体整体仍是 🟡 alpha，这些能力会随 alpha 版本漂移。

---

## 一、DSH 是什么（第 ① 段底稿）

### 1.1 一句话

**DeepSeek Harness（`dsh`）是一个 agent 运行时，不是一个聊天框。**

同样一句话的推论：**你要的能力是"装上去"的，不是内置的。**
所以它天然适合讲"编制"——因为编制本来就是配出来的，不是买来的。

### 1.2 三种入口，对应三种使用者

| 入口 | 谁在用 | 形态 |
|---|---|---|
| `dsh web` | **人** | 你看着它干活，随时插话 |
| `dsh --profile headless "job"` | **机器** | 跑一件活、打印最终答案、退出。适合 CI 里调 |
| `dsh --profile acp` | **外部自动化 client** | 通过 ACP stdio 提供服务，直到断开 |
| `dsh --profile sdk` / `sdk-minimal` | **别的系统** | 通过 JSON-RPC stdio 提供服务 |

> **这是整段最值钱的一张表。** 它说明 DSH 的设计目标不是"更好用的对话框"，而是**能被人、被机器、被别的系统三种方式驱动的执行引擎**。

### 1.3 配置模型（一句话带过，别展开）

profile = bundle + patch 层叠加：

```
空根 → bundles 的 patch → profile 自己的 cordis.patch.yml
     → home 级 cordis.patch.yml → --patch 覆盖层
```

`--dump-config` 可以在**不启动**的情况下看合成后的配置树。这一条只用来证明"它可检查、可审计"，不要讲实现。

### 1.4 诚实的自我定位

> 🟡 当前版本 `0.1.5-alpha.2`，**alpha**。
> 我今天不是推荐你们上生产，是**拿它把一件事推演清楚**。
---

## 二、原语速查表（课后答疑用）

> 讲的时候**不要念这张表**。它的用途是：IT 同事课后追问"具体用什么做的"时，你能立刻翻到。

### 2.1 你那条链路的每一环

| 流程环节 | DSH 原语 | 状态 |
|---|---|---|
| 自动接收 issue / bug | `dsh-webhook`（`register(rule)` / `dispatch(delivery)`；唯一内置动作是**在 Web Workspace 创建普通根 Session**）<br>`dsh-webhook-github`（GitHub 适配器） | 🟡 可选 overlay，默认 profile 不含 |
| 用外部事件建一个 agent 会话 | `WebhookSessionRequest`：要求 `workspacePath` / `title` / `prompt` / `agentPreset` / `permissionPreset` | 🟡 |
| 组织团队（起 subagent） | `dsh-tool-subagent` · `dsh-tool-subagent-control` · `dsh-subagent-fork-in-process` · `dsh-subagent-spawn-in-process` | 🟢 |
| 大规模扇出编排 | `dsh-tool-workflow` | 🟢 |
| 先规划、再执行、计划交人批 | `dsh-plan-mode`（`/plan`，通过 `exit_plan_mode` 呈交计划） | 🟢 |
| 审批门 | `dsh-user-approval` | 🟢 |
| 权限分级与隔离 | `dsh-permission-presets` · `dsh-sandbox-policy` · `dsh-fs-sandbox` | 🟢 |
| 长跑目标（跨轮次推进） | `dsh-goal` · `dsh-goal-round-driver` · `dsh-tool-goal` | 🟢 |
| 定时触发 | `dsh-schedule` | 🟡 可选 overlay |
| 每轮都用全新上下文迭代 | `dsh-tool-ralph` | 🟢 |
| 后台长任务 | `dsh-jobs-local` · `dsh-tool-jobs` | 🟢 |
| 留痕、回放、恢复 | `dsh-session-persistence-jsonl` · `dsh-session-checkpoint-policy` · `dsh-session-query` | 🟢 |
| 接外部工具 | `dsh-mcp-client`（stdio / streamable-http；工具名 `mcp__<server>__<tool>`；**只桥接 tools**，不支持 resources / prompts） | 🟢 |
| 技能包 | `dsh-skill` · `dsh-skill-filesystem` | 🟢 |
| 成本可见 | `dsh-token-meter` | 🟢 |
| 上下文压缩 | `dsh-compaction-basic` · `dsh-compaction-tool-result-pruner` | 🟢 |
| 计划模式 + 审批的 UI | `dsh-client-ui-approval` · `dsh-client-ui-workflow-run` · `dsh-client-ui-subagent` | 🟢 |

### 2.2 重要澄清：**不存在**一个叫 "agent team" 的功能

- `dsh-experimental-agent-team` 与 `dsh-experimental-tool-agent-team` 只出现在 DSH 的 **`devDependencies`**，**没有随 npm 包 `0.1.5-alpha.2` 发布**（已核实：安装目录下不存在该包）。
- 所以：**团队是用上面这些原语拼出来的模式，不是一个开关。**

> 这个真相**对分享有利**：你讲的是方法，不是产品说明。听众学到的东西换成任何 harness 都成立。

### 2.3 可选 overlay 是怎么用的

DSH 在 `config/examples/` 里交付几个**可选 overlay**（GitHub 评审 webhook、会话内 Schedule、记忆 MCP 服务、运行时 Cordis 工具）。
官方明确：**它们绝不属于默认 profile**。要手动组合、按官方安全说明安装。

> 讲这句话的用途：说明"接 webhook 是有现成路径的，但要你自己接上并配好密钥"——**不夸张，也不含糊**。

---

## 三、Workflow 怎么建、怎么工作（第 ③ 段底稿）

### 3.1 它是什么

`workflow` 是一个**给模型用的工具**：模型写一段 **JavaScript 编排脚本**，脚本把工作**扇出给多个 subagent**，最后返回脚本的最终 JSON 值。

三个关键事实：

1. **脚本是代码，不是配置。** 你可以写循环、条件、错误处理。
2. **脚本里没有文件系统、没有网络、没有定时器、没有 Node API。**
   → **脚本只负责协调，干活的是 agent。**（这句话是第 ③ 段的"必须说出口"）
3. **父级只看到最终结果**，永远看不到中间 subagent 的消息。子 agent 的工作不会污染父级对话。

### 3.2 调用形态

| 参数 | 必填 | 说明 |
|---|---|---|
| `meta` | ✅ | 身份数据：`name`、`description`，可选 `whenToUse`、`phases` |
| `script` | ✅ | **纯 JS 脚本体**（不含 `export const meta` 语句） |
| `args` | ❌ | JSON 对象，作为全局变量 `args` 暴露给脚本 |

成功返回规范包络 `{ runId, agentsStarted, result }`，向模型渲染为：

```
workflow "issue-to-pr" completed (3 agents).
Return value:
{ ... }
```

**取消与失败绝不会被报告成部分成功**——返回 `Error: workflow run was cancelled` 或 `Error: workflow run failed: <error>`。

### ⚠️ 一次 `workflow` 调用是**阻塞**的（讲稿里要点一句）

父级轮次会**一直等到整张扇出结束**才继续——**没有"后台启动再轮询"这种用法**。

- 脚本运行期间父级在**等待**，模型**只看到最终结果**，看不到中间子 agent 的消息
- 子 agent 的工作**不会进入父级对话**（上下文不会被污染）
- 这正是现场演示时"你会看到它安安静静地在跑、然后一口气给出结果"的原因

> 这一条在答疑时会被问（"它能后台跑吗？"），而它**也是第 ⑥ 段演示观感的解释**——所以不是冷知识，值得占 15 秒。

### 3.3 五个钩子

| 钩子 | 作用 | 关键细节 |
|---|---|---|
| `agent(prompt, opts?)` | 起一个 subagent 跑到完成 | 不带 `schema` 返回最终文本；带 `schema` 返回**校验过的对象**；子 agent 失败时 resolve 为 `null` |
| `phase(title)` | 开一个进度阶段 | 纯展示分组 |
| `pipeline(items, ...stages)` | 每个 item 独立走完各阶段，**阶段之间没有栅栏** | 每个 stage 收到 `(prev, item, index)`；某个 stage 抛错 → **该 item 变 `null`**，跳过它剩余的阶段 |
| `parallel(thunks)` | 并发跑零参函数，**等全部完成**（栅栏） | 抛错的 thunk resolve 为 `null` |
| `log(message)` | 叙述进度 | — |

`agent` 的 `opts` 支持：`label`（显示名）、`phase`（进度分组）、`provider` / `model`（独立覆盖 LLM 目标）。
**不接受** `effort` / `isolation` / `agentType` —— 传了会**大声报错**。

### 3.4 `pipeline` vs `parallel`：怎么选

| | 栅栏 | 什么时候用 |
|---|---|---|
| `pipeline` | **无** | 每个条目独立走完全流程。**首选**——一项慢不会拖住其他项 |
| `parallel` | **有** | 只有当某个阶段**确实需要所有前序结果放在一起**时才用 |

> ⚠️ **下面这条是本文的经验建议，不是官方口径。**
> 官方只规定"仅当用户明确要求工作流或大型多 agent 编排时使用"，**并没有规定 `pipeline` 和 `parallel` 该优先用哪个**。
> 但从机制上看：`pipeline` 阶段间无栅栏，一项慢不拖累其他项；`parallel` 是栅栏，会被最慢的那个拖住。
> **所以：默认用 `pipeline`，`parallel` 是例外。**（这是我的建议，别当成官方指导说。）

### 3.5 剧本骨架（示意）

> ⚠️ **下面是示意脚本，用于讲清结构。** 特别是 `args.approved` 那三行——
> DSH 的 workflow 钩子**没有** `approve()`。**审批门不在编排脚本里，它在运行时里**（见 §3.6）。

```js
// ── 阶段 ①：分析师（只读）─────────────────────────────
phase("① 分析师：评估与计划");

const triage = await agent(TRIAGE_PROMPT, {
  label: "分析师/评估",
  schema: {
    type: "object",
    properties: {
      type:     { type: "string", enum: ["bug", "feature", "question", "duplicate"] },
      severity: { type: "string", enum: ["P0", "P1", "P2", "P3"] },
      impact:   { type: "string" },
      uncertain:{ type: "array", items: { type: "string" } },   // 必须承认不确定项
    },
    required: ["type", "severity", "impact", "uncertain"],
    additionalProperties: false,
  },
});

const plan = await agent(PLAN_PROMPT, {
  label: "分析师/计划",
  schema: {
    type: "object",
    properties: {
      files:  { type: "array", items: { type: "string" } },
      steps:  { type: "array", items: { type: "string" } },
      verify: { type: "string" },
    },
    required: ["files", "steps", "verify"],
    additionalProperties: false,
  },
});

// ── ⭐ 硬门 ──────────────────────────────────────────
// ⚠️ 这三行是【形式】，不是强制点。真正的拒绝发生在运行时（见 §3.6）。
//    它只负责把"批准结果"带进编排；拦不拦得住，取决于审批 seam 有没有管辖下一步。
if (args.approved !== true) {
  return { stage: "awaiting-approval", triage, plan };
}

// ── 阶段 ②：实现者（可写工作区，不可推送）──────────────
phase("② 实现者：按已批准的计划开发");
const impl = await agent(IMPL_PROMPT + JSON.stringify(plan), {
  label: "实现者",
});

// ── 阶段 ③：验收者（独立上下文）────────────────────────
phase("③ 验收者：复查与验证");
const [review, verify] = await parallel([
  () => agent(REVIEW_PROMPT),   // 读 diff，不看实现者的推理
  () => agent(VERIFY_PROMPT),   // 跑测试
]);

return { triage, plan, impl, review, verify };  // 结构化返回给父级
```

### 3.6 ⭐ 门在运行时，不在脚本里（本段最重要的认知）

这是全场技术上最该讲清楚的一点：

> 脚本可以写"下一步调实现者"，
> 但**实现者拿不到"已批准"的凭据就起不来**——
> 因为门是 **`dsh-plan-mode` + `dsh-user-approval` 在运行时强制的**，不是脚本自觉遵守的。

对应关系：

| 门的职责 | 由谁承担 |
|---|---|
| 产出计划后**主动呈交人批准** | `dsh-plan-mode` 的 `exit_plan_mode` |
| 批准/拒绝是**一次性**的 | `dsh-user-approval`（每项批准只对应对应请求） |
| 没有应答者 → **拒绝**（不是通过） | `dsh-user-approval`（`unavailable` → fail-closed） |
| 每次请求与决定**留痕** | `dsh-user-approval` 写入发起会话的审计日志 |
| 只读 / 可写 / 命令白名单 | `dsh-permission-presets` · sandbox 系列 |

**一句话版本**：
> **流程的可靠性，不能建立在 agent 的自觉上。**

### 3.7 什么时候**不要**用 workflow

官方指导很明确：

> 仅当用户**明确要求工作流或大型多 agent 编排**时使用。
> **一两项委派应当直接用普通 `subagent` 调用。**

讲这一句的用途：证明你不是"手里有锤子看什么都是钉子"。**知道什么时候不用，比会用更重要。**

### 3.8 schema 的限制（被追问时用）

workflow 的 `schema` 只支持对象为根，且字段限于：
`type` · `properties` · `required` · `additionalProperties` · `items` · `enum` · `const` · `oneOf`
**不支持** `pattern` / `format` / 数值上下界。

> 用途：说明"结构化输出不是随便写 JSON Schema"，边界是明确的。

---

## 四、这一段的收尾句

> "所以 DSH 给我的不是'一个更聪明的员工'，是**一套能装配员工的零件**。
> 想清楚要几个岗位、每个岗位能碰什么、在哪一步必须停——
> 这三件事想清楚了，剩下的就是拼起来。"
