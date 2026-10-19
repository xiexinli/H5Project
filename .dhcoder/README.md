# H5Project AI 配置说明

> 本文档说明 H5Project 中 AI 助手配置的**每一个文件**：作用、生效时机、搭配关系与注意事项。
> 结构参照 AndroidBase 的 `.dhcoder/`，并按本工程（Vue 3 + TypeScript + Vant 4 + Vite 的移动端 H5）重写。

## 0. 先读这里：生效模型（重要）

这些文件**不是会被编译或自动执行的代码**，它们的「生效」= **AI 助手在合适的时机读取它们**。因此：

1. **发现目录取决于客户端**：
   - 本工程的权威 skill 源目录是 **`.dhcoder/skills/`**。
   - `.claude/skills/` 与 `.agents/skills/` 已配置为指向 `.dhcoder/skills/` 的软链，分别供 Claude Code 与 Codex 发现；不要复制出第二份 skill 源。
   - 新增或移动 skill 后，确认两个软链入口仍可访问该 skill，并更新根 `AGENTS.md` 的路由。
2. **入口唯一**：所有路由都从根目录 `AGENTS.md` 出发，它定义「什么时候读哪个文件」。
3. **懒加载**：`rules/` 在改代码前读；`skills/<x>/SKILL.md` 在任务命中触发词时读；skill 的 `references/`、`scripts/`、`templates/` 只在 SKILL.md 里按需引用时才读。

---

## 1. 根目录入口文件

| 文件 | 作用 | 生效时机 | 搭配 |
|---|---|---|---|
| `AGENTS.md` | AI 的总入口与唯一事实源：项目概览、技术栈、目录、构建命令、架构约束、**AI 工作流**（会话启动顺序 / 任务分级 L0–L3 / 执行闭环 / 必须遵守 / 严禁 / 输出要求 / Skill 路由）、评审入口、事实源索引 | 会话开始即读取；任务过程中反复回查 | `.dhcoder/rules/`、`.dhcoder/skills/`、`REVIEW.md` 的索引入口 |
| `CLAUDE.md` | Claude Code 的桥接文件，只指向 `AGENTS.md` | 用 Claude Code 打开工程时 | 保持极薄，避免出现两份规则 |
| `REVIEW.md` | 评审任务入口：评审默认只读、分层检查、验证升级路径、相关资产 | 任务为 review / diff review / MR review 时 | 承接 `AGENTS.md` 评审路由，指向 `web-review` |
| `README.md` | 工程对外说明 | 人阅读/上手时 | 不参与 AI 路由 |

---

## 2. `.dhcoder/config.yaml`

| 项 | 内容 |
|---|---|
| 作用 | 声明 AI 输出的语言约定：面向用户的评审文案用简体中文，路径/标识符/枚举值保持原样 |
| 生效时机 | 工具读取仓库配置时；主要影响 review 类输出的语言 |
| 搭配 | 与 `AGENTS.md`、`web-review` 配合；是「输出语言」的事实源 |
| 备注 | 目前仅含 `output.language: zh-Hans` |

---

## 3. `.dhcoder/rules/`（编码前必读）

| 文件 | 作用 | 生效时机 | 搭配 |
|---|---|---|---|
| `tech_stack.md` | 本工程技术栈速览：Vue 3 / TS / Vant 4 / Vite / 路由 / Pinia / axios / pnpm 与运行环境约定、常用命令 | 修改代码前（`AGENTS.md` 会话启动第 2 步） | 与 `package.json` 配套；版本以依赖声明为准 |
| `architecture.md` | 分层与依赖方向（`views → components → composables/stores → api/services → utils/types`）、网络封装、状态、路由、适配、禁止事项 | 写业务代码、做架构判断前 | 与 `AGENTS.md` 架构约束一节互补；评审时由 `web-review` 引用 |
| `code-style.md` | TS / Vue 组件 / 命名 / 控制流与体量 / 异步与副作用 / 格式工具，以及本项目硬约定 | 编码前 / 评审时 | 与 `web-native-dev` skill 配合（冲突以 skill 为准） |

