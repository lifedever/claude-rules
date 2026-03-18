# Vue 3 Guidelines

## Basic Component Rules

- Use only `<script setup lang="ts">`, Options API is forbidden
- A single component file must not exceed 200 lines; split if it does
- Components are responsible only for UI rendering and user interaction; extract business logic into composables
- Complex expressions in templates are forbidden; extract anything longer than one line into a computed property or method

```vue
<!-- Forbidden: logic in template -->
<span>{{ items.filter(i => i.active).map(i => i.name).join(', ') }}</span>

<!-- Correct -->
<span>{{ activeNames }}</span>

<script setup lang="ts">
const activeNames = computed(() =>
  items.value.filter(i => i.active).map(i => i.name).join(', ')
)
</script>
```

## Reactivity

- Use `ref` for primitive types, `reactive` for objects/arrays
- Do not use `reactive` for primitive types (reactivity will be lost)
- Do not destructure `reactive` objects (use `toRefs` or access properties directly with `.`)
- `computed` must be a pure function; modifying state or making requests inside computed is forbidden

```typescript
// Forbidden
const state = reactive(0)  // Cannot use reactive for primitive types
const { name } = reactive({ name: 'foo' })  // Destructuring loses reactivity

// Correct
const count = ref(0)
const user = reactive({ name: 'foo', age: 25 })
```

## Composables

- File names must have the `use` prefix: `useAuth.ts`, `usePagination.ts`
- A composable should do one thing only
- Return values as object destructuring, not arrays (unless there are only two values like `[state, setState]`)
- Composables are responsible for their own lifecycle cleanup (clear timers, listeners, etc. in `onUnmounted`)

```typescript
// Correct composable structure
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

  // Self-cleanup
  const debouncedSearch = useDebounceFn(search, 300)
  watch(query, debouncedSearch)

  return { query, results, isLoading, search }
}
```

## State Management

- Use Pinia; Vuex is forbidden
- Split stores by feature domain: `useUserStore`, `useCartStore`
- Only put cross-component shared state in stores; use `ref`/`reactive` for component-local state
- Handle async operations in actions; use getters for derived computations
- Do not modify store state directly in components; modify through actions

## Props and Emits

- Use the generic syntax of `defineProps` for type definitions, not runtime declarations
- Emit types must be defined
- Prefer primitive types over objects for props to reduce coupling

```typescript
// Forbidden: runtime declaration
const props = defineProps({
  title: { type: String, required: true }
})

// Correct: type declaration
const props = defineProps<{
  title: string
  count?: number
}>()

const emit = defineEmits<{
  update: [value: string]
  delete: [id: number]
}>()
```

## Routing

- Lazy-load route components with `defineAsyncComponent` or dynamic `import()`
- Async operations in route guards (authentication, etc.) must have error handling and loading state
- Pass route params to components via props; do not read `useRoute().params` directly inside components

## Performance

- Use virtual scrolling for long lists (`vue-virtual-scroller` or similar)
- Use `v-show` instead of `v-if` for display-only components that toggle frequently
- `v-for` must have a unique and stable `key`; using index as key is forbidden (unless the list never changes)
- Load large components on demand with `defineAsyncComponent`
