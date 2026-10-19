# 编码风格（H5Project）

> 编码前与评审时遵循；与 `web-native-dev` skill 冲突时以 skill 为准。

## TypeScript

- 开启 `strict`；不用 `any`，确需逃逸用 `unknown` + 类型收窄。
- 公共数据结构集中定义在 `types/`，组件间共享类型不重复声明。
- 接口响应字段按可能缺失处理（可选字段或 `| null`），消费前判空。
- 优先 `const`/`let`，不用 `var`；使用可选链 `?.` 与空值合并 `??`。

## Vue 组件

- 统一 `<script setup lang="ts">` + Composition API，不用 Options API。
- 组件文件命名 **PascalCase**（`UserCard.vue`）；目录名 **kebab-case**（`user-profile/`）。
- 单文件组件顺序：`<script setup>` → `<template>` → `<style scoped>`。
- props 用类型声明（`defineProps<{...}>()`），emits 用 `defineEmits<{...}>()`。
- 样式默认 `scoped`；跨组件公共样式抽到全局样式文件，不写深层选择的样式穿透除非必要。

## 命名约定

| 类型 | 约定 | 示例 |
|---|---|---|
| 组件 | PascalCase | `UserCard` |
| composable | `use` + PascalCase | `useUserProfile` |
| store | `use` + 名字 + `Store` | `useCartStore` |
| 变量/函数 | camelCase | `fetchList`、`isLoading` |
| 常量 | SCREAMING_SNAKE 或 camelCase 常量对象 | `MAX_RETRY` |
| CSS 类 | kebab-case | `.user-card` |
| 文件（非组件） | kebab-case | `request.ts`、`use-cart.ts` |

## 控制流与体量

- 提早返回，减少嵌套；避免超过 3 层的 `if` 嵌套。
- 单个 `.vue` 文件建议 ≤ 300 行；超过应拆分子组件或抽 composable。
- 单个函数建议 ≤ 50 行；复杂逻辑抽成具名函数或 composable。
- 避免重复逻辑：出现第二次即考虑抽取。

## 异步与副作用

- 用 `async/await`，统一 `try/catch` 或交由统一错误处理；不在多处重复 loading 逻辑。
- 组件挂载时的请求放在 composable 或统一的数据获取入口，避免散落在生命周期钩子里。
- 定时器、监听器、订阅在 `onUnmounted` 中清理，防止内存泄漏。

## 格式与工具

- 格式统一交给 Prettier；提交前跑 `pnpm lint`。
- import 顺序：内置/三方库 → `@/` 别名路径 → 相对路径。
- 禁止在源码中遗留 `console.log` 调试语句（必要的日志走统一 logger 或按环境判断）。

## 本项目硬约定

- HTTP 一律走 `api/` 层的统一 axios 封装，不直接用 `fetch`/裸 `axios`。
- UI 一律用 Vant 4，不自造与其重复的基础组件。
- 状态用 Pinia，不引入 Vuex 或全局事件总线作为并行方案。
- 路径别名统一用 `@/`（指向 `src/`）。
