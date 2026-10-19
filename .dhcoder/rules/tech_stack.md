# 技术栈（H5Project）

> 本文件是 H5Project 技术栈的事实源。版本以 `package.json` + lockfile 为唯一事实源；
> 本文件记录**约定与选型**，实际版本以工程内依赖声明为准并保持同步。

## 目标形态

移动端 H5（Web）应用，面向手机浏览器与内嵌 WebView。

## 核心选型

| 领域 | 选型 | 说明 |
|---|---|---|
| 框架 | **Vue 3** | Composition API + `<script setup lang="ts">`，不使用 Options API |
| 语言 | **TypeScript 5.x** | 开启 `strict`；公共类型集中在 `types/` |
| UI 组件库 | **Vant 4** | 移动端组件库，按需自动引入（`unplugin-vue-components`） |
| 构建工具 | **Vite 5/6** | 开发服务器 + 生产构建；环境变量用 `.env.*` |
| 路由 | **Vue Router 4** | history 模式；路由按页面懒加载 |
| 状态管理 | **Pinia** | 组合式 store；只在需要跨页面共享时使用 |
| HTTP | **axios** | 统一实例 + 请求/响应拦截器（见 `architecture.md`） |
| 包管理 | **pnpm** | 以实际 lockfile 为准；不混用 npm/yarn 生成多份锁文件 |
| 代码质量 | ESLint + Prettier | 统一格式化与静态检查（以工程配置为准） |

## 运行环境约定

- Node.js：LTS（≥ 18）。
- 目标浏览器：移动端 Safari / Chrome / 各类 App 内置 WebView（微信、飞书等）。
- 移动端适配：优先 vw/vh 或 rem 方案，避免固定 px 造成多机型错位。

## 常用命令

```bash
pnpm install            # 安装依赖
pnpm dev                # 启动开发服务器
pnpm build              # 生产构建
pnpm preview            # 预览生产构建产物
pnpm type-check         # 类型检查（tsc --noEmit，如已配置）
pnpm lint               # ESLint
pnpm test               # 单元测试（Vitest，如已配置）
```

> 上述脚本与依赖**以 `package.json` 实际声明为准**；工程尚未落地时，本表是目标约定，落地后请回写更新。

## 版本纪律

- 依赖版本统一在 `package.json` 管理，不随意引入第二个同类库（例如不要同时用 axios 与 fetch 封装两套请求层）。
- 升级框架（Vue / Vite / Vant）属大改动，先写计划再动（见 `.dhcoder/skills/writing-plans`）。
