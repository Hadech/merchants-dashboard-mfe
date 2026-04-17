# Merchants Dashboard — Migración Micro-Frontend

## El Problema Real

El Merchants Dashboard de Wompi es un monolito SPA con **Nuxt 1.0.0** (Vue 2.7), Webpack 3, Node 12, sin TypeScript. Acumula **60+ vulnerabilidades ignoradas** en Snyk. El framework está en EOL, las dependencias no se pueden actualizar, y la arquitectura monolítica impide escalar por equipos.

## La Propuesta: Nuxt 3 + Module Federation (NO Next.js)

### Por qué Nuxt 3 y no Next.js

| Factor | Next.js (React) | Nuxt 3 (Vue 3) | Ganador |
|--------|----------------|-----------------|---------|
| Curva de aprendizaje del equipo | Alta — reescribir TODO de Vue a React | Baja — migrar templates Vue 2→3, misma mentalidad | **Nuxt 3** |
| Velocidad de migración | Reescribir cada componente desde cero | Adaptar componentes existentes (80% reutilizable) | **Nuxt 3** |
| Module Federation | `@module-federation/nextjs-mf` (quirks con App Router, SSR issues) | `@module-federation/vite` (estable, Vite nativo) | **Nuxt 3** |
| Migración de estado | Vuex → Zustand/Redux (reescritura total) | Vuex → Pinia (migración casi 1:1, API similar) | **Nuxt 3** |
| Migración de i18n | vue-i18n → next-intl (API completamente diferente) | vue-i18n 9 → @nuxtjs/i18n (misma API, versión nueva) | **Nuxt 3** |
| Migración de auth | Cognito manual → React context (reescribir) | Cognito manual → composable (adaptar) | **Nuxt 3** |
| Riesgo en 48h | Muy alto — framework nuevo + paradigma nuevo | Medio — mismo ecosistema, versión moderna | **Nuxt 3** |
| Ecosistema MFE | Más maduro pero más complejo | Suficiente para el MVP | Empate |

**Conclusión:** Con 4 personas (1 QA) y 48 horas, cambiar de Vue a React es suicidio técnico. Nuxt 3 te da el mismo resultado (0 CVEs, MFE, TypeScript, stack moderno) en la mitad del tiempo porque reutilizas el conocimiento del equipo y adaptas código existente en vez de reescribirlo.

## Arquitectura

### Monorepo con Turborepo

```
merchants-dashboard-mfe/
├── apps/
│   ├── shell/                # Host — layout, auth, nav, routing global
│   ├── mfe-transactions/     # Remote — transacciones, disputas, payment links
│   ├── mfe-payouts/          # Remote — payouts completo (12 páginas)
│   └── mfe-settings/         # Remote — users, roles, keys, developers
├── packages/
│   ├── ui/                   # Design system: componentes compartidos (Nuxt UI)
│   ├── auth/                 # Auth: Cognito, tokens, session, composables
│   ├── api-client/           # API client: ofetch/axios tipado + interceptors
│   ├── i18n/                 # Traducciones CO/PA/SV
│   └── types/                # TypeScript types compartidos
├── turbo.json
├── package.json
└── tsconfig.base.json
```

### Module Federation con Vite

- **Shell (Host):** Nuxt 3 app que expone shared packages y consume MFEs como remotes
- **Cada MFE (Remote):** Nuxt 3 app independiente, deployable por separado
- **Plugin:** `@module-federation/vite` — integración nativa con Vite (bundler de Nuxt 3)
- **Shared:** Vue 3, Pinia, ofetch, Nuxt UI — singleton para evitar duplicación
- **Comunicación:** Event bus tipado para cross-MFE events (merchant seleccionado, auth state)

### Stack Técnico

| Capa | Legacy | Nuevo | Migración |
|------|--------|-------|-----------|
| Framework | Nuxt 1 (Vue 2) | Nuxt 3 (Vue 3) | Adaptar Composition API |
| Lenguaje | JavaScript | TypeScript strict | Agregar tipos progresivamente |
| Estado | Vuex + vuex-saga | Pinia | Migración casi 1:1 |
| UI | Element UI 2 | Nuxt UI (basado en Radix Vue + Tailwind) | Reemplazar componentes |
| HTTP | axios 0.30 | ofetch (nativo Nuxt 3) o axios 1.x | Adaptar interceptors |
| Auth | Cognito + localStorage | Cognito + composable tipado | Misma lógica, tipada |
| Build | Webpack 3 | Vite + Module Federation | Configuración nueva |
| Tests | Jest 23 (sin tests) | Vitest + Playwright | Desde cero |
| CSS | SASS + Element theme | Tailwind CSS + Nuxt UI | Reescribir estilos |
| i18n | vue-i18n 8 | @nuxtjs/i18n (vue-i18n 9) | Adaptar traducciones |
| Node | 12.22 | 20 LTS | Actualizar |

## Migración de Componentes Clave

### API Layer (`api/api.js` → `packages/api-client/`)

El API client actual crea instancias axios con interceptors para:
1. Refresh token Cognito antes de cada request
2. Bearer token + User-Principal-Id headers
3. 401 → logout automático
4. Base URL dinámica (sandbox/producción)

**Migración:** Composable `useApiClient()` que encapsula la misma lógica con ofetch tipado. Los interceptors se convierten en `onRequest`/`onResponseError` hooks de ofetch. Cada MFE importa el client del package compartido.

### Estado (Vuex → Pinia)

Ejemplo de migración directa:

