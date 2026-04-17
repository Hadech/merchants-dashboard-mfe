---
inclusion: always
---
# Reto 2 — Merchants Dashboard · Hackathon Wompi 2026

## Objetivo
Cerrar las 85 vulnerabilidades de seguridad del dashboard de comercios, modernizar el stack tecnológico (Nuxt 1/2 → Nuxt 4) y sentar las bases de una arquitectura micro-frontend escalable. Bono: agente IA que genera System Design en Figma.

## Contexto de Evaluación (Hackathon)
- 48 horas · 4 ingenieros
- 33 preguntas de evaluación en 4 dominios (Pitch, Funcional, Calidad, Técnicas IA)
- Cada decisión técnica debe ser justificable ante el jurado

## Criterios de Evaluación con Pesos

| Criterio | Peso | Evidencia requerida |
|---|---|---|
| Vulnerabilidades cerradas (todas las críticas y altas) | 30% | Pentest report + code review |
| Arquitectura Micro-Frontend (Module Federation con ≥2 MFEs) | 25% | Demo de deploy independiente |
| Modernización del stack (TS + Nuxt 4 + deps actualizadas) | 20% | Bundle + TS strict |
| Calidad & Tests (cobertura >60% en lógica crítica, E2E) | 15% | Coverage + CI verde |
| Observabilidad (Sentry + contexto de comercio) | 10% | Demo de error tracking |
| **Bono — Figma AI Agent** | **+10%** | Demo del agente generando componentes en Figma en vivo |

## Dominios de Evaluación WAI 2026

### Pitch (Preguntas 2-6 · 5 items · Escala 1-5)
- Historia coherente en tiempo
- Happy path E2E + caso borde
- Entregado por más de una persona
- Frase memorable para el jurado
- Cierre con visión tangible

### Funcional (Preguntas 7-12 · 6 items · Escala 1-5)
- Demo end-to-end sin fallos
- Cubre expectativas funcionales
- Operable sin guía del equipo
- Casos borde cubiertos
- Manejo de errores sin colapso
- Production-readiness

### Calidad de la Solución (Preguntas 13-16 · 4 items · Escala + Checkboxes)
- Cobertura pruebas unitarias ≥85%
- Tipos: unitarias / integración / E2E
- Documentación de la solución
- NFRs definidos

### Técnicas IA (Preguntas 17-33 · 17 items · Escala + Texto libre)
- Spec formal como contrato
- Subagents · Skills · Multi-agent
- Tests automáticos post-cambio
- Fallos de IA documentados + ADRs
- Tradeoffs · Sobre-ingeniería evitada
- Fricciones · Alucinaciones · Observabilidad
- Humano-centric vs Agent-centric
- Coherencia del código con N agentes

## Repositorio Legacy — Referencia para Migración

El código fuente original está en `merchants-dashboard/` (multi-root workspace). El agente DEBE consultar estos archivos cuando necesite entender lógica existente, endpoints, traducciones o comportamiento a replicar.

### Mapa de Referencia: Legacy → Nuevo

