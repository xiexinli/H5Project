# H5Project 验证命令地图

> ⚠️ 本文件只适用本仓库，换工程需重写。
> 命令名以 `package.json` 的 `scripts` 为准；下方为约定命名。

## 命令地图

| 目的 | 命令 | 何时用 |
|---|---|---|
| 安装依赖 | `pnpm install` | 首次或依赖变更后 |
| 类型检查 | `pnpm type-check` | 改动 TS 逻辑/类型/组件后（首选快速证据） |
| 静态检查 | `pnpm lint` | 改动源码后 |
| 单元测试 | `pnpm test` | 改动被测逻辑（utils/composable/store） |
| 单文件测试 | `pnpm test -- <path>` | 只验证某个测试文件 |
| 生产构建 | `pnpm build` | 类型通过但需证明产物可构建；改动构建配置时 |
| 预览产物 | `pnpm preview` | 需在真实产物上做运行期/真机验证 |
| diff 检查 | `git diff --check` | 任何改动，先说空白/冲突问题 |

## 选择策略

- **只改纯逻辑/类型**：`pnpm type-check`（+ 有测试则 `pnpm test`）。
- **改组件模板/样式**：`pnpm type-check` + `pnpm build`；视觉结论需浏览器/真机。
- **改构建配置/依赖**：`pnpm build` 必跑。
- **改接口层**：`pnpm type-check` + 相关单测；如有 mock，跑对应测试。

## 常见失败模式

| 现象 | 可能原因 | 处理 |
|---|---|---|
| `Cannot find module` | 依赖未安装 / 别名未配置 | `pnpm install`；检查 `vite.config.ts` + `tsconfig` paths |
| 类型错误集中在某文件 | 该文件类型签名变更未同步调用方 | 修正调用方，勿用 `any` 掩盖 |
| `pnpm build` 成功但白屏 | `base` 配置 / 资源路径 / 运行期错误 | 看浏览器控制台；检查部署路径 |
| Lint 大量既有告警 | 非本次 diff 引入 | 视为噪声，不升级为发现 |
| 依赖冲突/锁文件不一致 | 混用包管理器 | 统一 pnpm，删除多余 lockfile 后重装 |