```typescript
// ANTES: store/modules/transactions.js (Vuex + vuex-saga)
const actions = {
  *getReportTransaction(store) {
    yield put('transaction/GET_REPORT_REQUEST', true);
    const response = yield call(getTransactionReport, filters);
    yield put('transaction/GET_REPORT_REQUEST', false);
    return response;
  }
};

// DESPUÉS: stores/transactions.ts (Pinia)
export const useTransactionsStore = defineStore('transactions', () => {
  const filters = ref<TransactionFilters>({...});
  const loading = ref(false);

  async function getReport() {
    loading.value = true;
    const data = await getTransactionReport(removeEmpty(filters.value));
    loading.value = false;
    return data;
  }

  return { filters, loading, getReport };
});
```

La migración Vuex → Pinia es casi mecánica: `state` → `ref()`, `mutations` desaparecen, `actions` → funciones async, `getters` → `computed()`.

### Auth (Cognito)

La librería `amazon-cognito-identity-js` es framework-agnostic. La lógica de `utils/awsCognito.js` y `store/storage.js` se envuelve en un composable:

```typescript
// packages/auth/composables/useAuth.ts
export function useAuth() {
  const user = ref<CognitoUser | null>(null);
  const isAuthenticated = computed(() => !!user.value);

  async function login(username: string, password: string) { /* misma lógica */ }
  async function refreshSession() { /* misma lógica de awsRefreshSession */ }
  function logout() { /* removeSessionAndLogout */ }

  return { user, isAuthenticated, login, refreshSession, logout };
}
```

## Scope del MVP (48 horas)

### Prioridad 1 — Shell + Auth (Horas 0-8)
- Monorepo Turborepo configurado
- Shell Nuxt 3 con layout (sidebar + header + content)
- Auth completo con Cognito (login, logout, refresh, session)
- Module Federation host configurado
- Selector de merchant (sandbox/producción)
- Shared packages base (ui, auth, api-client, types)

### Prioridad 2 — MFE Transactions (Horas 8-20)
- Lista de transacciones con filtros y paginación
- Detalle de transacción
- Descarga de reportes
- Disputas (lista + detalle)
- Payment links (lista + crear + detalle)

### Prioridad 3 — MFE Payouts (Horas 20-32)
- Dashboard de balances
- Crear pago / dispersión
- Lista de transacciones de payouts
- Aprobaciones
- Favoritos y límites
- Reportes

### Prioridad 4 — MFE Settings (Horas 32-40)
- Gestión de usuarios (CRUD)
- Gestión de roles (CRUD)
- Llaves de API
- Mi cuenta
- Developers / integraciones

### Prioridad 5 — Polish + Demo (Horas 40-48)
- Testing E2E de flujos críticos
- UI polish y responsive
- Preparación de demo
- Documentación mínima

### EXCLUIDO del MVP
- Datáfonos, corresponsales bancarios, beneficiarios finales
- Vinculación / onboarding, Learn Wompi
- Custom reports avanzados, procedures/trámites
- Legacy app wrapper, ChatBot, Tour
- Migración de usuarios legacy (nickname flow)

## Distribución del Equipo

| Persona | Horas 0-8 | Horas 8-20 | Horas 20-32 | Horas 32-40 | Horas 40-48 |
|---------|-----------|------------|-------------|-------------|-------------|
| **Líder** | Monorepo + Shell + MF config | Shell polish + integración MFEs | Integración + routing cross-MFE | MFE Settings (parcial) | Demo + pitch |
| **Dev 1** | Shared packages (auth, api) | MFE Transactions completo | MFE Transactions polish | Ayuda en Settings | Testing + fix bugs |
| **Dev 2** | Shared packages (ui, i18n) | MFE Payouts (páginas core) | MFE Payouts completo | MFE Payouts polish | Testing + fix bugs |
| **QA** | Datos de prueba + env setup | Test Transactions E2E | Test Payouts E2E | Test Settings + cross-MFE | Demo rehearsal + video backup |

## Criterios de Éxito

1. **0 vulnerabilidades** en el stack nuevo (Snyk clean)
2. **3 MFEs** funcionando bajo un Shell con Module Federation
3. **Deploy independiente** demostrable
4. **Flujo E2E:** login → transacciones → crear payout → gestionar usuarios
5. **TypeScript** en todo el código nuevo
6. **Paridad funcional** en módulos migrados

## Riesgos

| Riesgo | Mitigación |
|--------|------------|
| Module Federation + Vite tiene issues | Fallback: MFEs como packages internos del monorepo (sin runtime federation, pero misma arquitectura) |
| No alcanza migrar 3 MFEs | Priorizar Shell + Transactions. Payouts segundo. Settings tercero. |
| Nuxt UI no tiene equivalente de algún componente Element UI | Construir componente básico con Tailwind. No buscar pixel-perfect. |
| Auth Cognito falla en Nuxt 3 | La librería es framework-agnostic. Si falla SSR, usar client-only. |

## Variables de Entorno (MVP)

```env
API_GW_BASE_URL=https://api.co.dev.wompi.dev/v1
API_GW_BASE_URL_SANDBOX=https://api-sandbox.co.dev.wompi.dev/v1
API_PAYOUTS_BASE_URL=...
DASHBOARD_USER_POOL_ID=...
DASHBOARD_CLIENT_ID=...
REGION=CO
I18N_LOCALE=es_CO
I18N_FALLBACK_LOCALE=es_PA
PAYOUTS_AVAILABLE=true
CHECKOUT_URL=https://checkout.co.dev.wompi.dev
```