| Necesitas... | Busca en el legacy | Migra a |
|---|---|---|
| **API endpoints y llamadas HTTP** | `merchants-dashboard/api/*.js` | `merchants-dashboard-mfe/packages/api-client/` |
| **Interceptors (auth headers, refresh, 401)** | `merchants-dashboard/api/interceptors.js` + `merchants-dashboard/api/api.js` | `merchants-dashboard-mfe/packages/api-client/client.ts` |
| **Auth Cognito (login, refresh, logout)** | `merchants-dashboard/utils/awsCognito.js` + `merchants-dashboard/store/storage.js` | `merchants-dashboard-mfe/packages/auth/` |
| **Vuex stores (state, mutations, actions, sagas)** | `merchants-dashboard/store/modules/*.js` + `merchants-dashboard/store/state.js` | `merchants-dashboard-mfe/apps/mfe-*/stores/*.ts` (Pinia) |
| **Traducciones i18n** | `merchants-dashboard/locales/es_CO.json`, `es_PA.json`, `es_GT.json` | `merchants-dashboard-mfe/packages/i18n/locales/` |
| **Layout principal (sidebar, header)** | `merchants-dashboard/layouts/layoutSidebar.vue` | `merchants-dashboard-mfe/apps/shell/app/layouts/default.vue` |
| **Middleware de auth** | `merchants-dashboard/middleware/auth.js` | `merchants-dashboard-mfe/apps/shell/app/middleware/auth.global.ts` |
| **Páginas/rutas** | `merchants-dashboard/pages/*/` | `merchants-dashboard-mfe/apps/mfe-*/app/pages/` |
| **Componentes de UI** | `merchants-dashboard/components/*/` | `merchants-dashboard-mfe/apps/mfe-*/app/components/` o `packages/ui/` |
| **Mixins (lógica reutilizable)** | `merchants-dashboard/mixins/*.js` | Composables en `merchants-dashboard-mfe/apps/mfe-*/app/composables/` |
| **Plugins (i18n, element-ui, etc.)** | `merchants-dashboard/plugins/*.js` | Nuxt modules/plugins en cada app |
| **Utils (formatters, validators, constants)** | `merchants-dashboard/utils/*.js` | Utils tipados en el package correspondiente |
| **Variables de entorno** | `merchants-dashboard/.env.example` | `merchants-dashboard-mfe/apps/*/.env` |
| **Config Nuxt legacy** | `merchants-dashboard/nuxt.config.js` | `merchants-dashboard-mfe/apps/*/nuxt.config.ts` |

### Archivos Clave del Legacy (consultar primero)

| Archivo | Qué contiene | Prioridad |
|---|---|---|
| `api/api.js` | Factory de instancias axios, base URLs, prefijos | Alta |
| `api/interceptors.js` | Refresh token, Bearer header, User-Principal-Id, 401 handling | Alta |
| `utils/awsCognito.js` | Login, refresh session, logout, Cognito config | Alta |
| `store/storage.js` | Gestión de tokens en localStorage/sessionStorage | Alta |
| `store/state.js` | Estado global (merchant, environment, user) | Alta |
| `store/modules/transactions.js` | Store de transacciones (patrón de referencia para migración) | Alta |
| `layouts/layoutSidebar.vue` | Layout principal con sidebar + header + content | Alta |
| `middleware/auth.js` | Lógica de protección de rutas | Media |
| `api/transactions.js` | Endpoints de transacciones | Media |
| `api/payouts.js` | Endpoints de payouts | Media |
| `api/users.js` + `api/roles.js` | Endpoints de settings | Media |
| `plugins/i18n.js` | Config de vue-i18n | Media |
| `locales/es_CO.json` | Traducciones principales | Media |

### Módulos Vuex del Legacy → Pinia Stores

| Vuex Module (legacy) | Pinia Store (nuevo) | MFE destino |
|---|---|---|
| `store/modules/transactions.js` | `transactions.ts` | mfe-transactions |
| `store/modules/reports.js` | (integrado en transactions) | mfe-transactions |
| `store/modules/merchants.js` | `useMerchantContext.ts` | shell |
| `store/modules/users.js` | `users.ts` | mfe-settings |
| `store/modules/userRoles.js` | `roles.ts` | mfe-settings |
| `store/modules/userPermissions.js` | (integrado en roles) | mfe-settings |
| `store/modules/products.js` | (evaluar si aplica al MVP) | — |
| `store/modules/sandBoxMerchants.js` | `useMerchantContext.ts` | shell |
| `store/modules/vouchers.js` | (evaluar si aplica al MVP) | — |
| `store/modules/dataphones.js` | EXCLUIDO del MVP | — |
| `store/modules/truora.js` | EXCLUIDO del MVP | — |
| `store/modules/transferFiles.js` | EXCLUIDO del MVP | — |
| `store/modules/userMigrations.js` | EXCLUIDO del MVP | — |
| `store/modules/userNickname.js` | EXCLUIDO del MVP | — |
| `store/modules/dependentUser.js` | EXCLUIDO del MVP | — |

### API Modules del Legacy → Composables

