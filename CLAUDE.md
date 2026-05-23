# CLAUDE.md

减少 LLM 常见编码错误的行为准则。可与项目特定指令合并使用。

**权衡：** 这些准则偏向谨慎而非速度。对于简单任务，自行判断即可。

## 0. 语言要求

**所有文档使用中文描述。** 包括注释、README、CHANGELOG、提交信息等所有文档性内容。

**代码必须包含中文注释。** 每个函数/方法、类、模块顶部必须有中文注释说明其用途。关键逻辑处添加行内中文注释。

## 0.1 工作流程约束

**功能开发必须按以下 6 个阶段顺序执行，每个阶段有明确的输入、产出和交接协议：**

```
阶段1 问题发现 → 阶段2 隔离工作区 → 阶段3 架构治理 → 阶段4 实现计划 → 阶段5 TDD执行 → 阶段6 完成分支
[Superpowers]    [Superpowers]       [ArcKit]         [Superpowers]      [Superpowers]      [Superpowers]
```

### 阶段 1：问题发现（Superpowers）

使用 `superpowers:brainstorming`，**职责限定为问题空间探索，不产出完整 spec**。

- **提问前调研**：搜索相关学术论文和技术文献，补充领域知识
- **提问**：基于调研结果提出关键问题（遵循 0.3 节系统性提问约束）
- **提问后论证**：搜索最新论文和开源项目，验证方案可行性
- **产出**：问题陈述 + 选定方案方向（保存至 `docs/superpowers/specs/` 作为探索记录）
- **不产出**：完整架构设计、组件列表、数据模型（这些属于阶段 3 的职责）

> **边界约束**：Brainstorming 回答"要不要做"和"大方向选哪个"。"具体做什么"由阶段 3 的 ArcKit REQ 文档权威定义。

### 阶段 2：隔离工作区（Superpowers）

使用 `superpowers:using-git-worktrees` 创建隔离工作区。

### 阶段 3：架构治理（ArcKit）

定义"做什么、为什么、给谁看"。**按 ArcKit 层级依赖顺序执行，不可跳层或乱序：**

| 层级 | 命令 | 产出 | 依赖 |
|------|------|------|------|
| Tier 0 | `/arckit:principles` | 架构原则 | 无（项目首次时执行） |
| Tier 1 | `/arckit:stakeholders` | 干系人分析 | principles |
| Tier 2 | `/arckit:risk` | 风险评估 | stakeholders |
| Tier 4 | `/arckit:requirements` | 需求规格（BR/FR/NFR/INT/DR）| stakeholders + principles |
| Tier 6 | `/arckit:data-model` | 数据模型设计 | requirements |
| Tier 6 | `/arckit:diagram` | 架构图（C4、组件图、部署图等）| requirements |
| Tier 6 | `/arckit:adr` | 关键架构决策记录 | requirements |
| Tier 6 | `/arckit:wardley` | Wardley Map 战略分析（需要时）| requirements |
| Tier 13 | `/arckit:traceability` | 可追溯性矩阵 | 所有上游制品 |

批量生成时使用 `/arckit:build` 并行编排（仅限同层级内并行）。

- **唯一权威 Spec**：ArcKit 的 REQ 文档（`ARC-*-REQ-v*.md`）是"做什么"的唯一权威源
- **产出位置**：`projects/<project-name>/` 目录下，按 ArcKit 命名规范

### 阶段 4：实现计划（Superpowers）

使用 `superpowers:writing-plans`。

- **输入源**：阶段 3 产出的 ArcKit REQ 文档（不是阶段 1 的探索记录）
- **制品桥接**：Plan 中的每个任务必须引用对应的 ArcKit 需求 ID（如 `FR-001`、`NFR-003`），确保追溯链不断裂
- **产出**：任务级实现计划（保存至 `docs/superpowers/plans/`）

### 阶段 5：TDD 执行（Superpowers）

使用 `superpowers:test-driven-development` + `superpowers:subagent-driven-development`。

- 每个子任务按 红→绿→重构 循环执行
- 子 agent 的 prompt 中必须包含对应的 ArcKit 需求 ID，确保实现可追溯

### 阶段 6：完成分支（Superpowers）

使用 `superpowers:finishing-a-development-branch`。

- 验证所有测试通过
- 提交/合并/PR

---

**分工原则（修订版）：**

| 关注点 | 权威插件 | 另一个插件的角色 |
|--------|----------|------------------|
| 做什么（需求/范围） | ArcKit REQ | Superpowers 消费 REQ 生成 Plan |
| 为什么（动机/风险） | ArcKit（stakeholders + risk） | Superpowers brainstorming 做初步探索 |
| 怎么做（代码实现） | Superpowers（plan + TDD） | ArcKit 不介入实现 |
| 怎么隔离（分支/工作区） | Superpowers（worktree） | ArcKit 不介入 |
| 做得对不对（追溯/合规） | ArcKit（traceability） | Superpowers 在任务中引用需求 ID |

**交接协议**：阶段之间的交接通过制品文件路径实现，不依赖对话上下文。每个阶段开始时必须读取上一阶段的产出文件，而非依赖记忆。

---

**小修复豁免：** 以下情况可跳过完整流程，直接在 worktree 中修复并提交：
- 单行 bug 修复或 typo 修正
- 配置项调整（不涉及逻辑变更）
- 文档补充或修正

豁免条件：改动不超过 10 行，且不引入新依赖或新接口。

## 0.2 版本迭代约束

- 每个版本迭代不超过 **5 个功能**。
- 如果涉及界面功能，**一个界面功能单独作为一个版本**。
- 所有版本控制操作（分支、提交、合并、发布）通过 superpowers:finishing-a-development-branch 和 superpowers:using-git-worktrees 管理。
- 文档必须增加版本信息来命名

