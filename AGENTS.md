# H5Project — AI 开发指南

> 本文件是给 AI 编程助手（Claude Code、Codex、Cursor 等）的项目上下文。
> 开始在本仓库写代码之前，请先完整阅读本文件。

## 0. 事实源与 AI 资产索引

| 信息 | 唯一事实源 |
|---|---|
| 项目开发约定（本文件） | `AGENTS.md` |
| AI 配置总说明 | `.dhcoder/README.md` |
| 技术栈 | `.dhcoder/rules/tech_stack.md`、`package.json` |
| 架构与分层边界 | `.dhcoder/rules/architecture.md` |
| 编码风格 | `.dhcoder/rules/code-style.md` |
| Skills（通用工作流 + 前端专项） | `.dhcoder/skills/`（`SKILL.md` 为入口） |
| Skill 发现入口（软链到 `.dhcoder/skills/`） | `.claude/skills/`（Claude Code）、`.agents/skills/`（Codex） |
| 需求 / 计划产物 | `docs/spec/`（Spec）、`docs/superpowers/plans/`（实现计划） |
| 评审入口 | `REVIEW.md` |
| 输出语言 | `.dhcoder/config.yaml`（简体中文） |

> `AGENTS.md` 是任务路由与上下文披露顺序的唯一入口；调整流程时优先改本文件。

## 项目概览

面向移动端浏览器与 App 内嵌 WebView 的 **H5 应用**，技术栈为 **Vue 3 + TypeScript + Vant 4 + Vite**。
工程处于初始化阶段，脚手架落地后请据实回写本文件与 `.dhcoder/rules/tech_stack.md`。
分层遵循 `视图 → 组件 → 组合式/状态 → 数据访问 → 纯逻辑/类型` 的单向依赖。

## 技术栈（版本以 `package.json` + lockfile 为唯一事实源）

- **Vue 3**（Composition API + `<script setup lang="ts">`）
- **TypeScript 5.x**（`strict`）
- **Vant 4**（移动端组件库，按需自动引入）
- **Vite 5/6**（构建工具，环境变量 `VITE_` 前缀）
- **Vue Router 4**（history 模式，页面懒加载）
- **Pinia**（组合式 store）
- **axios**（`api/` 层统一封装）
- **pnpm**（包管理）
- 目标环境：移动端 Safari / Chrome / WebView（Node LTS ≥ 18）

## 目录结构（目标形态）

```text
H5Project/
├── index.html
├── vite.config.ts              # 别名 @/ → src/、插件、代理、base
├── tsconfig.json               # paths 与 vite 别名保持一致
├── .env.development / .env.production
├── src/
│   ├── main.ts                 # 应用入口
│   ├── App.vue
│   ├── router/                 # 路由与守卫
│   ├── stores/                 # Pinia store
│   ├── api/                    # axios 实例 + 接口模块（唯一请求层）
│   ├── services/               # 业务服务/数据编排
│   ├── composables/            # useXxx 组合式函数
│   ├── components/             # 通用展示组件
│   ├── views/                  # 页面级组件
│   ├── utils/                  # 纯工具（不依赖 Vue/Vant）
│   ├── types/                  # 公共类型
│   └── assets/
└── .dhcoder/                   # AI 协作配置
```

## 构建 / 测试命令

```bash
pnpm install          # 安装依赖
pnpm dev              # 开发服务器
pnpm type-check       # 类型检查
pnpm lint             # ESLint
pnpm build            # 生产构建
pnpm preview          # 预览产物
pnpm test             # 单元测试（Vitest）
```
- 命令以 `package.json` 的 `scripts` 为准；未配置的项在脚手架阶段补齐。
- 依赖版本集中在 `package.json`，不在多处散写版本；不混用包管理器。
- 别名 `@/` → `src/`，`vite.config.ts` 与 `tsconfig.json` 需同步。

## 架构约束（新增代码必须遵守）

1. **依赖方向**：`views/` → `components/` → `composables/`、`stores/` → `api/`、`services/` → `utils/`、`types/`；禁止反向依赖与跨层直连。
2. **请求层唯一**：只有 `api/` / `services/` 可发起 HTTP；视图层不得直接调用 axios 或拼接请求。所有请求走统一 axios 实例（baseURL、超时、拦截器）。
3. **UI 组件**：一律用 Vant 4；主题用 CSS 变量定制，不覆盖组件内部类名。
4. **状态管理**：Pinia 组合式 store；只在跨页面共享或需持久化时使用，页面局部状态留在组件内。
5. **路由**：页面级懒加载；鉴权走统一守卫。
6. **移动端适配**：尺寸走适配方案（vw/rem），处理安全区 `env(safe-area-inset-*)`。
7. **类型**：`strict` 开启，不用 `any`；接口响应字段按可能缺失处理。
8. **环境变量**：仅 `VITE_` 前缀暴露；接口地址走 `.env.*`，不硬编码。

## 关键实现要点与坑

- **别名不一致**：`vite.config.ts` 与 `tsconfig.json` 的 paths 必须一致，否则 IDE 报错或构建失败。
- **白屏**：多为运行时错误或 `base`/资源路径问题；先看浏览器控制台。
- **Vant 按需引入**：由 `unplugin-vue-components` 自动完成，避免手动全局注册重复。
- **真机差异**：WebView / 移动端 Safari 与桌面浏览器不一致，UI 结论以真机为准。
- **锁文件**：统一 pnpm，删除多余 lockfile，避免依赖漂移。

## 测试

- 单元/组件测试用 **Vitest**（+ `@vue/test-utils`）；优先测纯逻辑与组件行为。
- 用户明确要求测试时再补测试依赖与用例。
- E2E（如 Playwright）按需接入，不作为默认。

## AI 工作流