---

## 4. `.dhcoder/skills/` — 通用工作流 Skills

> **所有 skill 的统一规则**：`SKILL.md` 是入口，其 `references/`、`scripts/`、`templates/` 只有被 SKILL.md 明确引用时才加载。
> 「生效时机」列指的是**触发该 skill 的任务类型**。

### 4.1 开发/排障工作流

| 文件 | 作用 | 生效时机 | 搭配 |
|---|---|---|---|
| `diagnose/SKILL.md` | 硬 Bug / 性能回退的纪律化诊断闭环：建反馈环 → 复现 → 假设 → 插桩 → 修复 → 回归 | 用户报 Bug、报错、白屏、跑不起来、性能退化 | 与 `tdd`、`improve-codebase-architecture` 搭配 |
| `diagnose/scripts/hitl-loop.template.sh` | 人工介入（HITL）复现循环脚本模板 | 无法自动复现、必须人工点按时 | 由 `diagnose` 引用 |
| `tdd/SKILL.md` | 红-绿-重构的 TDD 闭环；反对「一次写完所有测试」 | 要求测试先行、red-green-refactor、集成测试 | 主入口；按需引用其附件 |
| `tdd/tests.md`、`mocking.md`、`deep-modules.md`、`interface-design.md`、`refactoring.md` | 测试判定、mock 边界、深模块、可测接口设计、重构候选 | 写/设计测试与接口时 | 由 `tdd/SKILL.md` 引用 |

### 4.2 架构与全局认知

| 文件 | 作用 | 生效时机 | 搭配 |
|---|---|---|---|
| `zoom-out/SKILL.md` | 让 AI「升一层抽象」，用领域词汇画出相关模块与调用者地图 | 对该区域代码不熟、需要全局结构时 | 独立使用 |
| `improve-codebase-architecture/SKILL.md`（+ `DEEPENING.md`、`INTERFACE-DESIGN.md`、`LANGUAGE.md`） | 寻找「深化」机会：把紧耦合的浅模块合并/加深，提升可测性与可导航性 | 明确要做架构优化、解耦、重构 | 依赖 `CONTEXT.md` 与 `docs/adr/` |
| `codebase-design/SKILL.md` | 设计深模块与简洁接口，检查接缝、依赖、局部性和测试面；遵守本项目分层与 Pinia/axios 约定 | 设计/重构模块接口、提升可测试性 | 与 `improve-codebase-architecture`、`tdd` 互补 |
| `domain-modeling/SKILL.md` | 澄清领域术语、概念关系与边界场景，并把已确认的语言/重要权衡沉淀到文档 | 讨论领域模型、术语冲突，或形成架构决策时 | 沿用 `grill-with-docs` 的格式 |
| `prototype/SKILL.md`（+ `LOGIC.md`、`UI.md`） | 一次性原型：动手前验证设计——逻辑走终端小应用，UI 走多方案切换页 | 需求不稳定、想先做可运行草案 | 产物「用完即弃」 |

### 4.3 需求澄清与规划