## 0.3 系统性提问约束

**禁止随机抛出 2-3 个问题。所有向用户的提问必须系统性、结构化。**

无论是 Brainstorm、需求澄清、架构设计还是任何需要向用户提问的场景，都必须遵循以下方法论：

**提问框架：5W1H 全维度覆盖**

按以下维度系统性组织问题，确保无遗漏：

| 维度 | 核心问题 | 目的 |
|------|----------|------|
| **Why（动机）** | 为什么要做？解决谁的什么痛点？不做的代价？ | 明确价值与优先级 |
| **What（范围）** | 具体做什么？边界在哪？不做什么？ | 界定范围，防止范围蔓延 |
| **Who（干系人）** | 谁用？谁受影响？谁决策？谁维护？ | 识别所有利益相关方 |
| **Where（环境）** | 在哪运行？什么平台？什么网络环境？ | 明确技术约束 |
| **When（时序）** | 什么时候用？频率？时效性要求？截止日期？ | 确定时间约束与性能需求 |
| **How（方式）** | 怎么用？怎么集成？怎么部署？怎么回滚？ | 明确实现路径与运维需求 |

**提问原则：**

1. **先分类再提问** — 将问题按维度分组呈现，而非平铺罗列，让用户看到提问的结构和完整性
2. **苏格拉底式追问** — 对用户的回答进行假设挑战："如果 X 不成立呢？""是否考虑过 Y 的情况？"
3. **标注已知与未知** — 明确哪些是你已经推断出的（列出假设），哪些是必须用户确认的
4. **区分阻塞与非阻塞** — 标记哪些问题必须立即回答才能继续，哪些可以后续补充
5. **一次问完** — 同一阶段的问题一次性系统性提出，避免反复追问造成对话碎片化

**反模式（禁止）：**

- ❌ 随机抛出几个想到的问题
- ❌ 只问表面问题，不深入追问动机和约束
- ❌ 问了 What 但漏了 Why，问了 How 但漏了 When
- ❌ 每轮只问 1-2 个问题，拖成多轮对话
- ❌ 问题之间没有逻辑关系，看不出提问结构

## 1. 先思考再编码

**不要假设。不要隐藏困惑。暴露权衡。**

实现之前：
- 明确陈述你的假设。如果不确定，就问。
- 如果存在多种解读，列出来——不要默默选一个。
- 如果有更简单的方案，说出来。必要时提出反对意见。
- 如果有不清楚的地方，停下来。指出困惑所在。提问。

## 2. 简单优先

**用最少的代码解决问题。不做投机性开发。**

- 不添加未被要求的功能。
- 不为一次性代码创建抽象。
- 不添加未被要求的"灵活性"或"可配置性"。
- 不为不可能的场景编写错误处理。
- 如果你写了 200 行但 50 行就能搞定，重写它。

问自己："一个资深工程师会说这太复杂了吗？"如果是，简化它。

## 3. 精准修改

**只改必须改的。只清理自己造成的混乱。**

编辑现有代码时：
- 不要"改进"相邻的代码、注释或格式。
- 不要重构没有问题的东西。
- 匹配现有风格，即使你会用不同方式。
- 如果注意到不相关的死代码，提一下——不要删除它。

当你的修改产生孤立代码时：
- 删除你的修改导致不再使用的 import/变量/函数。
- 不要删除已有的死代码，除非被要求。

检验标准：每一行改动都应该直接追溯到用户的请求。

## 4. 目标驱动执行

**定义成功标准。循环直到验证通过。**

将任务转化为可验证的目标：
- "添加验证" → "为无效输入编写测试，然后让测试通过"
- "修复 bug" → "编写复现它的测试，然后让测试通过"
- "重构 X" → "确保重构前后测试都通过"

对于多步骤任务，列出简要计划：
```
1. [步骤] → 验证：[检查项]
2. [步骤] → 验证：[检查项]
3. [步骤] → 验证：[检查项]
```

强成功标准让你能独立循环。弱标准（"让它能跑"）需要不断澄清。

## CodeGraph

CodeGraph builds a semantic knowledge graph of codebases for faster, smarter code exploration.

### If `.codegraph/` exists in the project

**NEVER call `codegraph_explore` or `codegraph_context` directly in the main session.** These tools return large amounts of source code that fills up main session context. Instead, ALWAYS spawn an Explore agent for any exploration question (e.g., "how does X work?", "explain the Y system", "where is Z implemented?").

**When spawning Explore agents**, include this instruction in the prompt:

> This project has CodeGraph initialized (.codegraph/ exists). Use `codegraph_explore` as your PRIMARY tool — it returns full source code sections from all relevant files in one call.
>
> **Rules:**
> 1. Follow the explore call budget in the `codegraph_explore` tool description — it scales automatically based on project size.
> 2. Do NOT re-read files that codegraph_explore already returned source code for. The source sections are complete and authoritative.
> 3. Only fall back to grep/glob/read for files listed under "Additional relevant files" if you need more detail, or if codegraph returned no results.

**The main session may only use these lightweight tools directly** (for targeted lookups before making edits, not for exploration):

| Tool | Use For |
|------|---------|
| `codegraph_search` | Find symbols by name |
| `codegraph_callers` / `codegraph_callees` | Trace call flow |
| `codegraph_impact` | Check what's affected before editing |
| `codegraph_node` | Get a single symbol's details |

### If `.codegraph/` does NOT exist

At the start of a session, ask the user if they'd like to initialize CodeGraph:

"I notice this project doesn't have CodeGraph initialized. Would you like me to run `codegraph init -i` to build a code knowledge graph?"

---

**这些准则生效的标志：** diff 中不必要的改动更少，因过度复杂化导致的重写更少，澄清性问题出现在实现之前而非犯错之后。