### 会话启动顺序（Just-in-Time，禁止无目的加载整个仓库）

1. 读取本文件 `AGENTS.md`。
2. 读取 `.dhcoder/rules/` 下与当前任务相关的规则（改代码前：`tech_stack.md`、`architecture.md`、`code-style.md`）。
3. 按任务关键词读取命中的 `.dhcoder/skills/<skill>/SKILL.md`，再读其按需路由的 references。
4. 代码评审前额外读取 `REVIEW.md`。

### 任务分级

| 等级 | 场景 | 流程 |
|---|---|---|
| L0 | 问答、只读分析、状态查询，无任何实现性副作用 | 只给结论，不改文件 |
| L1 | 低风险局部小改（1–3 个同目录文件，不改对外接口/数据/路由） | 会话内 3–5 行 mini-spec → 实施 → Self Check |
| L2 | 多文件、多步骤或需要方案权衡 | 先 Spec（`prd-to-spec`）→ 用户确认 → 分步计划（`writing-plans`）→ 实施 → 验证 |
| L3 | 核心链路或大范围改造 | L2 流程 + 一次独立评审（`web-review`） |

- 需求有多种解释、涉及核心链路或显著外部影响时，不论等级都必须先与用户确认。
- 分级只决定工作流，不替代文件范围、破坏性操作或权限授权。

### 执行闭环

1. 先做只读审查，再做最小闭环改动。
2. 编码前说清假设和不确定项；关键术语不清或存在多种合理解释时，先澄清领域语言与边界场景。
3. 小步、刻意推进，尽早建立可观察反馈环；按接口契约验证行为，避免一次铺开大量未经验证的改动。
4. 设计时关注模块深度、接口简洁性与变化局部性；抽象必须解决真实复杂度，不为追求形式而增加层次。
5. 最小实现，不预埋未来扩展，不顺手重构无关内容。
6. 改完必须自检并给出验证证据（命令 + 结果），不能只写“已验证”。

### 必须遵守

- 新增或修改代码必须遵守 `.dhcoder/rules/` 中的架构与编码约束。
- 只做用户明确要求，只改与任务直接相关的代码。
- 完成时记录验证证据：命令、结果、失败原因或可观测行为。

### 严禁事项

- 未经明确要求做大规模重构或跨模块迁移。
- 绕过统一架构自建并行路径（第二套请求层/状态管理/UI 组件库）。
- 生产代码捏造数据或混入 mock/临时调试入口。
- 修改生成文件或依赖锁文件（除非用户明确要求）。

### 输出要求（L1 及以上）

1. 改动摘要
2. 受影响文件
3. 验证情况
4. 风险与后续建议
5. 本次 Skill 复用/新增情况（无则写“无”）

### Skill 分层与路由

- Skills 位于 `.dhcoder/skills/<name>/SKILL.md`；同名时项目 Skill 优先。
- 选定 Skill 后必须在执行前完整读取其 `SKILL.md`。
- 通用 Skill 触发规则：
  - `diagnose`：出现 Bug、报错、失败或性能回退。
  - `tdd`：要求测试先行，或功能/修复需要先写测试。
  - `to-prd` / `to-issues`：把讨论沉淀为 PRD，或拆成可独立执行的任务。
  - `triage`：对需求/缺陷做状态分流。
  - `zoom-out`：需要先看全局结构与模块关系。
  - `prototype`：需求不稳定，先做可运行草案验证方向。
  - `improve-codebase-architecture`：明确要做架构优化或解耦；用于扫描并提出深化候选。
  - `codebase-design`：设计/重构模块接口，提升可测试性/可导航性，或扫描候选已选定后比较设计方案。
  - `domain-modeling`：澄清领域术语、概念关系和边界场景，或需要沉淀已确认的领域语言/重要架构决策；仅阅读现有术语时无需启用。
  - `research`：用户明确要求调研/查证/对比，或结论依赖外部事实、可信来源、时效性信息时启用；简单问答、只读本仓库、按既有 Spec 实现时不启用。仅复杂、可复用或用户要求留档时保存带引用的研究文档。
  - `retro`：用户明确要求复盘，或较大开发/排障/评审任务中出现明显且可复用的 Agent 工作流摩擦时启用；简单问答、小型顺利任务不触发。只基于可观察证据提出建议，不自动修改配置。
  - `prd-to-spec` → `writing-plans`：需求 → 技术 Spec → 分步实现计划。
  - `grill-me` / `grill-with-docs`：需要高强度审视方案与边界。
  - `caveman`：用户明确要求极简输出。
  - `planning-with-files`：多步或研究型任务，或预计 >5 次工具调用、需要跨会话恢复时。
  - `app-grill-pm`：PRD/功能简述在实施前需要产品行为、流程与验收澄清时。
  - `prd-code-impact-architect`：只需 PRD 代码影响面分析、不做正式 Spec 也不实施代码时。
  - 前端专项：`web-native-dev`（开发/排障）、`web-verify`（类型检查/构建/测试/验证）、`web-review`（评审）。
- `to-prd` / `to-issues` / `triage` 首次使用前先运行 `setup-matt-pocock-skills` 配置 issue tracker 与标签；该 skill 会修改 `AGENTS.md` / `CLAUDE.md`，运行前先确认。

## 代码评审

评审默认只读；任务为 review / diff review / MR review 时，先读 `REVIEW.md`，再按 `.dhcoder/skills/web-review/SKILL.md` 执行。

## 变更纪律

- 提交信息遵循 Conventional Commits（`feat:` / `fix:` / `refactor:` / `docs:` / `test:`）。
- 大改动先写实现计划（见 `.dhcoder/skills/writing-plans`），再动代码。
- 需求级改动：先 PRD → 技术 Spec（见 `.dhcoder/skills/prd-to-spec`），再计划，再实现。
