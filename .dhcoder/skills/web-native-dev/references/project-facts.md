# H5Project 仓库事实

> 本文件是使用 `web-native-dev` skill 时**最先读**的文件。
> 事实与通用建议冲突时，以本文件为准。

## 工程形态

- 移动端 H5 应用（Vue 3 + TypeScript + Vant 4 + Vite）。
- ⚠️ 当前仓库处于**初始化阶段**：可能尚未包含 `package.json` / `src/` / `vite.config.ts`。
  在实现前先确认脚手架已建立；本文件描述**目标形态**，脚手架落地后请据实回写。
- 入口：`index.html` → `src/main.ts`；挂载根组件 `src/App.vue`。

## 目标目录结构

```text
H5Project/
├── index.html
├── package.json
├── vite.config.ts
├── tsconfig.json
├── .env.development / .env.production
├── src/
│   ├── main.ts                 # 应用入口（挂载、插件注册）
│   ├── App.vue                 # 根组件
│   ├── router/                 # Vue Router 4 路由与守卫
│   ├── stores/                 # Pinia store（组合式）
│   ├── api/                    # 统一 axios 实例 + 接口模块（唯一请求层）
│   ├── services/               # 业务服务/数据编排（可选，与 api 配合）
│   ├── composables/            # 可复用组合式函数（useXxx）
│   ├── components/             # 全局/通用展示组件
│   ├── views/                  # 页面级组件（按路由组织）
│   ├── utils/                  # 纯工具函数（不依赖 Vue/Vant）
│   ├── types/                  # 公共 TS 类型
│   └── assets/                 # 静态资源
└── .dhcoder/                   # AI 协作配置
```

> 目录命名与职责以 `.dhcoder/rules/architecture.md` 为准。

## 版本与依赖

- 版本以 `package.json` + lockfile 为唯一事实源（见 `.dhcoder/rules/tech_stack.md`）。
- 包管理统一 pnpm，不混用 npm/yarn。

## 验证命令

| 目的 | 命令 |
|---|---|
| 安装依赖 | `pnpm install` |
| 开发服务 | `pnpm dev` |
| 类型检查 | `pnpm type-check` |
| 静态检查 | `pnpm lint` |
| 生产构建 | `pnpm build` |
| 预览产物 | `pnpm preview` |
| 单元测试 | `pnpm test` |

> 以上脚本名以 `package.json` 实际声明为准；未配置的项在脚手架阶段补齐。

## 约定与坑

- 别名 `@/` → `src/`，需在 `vite.config.ts` 与 `tsconfig.json` 的 `paths` 同步配置。
- 环境变量仅 `VITE_` 前缀暴露给客户端；接口地址走 `.env.*`，禁止硬编码。
- Vant 按需引入由 `unplugin-vue-components` 自动完成，避免手动全局注册重复。
- 移动端真机/WebView 与桌面浏览器存在差异，UI 结论以真机为准。
- 构建产物部署在子路径时，需设置 Vite 的 `base`，否则白屏/资源 404。

## 与其他资产的关系

- 分层约束：`.dhcoder/rules/architecture.md`
- 编码风格：`.dhcoder/rules/code-style.md`
- 验证工作流：`.dhcoder/skills/web-verify`
- 评审工作流：`.dhcoder/skills/web-review`
