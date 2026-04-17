---
inclusion: always
---

# Patrones de Migración — Vuex/Vue 2 → Pinia/Vue 3

## Migración de Estado: Vuex + vuex-saga → Pinia

| Vuex | Pinia | Notas |
|------|-------|-------|
| `state: { count: 0 }` | `const count = ref(0)` | Reactivo por defecto |
| `mutations: { SET(state, val) { state.x = val } }` | `x.value = val` | Mutación directa, sin mutations |
| `actions: { *fetch() { yield call(...) } }` | `async function fetch() { await ... }` | Eliminar vuex-saga |
| `getters: { double: s => s.count * 2 }` | `const double = computed(() => count.value * 2)` | Computed |
| `mapState(['count'])` | `const { count } = storeToRefs(store)` | Destructuring reactivo |

## Migración de Componentes: Options API → Composition API

```vue
<!-- ANTES (Vue 2 Options API) -->
<script>
export default {
  data() { return { loading: false } },
  computed: { isReady() { return !this.loading } },
  methods: { async fetch() { this.loading = true; /* ... */ } },
  mounted() { this.fetch() }
}
</script>

<!-- DESPUÉS (Vue 3 Composition API + script setup) -->
<script setup lang="ts">
const loading = ref(false)
const isReady = computed(() => !loading.value)
async function fetch() { loading.value = true; /* ... */ }
onMounted(fetch)
</script>
```

## Migración de Templates

- `v-model` en componentes custom: Vue 3 usa `modelValue` prop + `update:modelValue` emit
- Filtros (`{{ value | currency }}`): Eliminados en Vue 3. Usar funciones: `{{ formatCurrency(value) }}`
- `$refs`: Usar `const myRef = ref<HTMLElement>()`
- `this.$router.push`: Usar `navigateTo()` de Nuxt 4
- `this.$store`: Usar `const store = useMyStore()`
- `this.$t('key')`: Usar `const { t } = useI18n()`

## Migración de Element UI → Nuxt UI

| Element UI | Nuxt UI | Notas |
|-----------|---------|-------|
| `<el-table>` | `<UTable>` | Props diferentes, adaptar columns |
| `<el-button>` | `<UButton>` | Similar API |
| `<el-input>` | `<UInput>` | Similar API |
| `<el-select>` | `<USelect>` o `<USelectMenu>` | API diferente |
| `<el-dialog>` | `<UModal>` | Usar `v-model` |
| `<el-message>` | `useToast()` | Composable en vez de global |
| `<el-pagination>` | Incluido en `<UTable>` | Paginación integrada |
| `<el-date-picker>` | `<UPopover>` + date lib | No hay date picker nativo |
| `<el-form>` + rules | `<UForm>` + Zod/Yup | Validación con schemas |