| API Module (legacy) | Composable (nuevo) | MFE destino |
|---|---|---|
| `api/transactions.js` | `useTransactionsApi.ts` | mfe-transactions |
| `api/payouts.js` | `usePayoutsApi.ts` | mfe-payouts |
| `api/users.js` | `useUsersApi.ts` | mfe-settings |
| `api/roles.js` | `useRolesApi.ts` | mfe-settings |
| `api/merchants.js` | `useMerchantContext.ts` | shell |
| `api/reports.js` | `useReportsApi.ts` | mfe-transactions |
| `api/dataphones.js` | EXCLUIDO del MVP | — |
| `api/vouchers.js` | EXCLUIDO del MVP | — |
| `api/truora.js` | EXCLUIDO del MVP | — |
| `api/transfer-files.js` | EXCLUIDO del MVP | — |
| `api/user-migrations.js` | EXCLUIDO del MVP | — |
| `api/user-nickname.js` | EXCLUIDO del MVP | — |
| `api/check-if-nickname-exists.js` | EXCLUIDO del MVP | — |
| `api/dependent-user.js` | EXCLUIDO del MVP | — |
| `api/otp-merchants.js` | EXCLUIDO del MVP | — |
| `api/sandbox-validate.js` | (integrado en shell) | shell |
| `api/products.js` | (evaluar si aplica) | — |

## Auditoría de Vulnerabilidades — 13 paquetes · 85 issues

### Críticas (prioridad máxima)
| Paquete | Versión actual | Target | Issues | Detalle |
|---|---|---|---|---|
| nuxt | 2.x | 4.x | 66 Critical + 1 High | Core del framework. Migración completa a Nuxt 4 (Vue 3, Vite 6) |
| axios | 0.x | Eliminado → ofetch | 1 Critical + 2 High | Reemplazado por ofetch nativo de Nuxt 4. 0 CVEs |

### Altas
| Paquete | Versión actual | Target | Issues | Detalle |
|---|---|---|---|---|
| vue-i18n | legacy | 11.2.8 | 1 Major | Bloquea la localización segura |
| @nuxtjs/svg | 0.4.0 | Reemplazar por vite-svg-loader | 8 transitivas | Sin mantenimiento activo |

### Medias
| Paquete | Versión actual | Target | Issues | Detalle |
|---|---|---|---|---|
| highlight.js | 9.18.5 | 11.x | 1 directo | ReDoS vulnerability. Versión 9.x sin soporte |
| vue-svg-loader | legacy | Eliminar | 1 transitiva | Reemplazado por vite-svg-loader |
| nuxt-svg-loader | legacy | Eliminar | 1 transitiva | Reemplazado por vite-svg-loader |
| vuex-persistedstate | 4.1.0 | Pinia persisted | 1 transitiva | Migrar a Pinia |
| element-ui | 2.15.8 | Nuxt UI / PrimeVue | 1 transitiva | EOL, sin soporte Vue 3 |
| express | 4.22.1 | 5.x o Nitro | 1 transitiva | Reemplazado por Nitro en Nuxt 3 |

## Stack: Before → After (DECISIONES FINALES)

| Categoría | Legacy | Decisión Final |
|---|---|---|
| Framework | Nuxt 1.0.0 (Vue 2.7, Webpack 3) | **Nuxt 4** (Vue 3.5, Vite 6) — estructura `app/` |
| Lenguaje | JavaScript ES5 | **TypeScript 5 strict** |
| Monorepo | Monolito | **Turborepo + pnpm workspaces** |
| MFE Runtime | N/A | **`@module-federation/vite`** |
| HTTP | axios 0.x (3 CVEs) | **`ofetch`** (nativo Nuxt 4, 0 CVEs) |
| Estado | Vuex + vuex-saga | **Pinia 3** (Composition API) |
| UI | element-ui@2.15.8 | **Nuxt UI v3** (Radix Vue + Tailwind CSS 4) |
| Auth | amazon-cognito-identity-js + localStorage | **Misma lib** envuelta en composable tipado |
| Comunicación MFE | N/A | **Event Bus tipado (mitt)** |
| i18n | vue-i18n 8 | **vue-i18n@11** via `@nuxtjs/i18n` |
| SVG | 3 loaders vulnerables | vite-svg-loader |
| Tests | Jest 23 (sin tests) | **Vitest + fast-check + Playwright** |
| Observabilidad | console.log() | Sentry + OTel |
| Node | 12.22 | **20 LTS** |
| CSS | SASS + Element theme | **Tailwind CSS 4** (no requiere sass) |