| 文件 | 作用 | 生效时机 | 搭配 |
|---|---|---|---|
| `prd-to-spec/SKILL.md` | 把 PRD 转成可据以施工的技术 Spec（需求澄清 → 功能/流程/数据/接口/异常/验收） | 拿到产品需求、写实现计划或动代码之前 | 下一步接 `writing-plans`；Spec 存 `docs/spec/YYYY-MM-DD-<feature>.md` |
| `writing-plans/SKILL.md`（+ `plan-document-reviewer-prompt.md`） | 把 Spec 拆成 bite-sized 的实现计划 | 有 Spec/需求、要落地多步任务时 | 接 `prd-to-spec`；计划存 `docs/superpowers/plans/` |
| `grill-me/SKILL.md` | 就一个方案/设计对用户穷追猛问，直到达成共识 | 用户想「拷问」自己的方案 | 独立使用 |
| `grill-with-docs/SKILL.md`（+ `ADR-FORMAT.md`、`CONTEXT-FORMAT.md`） | 结合既有领域模型拷问方案，并**就地更新** `CONTEXT.md` 与 ADR | 需要方案对照项目词汇与已记录决策 | 决策落文档 |
| `app-grill-pm/SKILL.md`（+ 两个 references） | 面向 App/前端的「PM 拷问」：实施前澄清产品行为、用户流程、设计边界、验收标准 | PRD/功能简述在实施前需要产品澄清 | 只澄清产品语义，不出技术方案 |
| `prd-code-impact-architect/SKILL.md`（+ 三个 templates） | PRD 代码影响分析子 skill：输出代码落点、模块边界、方案草稿、可执行验收点 | 只需 PRD 代码影响面分析时独立使用 | 上游 `prd-to-app-spec` 未移植 |

### 4.4 任务拆解与分流

| 文件 | 作用 | 生效时机 | 搭配 |
|---|---|---|---|
| `to-prd/SKILL.md` | 把当前对话上下文**合成**成 PRD，并发布到 issue tracker | 想把讨论沉淀成 PRD 时 | 依赖 `setup-matt-pocock-skills` |
| `to-issues/SKILL.md` | 把计划/PRD 拆成可独立认领的纵向切片 issue | 想把计划拆成工单时 | 依赖 tracker 配置 |
| `triage/SKILL.md`（+ `AGENT-BRIEF.md`、`OUT-OF-SCOPE.md`） | 用状态机给 issue 分流 | 建 issue、审 bug/需求、准备给 Agent 时 | 依赖 tracker 配置 |
| `caveman/SKILL.md` | 极简输出模式：砍掉填充词，token 约省 75% | 用户明确要求极简输出时 | 独立使用 |

### 4.5 文件化任务管理

| 文件 | 作用 | 生效时机 | 搭配 |
|---|---|---|---|
| `planning-with-files/SKILL.md`（+ `templates/`、`scripts/`、`examples.md`、`reference.md`） | Manus 风格文件化规划：用 `task_plan.md` / `findings.md` / `progress.md` 承载任务上下文 | 多步任务、研究型任务、预计 >5 次工具调用时 | 支持 `/clear` 后恢复 |

### 4.6 首次配置

| 文件 | 作用 | 生效时机 | 搭配 |
|---|---|---|---|
| `setup-matt-pocock-skills/SKILL.md`（+ `domain.md`、`issue-tracker-*.md`、`triage-labels.md`） | 首次接入：写入 `## Agent skills` 块并落 tracker/标签/领域文档布局 | 首次使用 `to-issues`/`to-prd`/`triage`/`diagnose`/`tdd` 等之前 | ⚠️ **会修改 `AGENTS.md`/`CLAUDE.md`**，运行前先确认 |

### 4.7 按需外部研究 / 复盘

| 文件 | 作用 | 生效时机 |
|---|---|---|
| `research/SKILL.md` | 面向需要外部证据的问题，优先查一手来源、交叉核验并注明引用 | 用户要求研究/查证/对比，或答案依赖外部、时效性事实时 |
| `retro/SKILL.md` | 基于任务证据复盘 Agent 工作流，提出改进建议 | 用户要求复盘，或较大任务出现可复用摩擦时 |

---

## 5. `.dhcoder/skills/` — 前端专项 Skills

