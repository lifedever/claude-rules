# Vue 3 规范

## 组件基本规则

- 只用 `<script setup lang="ts">`，禁止选项式 API
- 单个组件文件不超过 200 行，超过必须拆分
- 组件只负责 UI 渲染和用户交互，业务逻辑提取到 composable
- 模板中禁止复杂表达式，超过一行的逻辑提取为 computed 或方法

```vue
<!-- 禁止：模板里写逻辑 -->
<span>{{ items.filter(i => i.active).map(i => i.name).join(', ') }}</span>

<!-- 正确 -->
<span>{{ activeNames }}</span>

<script setup lang="ts">
const activeNames = computed(() =>
  items.value.filter(i => i.active).map(i => i.name).join(', ')
)
</script>
```

## 响应式

- 基本类型用 `ref`，对象/数组用 `reactive`
- 禁止对基本类型用 `reactive`（会丢失响应性）
- 禁止解构 `reactive` 对象（用 `toRefs` 或直接 `.` 访问）
- `computed` 必须是纯函数，禁止在 computed 里修改状态或发请求

```typescript
// 禁止
const state = reactive(0)  // 基本类型不能用 reactive
const { name } = reactive({ name: 'foo' })  // 解构丢失响应性

// 正确
const count = ref(0)
const user = reactive({ name: 'foo', age: 25 })
```

## Composable

- 文件名 `use` 前缀：`useAuth.ts`、`usePagination.ts`
- 一个 composable 只做一件事
- 返回值用对象解构，不要返回数组（除非只有两个值如 `[state, setState]`）
- composable 内部负责自己的生命周期清理（`onUnmounted` 里清除定时器、监听器等）

```typescript
// 正确的 composable 结构
export function useSearch(initialQuery = '') {
  const query = ref(initialQuery)
  const results = ref<SearchResult[]>([])
  const isLoading = ref(false)

  const search = async () => {
    isLoading.value = true
    try {
      results.value = await searchApi(query.value)
    } finally {
      isLoading.value = false
    }
  }

  // 自己清理
  const debouncedSearch = useDebounceFn(search, 300)
  watch(query, debouncedSearch)

  return { query, results, isLoading, search }
}
```

## 状态管理

- 用 Pinia，禁止 Vuex
- Store 按功能域拆分：`useUserStore`、`useCartStore`
- Store 里只放跨组件共享的状态，组件内部状态用 `ref`/`reactive`
- Action 里处理异步操作，getter 做派生计算
- 禁止在组件里直接修改 store 的 state，通过 action 修改

## Props 和 Emits

- 用 `defineProps` 的泛型语法做类型定义，不用运行时声明
- 必须定义 emit 类型
- Props 能用基本类型就不用对象，降低耦合

```typescript
// 禁止：运行时声明
const props = defineProps({
  title: { type: String, required: true }
})

// 正确：类型声明
const props = defineProps<{
  title: string
  count?: number
}>()

const emit = defineEmits<{
  update: [value: string]
  delete: [id: number]
}>()
```

## 路由

- 路由组件用 `defineAsyncComponent` 或动态 `import()` 懒加载
- 路由守卫里的异步操作（鉴权等）必须有错误处理和 loading 状态
- 路由参数通过 props 传入组件，不在组件里直接读 `useRoute().params`

## 性能

- 长列表用虚拟滚动（`vue-virtual-scroller` 或类似方案）
- 纯展示组件不需要 `v-if` 频繁切换的用 `v-show`
- `v-for` 必须有唯一且稳定的 `key`，禁止用 index 做 key（除非列表不会变动）
- 大型组件用 `defineAsyncComponent` 按需加载
