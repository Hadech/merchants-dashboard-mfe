# Skill: Migrar Store Vuex a Pinia

## Descripción
Migra un módulo Vuex + vuex-saga del legacy a un Pinia store tipado con Composition API.

## Cuándo usarlo
- "Migra el store de transactions"
- "Convierte este módulo Vuex a Pinia"
- "Crea el store de payouts basado en el legacy"

## Instrucciones

### Entrada
El usuario proporciona un archivo de `merchants-dashboard/store/modules/{nombre}.js` del legacy.

### Proceso

1. **Leer el módulo Vuex** completo (state, mutations, actions, getters)
2. **Identificar dependencias**: qué APIs llama, qué tipos necesita
3. **Crear el Pinia store** siguiendo este patrón:

```typescript
// apps/mfe-{domain}/app/stores/{nombre}.ts
import { defineStore } from 'pinia'
import type { /* tipos */ } from '@wompi/types'
import { useApiClient } from '@wompi/api-client'

export const use{Nombre}Store = defineStore('{nombre}', () => {
  const api = useApiClient()

  // state → ref()
  const loading = ref(false)
  const error = ref<string | null>(null)
  const data = ref<TipoData[]>([])

  // mutations → eliminadas, mutar directamente

  // getters → computed()

  // actions (generators → async functions)
  async function fetchData() {
    loading.value = true
    error.value = null
    try {
      const response = await api('/endpoint')
      data.value = response.data || []
    } catch (e: unknown) {
      error.value = e instanceof Error ? e.message : 'Error desconocido'
    } finally {
      loading.value = false
    }
  }

  return { loading, error, data, fetchData }
})
```

4. **Reglas de conversión**:
   - `state: { x: val }` → `const x = ref(val)`
   - `mutations` → eliminar completamente
   - `yield put('mutation', val)` → `x.value = val`
   - `yield call(apiFunction, args)` → `await apiFunction(args)`
   - `getters: { x: state => ... }` → `const x = computed(() => ...)`
   - Callbacks (successCallback/errorCallback) → try/catch con return

5. **Crear composable de API** si no existe:

```typescript
// apps/mfe-{domain}/app/composables/use{Domain}Api.ts
import { useApiClient } from '@wompi/api-client'
import type { /* tipos */ } from '@wompi/types'

export function use{Domain}Api() {
  const api = useApiClient()
  // funciones tipadas por endpoint
  return { /* funciones */ }
}
```

6. **Verificar** que los tipos existen en `@wompi/types`, crearlos si no

### Salida
- Archivo de store en `apps/mfe-{domain}/app/stores/{nombre}.ts`
- Composable de API si es necesario
- Tipos nuevos en `packages/types/src/` si es necesario