## Arquitectura Micro-Frontend (DEFINIDA)

### Estructura del Monorepo

```
merchants-dashboard-mfe/
├── apps/
│   ├── shell/                # Host — layout, auth, nav, routing global (Nuxt 4)
│   ├── mfe-transactions/     # Remote — transacciones, disputas, payment links
│   ├── mfe-payouts/          # Remote — balances, pagos, aprobaciones, favoritos
│   └── mfe-settings/         # Remote — users, roles, keys, developers
├── packages/
│   ├── auth/                 # @wompi/auth — Cognito composable
│   ├── api-client/           # @wompi/api-client — ofetch + interceptors
│   ├── ui/                   # @wompi/ui — Nuxt UI + Tailwind components
│   ├── i18n/                 # @wompi/i18n — Traducciones CO/PA/GT
│   ├── types/                # @wompi/types — TypeScript interfaces compartidas
│   └── event-bus/            # @wompi/event-bus — mitt tipado
├── turbo.json
├── package.json              # packageManager: pnpm@9.15.0
├── pnpm-workspace.yaml
└── tsconfig.base.json
```

### Module Federation — Shared Singletons
Estas dependencias se comparten como singleton entre Shell y todos los MFEs:
- `vue` (^3.5.0)
- `pinia` (^3.0.0)
- `vue-i18n` (^11.0.0)
- `ofetch`

### Routing MFE — Mapa de Slugs a Remotes

| Slug URL | Remote | Módulo Expuesto |
|---|---|---|
| `/transactions`, `/disputes`, `/payment-links` | `mfe-transactions` | `TransactionsApp` |
| `/payouts` (balances, create-payment, approvals, etc.) | `mfe-payouts` | `PayoutsApp` |
| `/users`, `/roles`, `/keys`, `/my-account`, `/developers` | `mfe-settings` | `SettingsApp` |

El Shell usa un catch-all `[...slug].vue` que resuelve el primer segmento contra este mapa.

### Comunicación Cross-MFE (Event Bus)
Eventos tipados via `@wompi/event-bus`:
- `merchant:changed` → Todos los MFEs recargan datos
- `environment:changed` → Cambio sandbox/producción
- `auth:logout` → Limpieza global
- `auth:session-refreshed` → Token actualizado

Cada MFE se registra con un `mfeId` y ejecuta `cleanup()` en `onUnmounted`.

## Patrones de Migración

### Vuex + vuex-saga → Pinia (Composition API)

| Vuex | Pinia | Notas |
|---|---|---|
| `state: { count: 0 }` | `const count = ref(0)` | Reactivo por defecto |
| `mutations: { SET(state, val) }` | `count.value = val` | Mutación directa, sin mutations |
| `actions: { *fetch() { yield call(...) } }` | `async function fetch() { await ... }` | Eliminar vuex-saga, usar async/await |
| `getters: { double: s => s.count * 2 }` | `const double = computed(() => count.value * 2)` | Computed estándar |
| `mapState(['count'])` | `const { count } = storeToRefs(store)` | Destructuring reactivo |
| `mapSagas({ fetch: 'fetchData' })` | `const { fetchData } = store` | Llamada directa |

### Patrón estándar de Store (Pinia)
Todos los stores siguen: `loading` ref + `error` ref + async action con try/catch/finally.

### API Client — Interceptors
`@wompi/api-client` usa `ofetch.create()` con:
- `onRequest`: refresh token Cognito (2 intentos) + headers `Authorization` y `User-Principal-Id`
- `onResponseError`: HTTP 401 → `logout()` automático
- Base URL dinámica desde `localStorage('apiEnvironment')`
- Prefijo `/dashboard` automático

