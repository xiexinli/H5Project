---
name: web-native-dev
description: H5Project 的 Web（移动端 H5）开发总纲。覆盖 Vue 3 + Vant 4 + Vite + TypeScript 技术栈、分层与目录约定、组件与组合式写法、移动端适配、构建与验证流程、常见构建/运行报错速查。在本仓库做 H5 开发或排障前使用。
---

# Web Native Dev

本仓库的 H5 开发总纲。

## 1. 仓库优先

应用下面的通用建议之前，先读 `references/project-facts.md`。

**当通用 H5 建议与本仓库事实冲突时，以仓库事实为准。**

本仓库是移动端 H5 工程：

- 技术栈为 **Vue 3 + TypeScript + Vant 4 + Vite**，包管理用 pnpm。
- 使用 `<script setup lang="ts">` 与 Composition API，**不写 Options API**。
- 构建与脚本命令以 `package.json` 为准（`dev` / `build` / `preview` / `lint` / `type-check` / `test`）。
- 分层与依赖方向以 `.dhcoder/rules/architecture.md` 为准（`views` → `components` → `composables/stores` → `api/services` → `utils/types`）。
- 编码风格以 `.dhcoder/rules/code-style.md` 为准。
- 依赖版本以 `package.json` + lockfile 为准，不在多处散写版本。

### 1.1 默认验证路径

用能证明改动的最小命令（具体见 `web-verify` 与 `references/task-map.md`）：

- 类型/编译证明：`pnpm type-check`（若已配置 `tsc --noEmit`）
- 静态检查：`pnpm lint`
- 构建证明：`pnpm build`
- 单元测试：`pnpm test`（Vitest，若已配置）

### 1.2 项目级坑

- 项目若尚未初始化（无 `package.json`/`src`），先确认脚手架与依赖已就绪再谈实现。
- Vite 别名 `@/` 指向 `src/`；新增深层相对路径穿透前优先用别名。
- Vant 组件按需自动引入（`unplugin-vue-components`）；不要在业务里重复全局注册。
- 环境变量只暴露 `VITE_` 前缀；接口地址按环境走 `.env.*`，不硬编码在源码。
- 移动端调试需真机或浏览器设备模拟；WebView 差异以真机为准。

## 2. 项目快照（默认心智模型）

| 领域 | 仓库事实 |
|---|---|
| 构建工具 | Vite（配置文件 `vite.config.ts`） |
| 框架 | Vue 3（Composition API + `<script setup>`） |
| 语言 | TypeScript（`strict`） |
| UI 组件库 | Vant 4（按需引入） |
| 路由 | Vue Router 4 |
| 状态 | Pinia |
| HTTP | axios（`api/` 层统一封装） |
| 包管理 | pnpm（以 lockfile 为准） |
| 入口 | `src/main.ts`、`index.html` |
| 别名 | `@/` → `src/` |

> 上表是目标约定；以工程实际文件为准，落地后回写 `references/project-facts.md`。

## 3. TypeScript 与 Vue 规范

### 3.1 命名约定

| 类型 | 约定 | 示例 |
|---|---|---|
| 组件 | PascalCase | `UserCard.vue` |
| composable | `useXxx` | `useUserProfile` |
| store | `useXxxStore` | `useCartStore` |
| 变量/函数 | camelCase | `fetchList` |
| 常量 | SCREAMING_SNAKE | `MAX_RETRY` |
| 目录 | kebab-case | `user-profile/` |

### 3.2 类型与空安全

```ts
// ❌ 避免：any 削弱类型检查
function handle(data: any) { return data.name }

// ✅ 推荐：unknown + 收窄；接口字段按可能缺失处理
function handle(data: unknown) {
  if (isUser(data)) return data.name
  return ''
}
```

```ts
// ❌ 避免：直接假设响应字段存在
const name = res.data.user.name

// ✅ 推荐：可选链 + 兜底
const name = res.data?.user?.name ?? ''
```

### 3.3 组件写法

```vue
<script setup lang="ts">
interface Props {
  title: string
  count?: number
}
const props = withDefaults(defineProps<Props>(), { count: 0 })
const emit = defineEmits<{ (e: 'change', value: number): void }>()
</script>

<template>
  <van-button type="primary" @click="emit('change', props.count + 1)">
    {{ props.title }}
  </van-button>
</template>
```

### 3.4 组合式函数与副作用

```ts
// composables/use-user-profile.ts
export function useUserProfile(id: MaybeRef<string>) {
  const data = ref<User | null>(null)
  const loading = ref(false)

  async function load() {
    loading.value = true
    try {
      data.value = await fetchUser(toValue(id))
    } finally {
      loading.value = false
    }
  }

  onMounted(load)
  return { data, loading, reload: load }
}
```

- 计时器、事件监听、订阅必须在 `onUnmounted` 清理。
- 组件挂载请求放 composable，不散落在生命周期里。

## 4. 请求与数据

- 只在 `api/` / `services/` 层发起请求，统一用封装的 axios 实例。
- 统一处理：baseURL、超时、token 注入、错误归一化、loading。
- 页面层只消费数据与处理提示，不拼请求、不吞异常。

```ts
// api/request.ts（示意）
const request = axios.create({ baseURL: import.meta.env.VITE_API_BASE_URL, timeout: 15000 })
request.interceptors.request.use((config) => { /* 注入 token */ return config })
request.interceptors.response.use(
  (res) => res.data,
  (err) => Promise.reject(normalizeError(err)),
)
export default request
```

## 5. 移动端适配与 UI

- 尺寸走移动端适配方案（vw/rem），避免散落裸 px。
- 安全区：`padding-bottom: env(safe-area-inset-bottom)` 处理底部安全区。
- 触控目标建议 ≥ 44×44px，间距足够，避免误触。
- 文字对比度正文 ≥ 4.5:1，大字 ≥ 3:1。
- Vant 主题用 CSS 变量定制，不覆盖组件内部类名。
- 复杂列表用虚拟滚动（`van-list` + 分页）避免长列表卡顿。

## 6. 常见构建/运行报错速查

| 报错关键词 | 原因 | 处理 |
|---|---|---|
| `Cannot find module '@/...'` | 别名未配置/未生效 | 检查 `vite.config.ts` 与 `tsconfig` 的 paths |
| `Failed to resolve import` | 依赖未安装或路径错误 | `pnpm install`；核对路径 |
| `Type error ... is not assignable` | 类型不匹配 | 修正类型，勿用 `any` 掩盖 |
| `Component name "xxx" should be multi-word` | 组件命名不规范 | 用多词 PascalCase |
| `[vite] Internal server error` | 配置/语法错误 | 看完整堆栈，常见于 SSR/插件配置 |
| White screen / 白屏 | 运行时错误或资源路径问题 | 看浏览器控制台；检查 `base` 配置 |

## 7. 测试

> 用户明确要求测试时再补测试依赖与用例。

- 单元/组件测试用 **Vitest**（+ `@vue/test-utils`）；放在 `src/**/__tests__` 或 `*.spec.ts`。
- 优先测纯逻辑（`utils`、composable、store）与组件行为，不测实现细节。
- E2E（如 Playwright）按需接入，不作为默认。
- 与 `tdd` skill 配合（先写测试）。

## 8. References

| 主题 | 文件 |
|---|---|
| 本仓库事实（build 形态、目录、约定、验证命令、坑） | [project-facts.md](references/project-facts.md) |