| 文件 | 作用 | 生效时机 | 搭配 |
|---|---|---|---|
| `web-native-dev/SKILL.md` | 本工程的 H5 开发总纲：仓库事实、默认验证路径、TS/Vue 规范、Vant 与移动端适配、构建报错速查、测试分层 | H5 开发、功能实现、重构、构建排障 | 必须先读 `references/project-facts.md` |
| `web-native-dev/references/project-facts.md` | **本工程的仓库事实**（形态、目录、依赖、验证命令、约定与坑） | 用 `web-native-dev` 时最先读 | 事实与通用建议冲突时以本文件为准 |
| `web-verify/SKILL.md` | 验证工作流：先看 diff → 选最小验证 → 跑精确命令 → 结构化汇报（Verified/Blocked/Noise/Risk） | 需要类型检查、构建、测试验证时 | 评审时与 `web-review` 合用；Bug 时与 `diagnose` 合用 |
| `web-verify/references/task-map.md` | 本工程验证命令地图与常见失败模式 | 选验证命令时 | ⚠️ 只适用本工程，换工程要重写 |
| `web-review/SKILL.md` | 通用评审流程：定范围 → 先看 diff → 按需扩上下文 → 读规则 → 输出高信号发现 | diff review / MR review / 自动化评审 | 读 `REVIEW.md` 与 `.dhcoder/rules/`；需要证据时调 `web-verify` |

---

## 6. 组合用法（推荐路径）

```
① 需求接入
   prd-to-spec ──► writing-plans ──► 实施（web-native-dev）
   （产品语义不清时先走 app-grill-pm；方案要拷问时走 grill-me / grill-with-docs）

② 日常改代码
   AGENTS.md → .dhcoder/rules/{tech_stack,architecture,code-style}.md → 编码 → web-verify

③ 排障
   diagnose（＋ web-verify 取证据；发现架构阻碍转 improve-codebase-architecture）

④ 评审
   REVIEW.md → web-review（＋ web-verify；＋ .dhcoder/rules/）

⑤ 多步/长任务
   planning-with-files（task_plan / findings / progress 三文件）

⑥ 任务拆解与分流
   to-prd / to-issues / triage（首次需先跑 setup-matt-pocock-skills 配置 tracker 与标签）

⑦ 外部技术调研（按需） / ⑧ 工作流复盘（按需）
   research / retro
```

---

## 7. 与 AndroidBase 的对应关系

| H5Project | AndroidBase 对应 |
|---|---|
| `.dhcoder/rules/tech_stack.md` | 同名（内容改写为 Vue 3 + Vant + Vite） |
| `.dhcoder/rules/architecture.md` | 同名（改写为前端分层） |
| `.dhcoder/rules/code-style.md` | 同名（改写为 TS/Vue 风格） |
| `.dhcoder/config.yaml` | 照搬 |
| `.dhcoder/skills/<通用 skill>/` | 照搬（`codebase-design`、`domain-modeling`、`prd-to-spec`、`research` 已去 Android 化） |
| `.dhcoder/skills/web-native-dev/` | 由 `android-native-dev` 改写 |
| `.dhcoder/skills/web-verify/` | 由 `android-verify` 改写 |
| `.dhcoder/skills/web-review/` | 由 `android-review` 改写 |
| `REVIEW.md`、`AGENTS.md`、`CLAUDE.md` | 由 AndroidBase 版本改写 |

**未移植**（Android 专属，前端不需要）：`android-native-dev`、`android-verify`、`android-review` 及其 Android 专属 references；Android 相关的 build/Gradle 命令与模块结构。

---

## 8. 维护指南

- **新增 skill**：建 `.dhcoder/skills/<name>/SKILL.md`，frontmatter 至少含 `name` 与 `description`；在 `AGENTS.md` 的「Skill 分层与路由」里登记触发规则。
- **改名/移动**：同步更新 `AGENTS.md`、`REVIEW.md` 与其它 skill 中对它的引用。
- **规则变更**：优先改 `.dhcoder/rules/` 或对应 skill，再回写 `AGENTS.md` 的索引。
- **脚手架落地后**：用实际 `package.json` 的脚本与依赖回写 `tech_stack.md`、`AGENTS.md` 与 `web-native-dev/references/project-facts.md` 的目标形态描述。
- **Skill 发现入口**：`.claude/skills` 与 `.agents/skills` 是指向 `.dhcoder/skills` 的软链（见 §0）。新增或移动 skill 后确认两个入口仍能解析到它。
