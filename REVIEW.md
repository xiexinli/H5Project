# 评审入口（H5Project）

本文件是评审任务的入口。评审默认**只读**。

## 何时读本文件

任务类型为 review / diff review / MR review / 交互式评审时，先读本文件，再按 `.dhcoder/skills/web-review/SKILL.md` 执行。

## 评审原则

- 先看 diff 定义范围，再从仓库扩大上下文；不要从全仓漫游开始。
- 只输出高信号发现，锚定到具体文件、函数、符号或代码路径。
- 严重发现需有证据；无发现时直接说明，不凑数。
- 需要类型/构建/测试证据时，用 `.dhcoder/skills/web-verify`。

## 分层检查（先读规则）

判断前先读 `.dhcoder/rules/architecture.md` 与 `code-style.md`，重点核对：

- `views/` 是否绕过 `api/` 层直接发请求。
- `composables/`、`stores/` 是否越界操作 DOM/组件内部。
- `utils/`、`types/` 是否引入了 Vue / Vant / DOM 依赖。
- 是否引入了与既有方案并行的第二套请求层/状态管理/UI 组件库。
- 类型是否用了 `any` 掩盖问题；响应字段是否按可能缺失处理。

## 验证升级路径

1. `git diff --check`
2. `pnpm type-check`
3. `pnpm lint`
4. 相关单元测试 `pnpm test`
5. 运行期/视觉行为无法静态证明时，写人工验证说明

证据缺失时说明缺口，不提高严重度。

## 相关资产

- 规则：`.dhcoder/rules/{architecture,code-style,tech_stack}.md`
- Skill：`.dhcoder/skills/web-review`、`.dhcoder/skills/web-verify`
- 总入口：`AGENTS.md`
