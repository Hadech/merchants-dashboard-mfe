# Skill: Integrar API de Wompi

## Descripción
Crea composables tipados para integrar endpoints del API de Wompi en los MFEs.

## Cuándo usarlo
- "Integra el endpoint de transacciones"
- "Crea el composable de API para payouts"
- "Conecta el MFE con el backend de Wompi"

## Instrucciones

### Contexto del API de Wompi

Base URLs (configuradas por env vars):
- Producción: `VITE_API_GW_BASE_URL` (ej: `https://api.co.dev.wompi.dev/v1`)
- Sandbox: `VITE_API_GW_BASE_URL_SANDBOX`
- Payouts: `VITE_API_PAYOUTS_BASE_URL`

Prefijo: Todos los endpoints del dashboard usan `/dashboard` como prefijo.

Headers requeridos:
- `Authorization: Bearer {accessToken}` — token de Cognito
- `User-Principal-Id: {merchantUserId}` — ID del merchant activo

Estos headers los inyecta automáticamente `@wompi/api-client`.

### Proceso

1. **Identificar los endpoints** que necesita el MFE (revisar `merchants-dashboard/api/{dominio}.js`)

2. **Crear composable de API**:
```typescript
// apps/mfe-{domain}/app/composables/use{Domain}Api.ts
import { useApiClient } from '@wompi/api-client'
import type { /* tipos */ } from '@wompi/types'

export function use{Domain}Api() {
  const api = useApiClient()

  async function getList(filters?: Partial<Filters>) {
    return api<{ data: Item[] }>('/endpoint', { query: filters })
  }

  async function getById(id: string) {
    return api<{ data: Item }>(`/endpoint/${id}`)
  }

  async function create(payload: CreateRequest) {
    return api<{ data: Item }>('/endpoint', {
      method: 'POST',
      body: payload,
    })
  }

  async function update(id: string, payload: Partial<Item>) {
    return api<{ data: Item }>(`/endpoint/${id}`, {
      method: 'PATCH',
      body: payload,
    })
  }

  return { getList, getById, create, update }
}
```

3. **Tipar las respuestas** en `@wompi/types` si no existen

4. **Usar en el store o componente**:
```typescript
const { getList } = use{Domain}Api()
const data = await getList({ status: 'APPROVED' })
```

### Reglas
- SIEMPRE usar `useApiClient()`, NUNCA fetch/axios directo
- SIEMPRE tipar request y response
- Los interceptors de auth se aplican automáticamente
- Para endpoints de payouts, usar `useApiClient({ baseUrl: import.meta.env.VITE_API_PAYOUTS_BASE_URL })`
- Manejar errores en el store, no en el composable de API