## Bono — Figma AI Agent (+10%)
Agente sobre Claude API que:
- Parsea requerimientos en lenguaje natural o JSON
- Genera árbol de componentes con jerarquía
- Propone design tokens desde el brand
- Crea frames en Figma con layout auto (vía Figma REST API)
- Añade anotaciones de UX y handoff
- Itera sobre feedback

Stack del agente: Claude API · Figma REST · Tool Use · Agentic Loop

## Scope del MVP (48 horas)

### Incluido
- Shell + Auth completo (Cognito login/logout/refresh, selector merchant/sandbox)
- MFE Transactions (lista, detalle, filtros, disputas, payment links)
- MFE Payouts (balances, crear pago, aprobaciones, favoritos, límites, reportes)
- MFE Settings (users CRUD, roles CRUD, keys, mi cuenta, developers)
- Shared packages completos (@wompi/auth, api-client, ui, i18n, types, event-bus)
- Module Federation host+remotes configurado
- TypeScript strict en todo código nuevo

### EXCLUIDO del MVP (no implementar)
- Datáfonos, corresponsales bancarios, beneficiarios finales
- Vinculación / onboarding, Learn Wompi
- Custom reports avanzados, procedures/trámites
- Legacy app wrapper, ChatBot, Tour
- Migración de usuarios legacy (nickname flow)
- CI/CD elaborado
- SSR (todo es SPA mode, `ssr: false`)

## Problemas del Dashboard Actual

### Vulnerabilidades de Seguridad
- nuxt@2.x con 66 issues críticas conocidas (CVEs)
- axios desactualizado con 3 vulnerabilidades (1 Critical + 2 High)
- highlight.js@9.18.5 con issue directo de seguridad
- express@4.22.1 con issue transitivo sin parchear
- 13 dependencias con issues transitivos
- @nuxtjs/svg, vue-svg-loader, nuxt-svg-loader — 10 issues combinados
- element-ui@2.15.8 y vuex-persistedstate@4.1.0 con issues

### Deuda Técnica del Stack
- Nuxt 1/2 en EOL — requiere migración a Nuxt 4
- Arquitectura monolítica imposible de escalar por equipo
- vue-i18n desactualizado con 1 issue Major
- Sin TypeScript — errores en runtime evitables
- Cobertura de tests < 10%
- Sin observabilidad ni error tracking en producción

## Agentes Especializados — Uso OBLIGATORIO

El proyecto tiene 4 agentes custom en `.kiro/agents/`. El agente orquestador DEBE usar `invokeSubAgent` para delegarles trabajo cuando la tarea coincida con su especialidad. NO hacer el trabajo manualmente si hay un agente para ello.

### Agentes Disponibles

| Agente | ID para invokeSubAgent | Cuándo usarlo |
|---|---|---|
| **MFE Scaffolder** | `mfe-scaffolder` | Crear la estructura de cualquier MFE nuevo (nuxt.config, app.vue, package.json, pages, stores) |
| **Vuex→Pinia Migrator** | `vuex-to-pinia-migrator` | Migrar cualquier store de `merchants-dashboard/store/modules/*.js` a Pinia |
| **API Module Migrator** | `api-module-migrator` | Migrar cualquier módulo de `merchants-dashboard/api/*.js` a composable ofetch |
| **Legacy Code Analyzer** | `legacy-code-analyzer` | Analizar código legacy antes de migrar (produce reporte de complejidad) |

### Mapeo Tasks → Agentes (OBLIGATORIO)

| Task | Agente OBLIGATORIO | Prompt sugerido |
|---|---|---|
| 7.1 Crear MFE Transactions | `mfe-scaffolder` | "Scaffold mfe-transactions con pages: index.vue, [id].vue, disputes/index.vue, disputes/[id].vue, payment-links/index.vue, payment-links/create.vue, payment-links/[id].vue y stores: transactions.ts, disputes.ts, paymentLinks.ts" |
| 7.2 Store de transacciones | `vuex-to-pinia-migrator` | "Migra merchants-dashboard/store/modules/transactions.js a Pinia store en mfe-transactions" |
| 7.2 Composable de transacciones | `api-module-migrator` | "Migra merchants-dashboard/api/transactions.js a composable ofetch en mfe-transactions" |
| 9.1 Crear MFE Payouts | `mfe-scaffolder` | "Scaffold mfe-payouts con pages: balances.vue, create-payment.vue, transactions.vue, approvals.vue, favorites.vue, limits.vue, reports.vue y stores: balances.ts, payments.ts, approvals.ts" |
| 9.2 Composable de payouts | `api-module-migrator` | "Migra merchants-dashboard/api/payouts.js a composable ofetch en mfe-payouts" |
| 11.1 Crear MFE Settings | `mfe-scaffolder` | "Scaffold mfe-settings con pages: users/index.vue, users/create.vue, users/[id].vue, roles/index.vue, roles/create.vue, roles/[id].vue, keys.vue, my-account.vue, developers.vue y stores: users.ts, roles.ts" |
| 11.2 Store de usuarios | `vuex-to-pinia-migrator` | "Migra merchants-dashboard/store/modules/users.js a Pinia store en mfe-settings" |
| 11.2 Store de roles | `vuex-to-pinia-migrator` | "Migra merchants-dashboard/store/modules/userRoles.js a Pinia store en mfe-settings" |
| 11.2 Composable de users | `api-module-migrator` | "Migra merchants-dashboard/api/users.js a composable ofetch en mfe-settings" |
| 11.2 Composable de roles | `api-module-migrator` | "Migra merchants-dashboard/api/roles.js a composable ofetch en mfe-settings" |

### Reglas de Uso de Agentes

1. **SIEMPRE usar `mfe-scaffolder`** para crear la estructura de un MFE nuevo (tasks 7.1, 9.1, 11.1). NO crear manualmente nuxt.config.ts, app.vue, package.json de MFEs.
2. **SIEMPRE usar `vuex-to-pinia-migrator`** cuando la task involucre migrar un store Vuex a Pinia. NO convertir manualmente generators/yield/put a async/await.
3. **SIEMPRE usar `api-module-migrator`** cuando la task involucre crear un composable de API basado en un módulo legacy. NO convertir manualmente axios a ofetch.
4. **OPCIONALMENTE usar `legacy-code-analyzer`** antes de migrar un módulo complejo para documentar la complejidad y el plan.
5. Después de que un agente genere código, el orquestador PUEDE hacer ajustes menores (imports, integración), pero el grueso del trabajo lo hace el agente.

## Reglas para el Agente

Al tomar decisiones sobre este reto, el agente DEBE:
1. Priorizar el cierre de vulnerabilidades críticas (nuxt, axios) antes que cualquier feature nueva
2. Justificar cada decisión técnica contra los criterios de evaluación y sus pesos
3. Preferir migración incremental sobre rewrite completo
4. Documentar tradeoffs explícitamente (el jurado evalúa esto en Técnicas IA)
5. Mantener el dashboard funcional en cada paso (no romper producción)
6. Considerar que el jurado evaluará production-readiness y manejo de errores
7. Registrar decisiones arquitectónicas como ADRs en el skill `#reto2-decisiones`
8. Usar SIEMPRE los nombres de packages definidos: `@wompi/auth`, `@wompi/api-client`, `@wompi/ui`, `@wompi/i18n`, `@wompi/types`, `@wompi/event-bus`
9. Respetar la estructura `apps/` + `packages/` del monorepo — no inventar carpetas fuera de esta estructura
10. Usar `ofetch` (NO axios) para HTTP. Usar `Pinia` (NO Vuex). Usar `Nuxt UI v3` (NO Element UI, NO PrimeVue)
11. Todo código nuevo en Nuxt 4 con estructura `app/` — NO usar estructura legacy de Nuxt 2/3
12. Module Federation via `@module-federation/vite` — NO webpack
13. SPA mode (`ssr: false`) en todas las apps — NO SSR
14. Consultar `docs/design.md` para implementaciones detalladas de componentes y configuraciones
15. ANTES de implementar cualquier feature, leer el archivo equivalente en `merchants-dashboard/` para entender la lógica existente y replicarla fielmente (ver mapa de referencia arriba)
16. Las rutas al repo legacy usan el prefijo `merchants-dashboard/` (multi-root workspace). Ejemplo: `merchants-dashboard/api/transactions.js`
