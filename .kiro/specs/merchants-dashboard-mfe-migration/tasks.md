# Plan de Implementación: Migración Merchants Dashboard a Micro-Frontends

## Visión General

Plan de implementación para migrar el Merchants Dashboard de Wompi desde un monolito Nuxt 1 hacia una arquitectura de micro-frontends con Nuxt 3, TypeScript, Module Federation y Turborepo. Las tareas están ordenadas por prioridad para un hackathon de 48 horas con 4 personas. Se usa TypeScript en todo el código nuevo.

## Tareas

- [ ] 1. Configurar monorepo con Turborepo y estructura base
  - [ ] 1.1 Crear estructura raíz del monorepo
    - Crear `package.json` raíz con scripts `dev`, `build`, `lint`, `test` usando Turborepo
    - Crear `pnpm-workspace.yaml` declarando `apps/*` y `packages/*`
    - Crear `turbo.json` con pipelines de `build`, `dev`, `lint`, `test` y dependencias entre tasks
    - Crear `tsconfig.base.json` con configuración TypeScript strict compartida
    - Crear `.gitignore`, `.nvmrc` (Node 20 LTS), `.env.example`
    - _Requerimientos: 1.1, 1.2, 1.3, 1.5_

  - [ ] 1.2 Crear package `@wompi/types` con interfaces TypeScript compartidas
    - Crear `packages/types/package.json` con nombre `@wompi/types`
    - Crear `packages/types/tsconfig.json` extendiendo `tsconfig.base.json`
    - Crear interfaces: `merchant.ts` (Merchant, ApiEnvironment), `transaction.ts` (Transaction, TransactionStatus, TransactionFilters, PaymentMethod), `payout.ts` (PayoutBalance, PayoutTransaction, PayoutStatus, PayoutDestination, CreatePayoutRequest), `user.ts` (DashboardUser, UserRole, Permission, CreateUserRequest), `auth.ts` (CognitoTokens, AuthState, CognitoUserInfo, LoginCredentials, ApiClientConfig)
    - Crear `packages/types/index.ts` re-exportando todos los tipos
    - _Requerimientos: 15.4, 19.3_

  - [ ] 1.3 Crear package `@wompi/event-bus` con comunicación tipada cross-MFE
    - Crear `packages/event-bus/package.json` con dependencia `mitt`
    - Crear `packages/event-bus/types.ts` con `EventMap` tipando eventos: `merchant:changed`, `environment:changed`, `auth:logout`, `auth:session-refreshed`
    - Crear `packages/event-bus/bus.ts` con singleton mitt, registry de suscripciones por MFE y función `cleanup()` para limpieza automática
    - Crear `packages/event-bus/index.ts` exportando `useEventBus`
    - _Requerimientos: 20.1, 20.2, 20.3, 20.4_


  - [ ]* 1.4 Escribir property tests para `@wompi/event-bus`
    - **Property 9: Event Bus entrega eventos a todos los suscriptores**
    - Para cualquier tipo de evento y N suscriptores, todos los handlers reciben el payload exacto emitido
    - **Valida: Requerimiento 20.1**

  - [ ]* 1.5 Escribir property test de cleanup del Event Bus
    - **Property 10: Event Bus limpia suscripciones al desmontar MFE**
    - Después de `cleanup()`, ninguna suscripción registrada bajo ese `mfeId` recibe eventos futuros
    - **Valida: Requerimiento 20.4**

- [ ] 2. Implementar packages compartidos de auth y API client
  - [ ] 2.1 Crear package `@wompi/auth` con composable de Cognito
    - Crear `packages/auth/package.json` con dependencia `amazon-cognito-identity-js`
    - Crear `packages/auth/utils/cognito.ts` con configuración del UserPool (env vars `VITE_DASHBOARD_USER_POOL_ID`, `VITE_DASHBOARD_CLIENT_ID`)
    - Crear `packages/auth/composables/useAuth.ts` con: `login()` (authenticateUser con USER_PASSWORD_AUTH), `refreshSession()` (refresh con CognitoRefreshToken), `logout()` (limpiar localStorage/sessionStorage, resetear estado, redirect a /login), `saveSession()` (persistir tokens en localStorage)
    - Estado global reactivo singleton: `user`, `tokens`, `isAuthenticated`
    - Crear `packages/auth/index.ts` exportando `useAuth`
    - _Requerimientos: 3.1, 3.2, 3.3, 3.4, 3.5_

  - [ ] 2.2 Crear package `@wompi/api-client` con ofetch e interceptors
    - Crear `packages/api-client/package.json` con dependencia `ofetch`
    - Crear `packages/api-client/client.ts` con función `createApiClient()`: base URL dinámica (sandbox/producción) + prefijo `/dashboard`, interceptor `onRequest` que ejecuta refresh de sesión y adjunta headers `Authorization: Bearer {token}` y `User-Principal-Id`, interceptor `onResponseError` que invoca logout en HTTP 401, timeout configurable (default 60s)
    - Crear `packages/api-client/index.ts` con factory `useApiClient()` que conecta auth con api-client
    - Crear `packages/api-client/types.ts` con tipos de respuestas API
    - _Requerimientos: 15.1, 15.2, 15.3, 15.4, 15.5_

  - [ ]* 2.3 Escribir property test para API Client factory
    - **Property 1: API Client factory produce instancias correctamente configuradas**
    - Para cualquier combinación de ambiente y base URL, `createApiClient` produce instancia con `baseURL` = URL + `/dashboard`
    - **Valida: Requerimiento 15.1**

  - [ ]* 2.4 Escribir property test para interceptor de headers
    - **Property 2: Request interceptor adjunta headers de autenticación correctos**
    - Para cualquier token y User-Principal-Id válidos, el interceptor adjunta los headers correctos
    - **Valida: Requerimiento 15.2**

- [ ] 3. Checkpoint — Verificar packages base
  - Ejecutar `turbo run build` y verificar que todos los packages compilan sin errores
  - Verificar que los tipos se resuelven correctamente entre packages
  - Asegurar que todos los tests pasan, preguntar al usuario si surgen dudas


- [ ] 4. Implementar packages compartidos de UI e i18n
  - [ ] 4.1 Crear package `@wompi/ui` con componentes compartidos
    - Crear `packages/ui/package.json` con dependencias `@nuxt/ui` y `tailwindcss`
    - Crear `packages/ui/tailwind.config.ts` con colores, tipografía y espaciado del design system de Wompi
    - Crear componentes reutilizables: `WDataTable.vue` (tabla con paginación), `WFilterPanel.vue` (panel de filtros), `WStatusBadge.vue` (badge de estado con colores por tipo), `WCopyButton.vue` (botón copiar al portapapeles)
    - Crear `packages/ui/index.ts` exportando todos los componentes
    - _Requerimientos: 16.1, 16.2, 16.3_

  - [ ] 4.2 Crear package `@wompi/i18n` con traducciones
    - Crear `packages/i18n/package.json`
    - Crear archivos de traducción `locales/es_CO.json`, `locales/es_PA.json`, `locales/es_GT.json` migrando las traducciones existentes del legacy (`merchants-dashboard/locales/`)
    - Crear `packages/i18n/index.ts` con configuración de locale basada en env var `VITE_I18N_LOCALE` y fallback `VITE_I18N_FALLBACK_LOCALE`
    - _Requerimientos: 17.1, 17.2, 17.3_

- [ ] 5. Implementar Shell App — Layout, Auth y Module Federation
  - [ ] 5.1 Crear app Shell con Nuxt 3 y configuración Module Federation host
    - Crear `apps/shell/` con `nuxt.config.ts` configurando: `ssr: false` (SPA mode), `@nuxt/ui` module, `@module-federation/vite` como host declarando remotes (mfe-transactions, mfe-payouts, mfe-settings) con URLs configurables por env vars, shared singletons (vue, pinia, vue-i18n, ofetch)
    - Crear `apps/shell/package.json` con dependencias de los packages compartidos (`@wompi/auth`, `@wompi/api-client`, `@wompi/ui`, `@wompi/i18n`, `@wompi/event-bus`, `@wompi/types`)
    - Crear `apps/shell/tsconfig.json` extendiendo `tsconfig.base.json`
    - _Requerimientos: 5.1, 5.3_

  - [ ] 5.2 Implementar layout del Shell (sidebar + header + content area)
    - Crear `apps/shell/layouts/default.vue` con tres zonas: sidebar lateral, header superior, área de contenido principal
    - Crear `apps/shell/components/AppSidebar.vue` con enlaces de navegación agrupados por dominio: Transacciones (transacciones, disputas, payment links), Payouts (balances, crear pago, transacciones payouts, aprobaciones, favoritos, límites, reportes), Configuración (usuarios, roles, llaves API, mi cuenta, developers)
    - Crear `apps/shell/components/AppHeader.vue` mostrando nombre del merchant activo, ambiente actual (sandbox/producción), menú desplegable del usuario
    - _Requerimientos: 2.1, 2.2, 2.4_

  - [ ] 5.3 Implementar autenticación con Cognito en el Shell
    - Crear `apps/shell/pages/login.vue` con formulario de login que usa `useAuth().login()`
    - Crear `apps/shell/middleware/auth.global.ts` que redirige rutas protegidas a `/login` si no hay sesión activa (rutas públicas: `/login`, `/password-recovery`, `/register`, `/confirm-code`)
    - Integrar `useAuth()` en el layout para mostrar estado de autenticación y botón de logout
    - _Requerimientos: 3.1, 3.4, 3.6, 2.5_

  - [ ] 5.4 Implementar selector de merchant y toggle de ambiente
    - Crear `apps/shell/composables/useMerchantContext.ts` con: `selectMerchant()` que actualiza `userPrincipalID` en sessionStorage y emite `merchant:changed` via Event Bus, `toggleEnvironment()` que alterna base URL entre producción y sandbox y emite `environment:changed`
    - Integrar selector de merchant en el header
    - Integrar toggle de ambiente sandbox/producción en el header
    - _Requerimientos: 4.1, 4.2, 4.3, 4.4_

  - [ ]* 5.5 Escribir property test para selección de merchant
    - **Property 3: Selección de merchant actualiza contexto y emite evento**
    - Para cualquier merchant ID válido, `selectMerchant` actualiza sessionStorage y emite evento correcto
    - **Valida: Requerimiento 4.2**

  - [ ] 5.6 Implementar MfeLoader y routing catch-all para MFEs
    - Crear `apps/shell/components/MfeLoader.vue` con carga dinámica via `loadRemote()` de `@module-federation/runtime`, estado de loading con spinner, estado de error con mensaje descriptivo y botón reintentar
    - Crear `apps/shell/pages/[...slug].vue` con mapa de rutas a MFEs: `transactions|disputes|payment-links` → mfe-transactions, `payouts|balances|...` → mfe-payouts, `users|roles|keys|...` → mfe-settings
    - Crear `apps/shell/pages/index.vue` con redirect a `/transactions`
    - _Requerimientos: 2.3, 5.2, 5.4_


- [ ] 6. Checkpoint — Shell funcional con auth y layout
  - Verificar que el Shell arranca con `pnpm dev`, muestra layout completo (sidebar + header + content)
  - Verificar flujo de login con Cognito (login → dashboard → logout)
  - Verificar que el selector de merchant y toggle de ambiente funcionan
  - Asegurar que todos los tests pasan, preguntar al usuario si surgen dudas

- [ ] 7. Implementar MFE Transactions
  - [ ] 7.1 Crear app MFE Transactions con Nuxt 3 y Module Federation remote
    - Crear `apps/mfe-transactions/` con `nuxt.config.ts` configurando: `ssr: false`, `@nuxt/ui`, `@module-federation/vite` como remote exponiendo `./TransactionsApp` desde `app.vue`, shared singletons
    - Crear `apps/mfe-transactions/package.json` con dependencias de packages compartidos
    - Crear `apps/mfe-transactions/app.vue` como entry point con suscripción a eventos del Shell (`merchant:changed`, `environment:changed`) y cleanup en `onUnmounted`
    - _Requerimientos: 5.1, 5.3, 5.5_

  - [ ] 7.2 Implementar store y composable de transacciones
    - Crear `apps/mfe-transactions/stores/transactions.ts` con Pinia store: `filters` (ref con TransactionFilters), `loading` (ref boolean), `transactions` (ref array), `setFilter()`, `getReport()` async con `removeEmpty()` para limpiar filtros vacíos
    - Crear `apps/mfe-transactions/composables/useTransactionsApi.ts` con funciones: `getTransactions()`, `getTransaction()`, `downloadReport()`
    - _Requerimientos: 6.1, 6.2, 6.3, 19.1, 19.2, 19.3_

  - [ ]* 7.3 Escribir property test para transformación de filtros
    - **Property 4: Transformación de filtros elimina valores vacíos**
    - Para cualquier objeto de filtros, `removeEmpty` retorna solo claves con valores no vacíos y no `false`
    - **Valida: Requerimiento 6.3**

  - [ ]* 7.4 Escribir property test para paridad Vuex → Pinia
    - **Property 8: Paridad funcional Vuex → Pinia**
    - Para cualquier secuencia de operaciones `setFilter`, el Pinia store produce el mismo estado de filtros
    - **Valida: Requerimientos 19.1, 19.2**

  - [ ] 7.5 Implementar páginas de lista y detalle de transacciones
    - Crear `apps/mfe-transactions/pages/index.vue` con tabla paginada (columnas: ID, referencia, monto, método de pago, estado, canal, fecha), panel de filtros (ID, referencia, email, estado, método de pago, canal), botón de descarga de reportes CSV/Excel
    - Crear `apps/mfe-transactions/pages/[id].vue` con vista de detalle: información completa de la transacción, método de pago, timeline de estados, datos del cliente
    - _Requerimientos: 6.1, 6.2, 6.3, 6.4, 6.5_

  - [ ] 7.6 Implementar páginas de disputas
    - Crear `apps/mfe-transactions/pages/disputes/index.vue` con lista paginada de disputas (estado, monto, fecha, transacción asociada)
    - Crear `apps/mfe-transactions/pages/disputes/[id].vue` con detalle de disputa, transacción asociada y timeline de estados
    - _Requerimientos: 7.1, 7.2_

  - [ ] 7.7 Implementar páginas de payment links
    - Crear `apps/mfe-transactions/stores/paymentLinks.ts` con Pinia store para payment links
    - Crear `apps/mfe-transactions/pages/payment-links/index.vue` con lista paginada (nombre, monto, estado, fecha)
    - Crear `apps/mfe-transactions/pages/payment-links/create.vue` con formulario de creación (nombre, descripción, monto, moneda, fecha expiración) y visualización del link generado con botón copiar
    - Crear `apps/mfe-transactions/pages/payment-links/[id].vue` con detalle del link, URL y transacciones asociadas
    - _Requerimientos: 8.1, 8.2, 8.3, 8.4_

  - [ ]* 7.8 Escribir property test para creación de payment link
    - **Property 5: Creación de payment link con datos válidos invoca API correctamente**
    - Para cualquier payload válido (nombre no vacío, monto > 0, moneda válida), el formulario invoca el endpoint con el payload exacto
    - **Valida: Requerimiento 8.3**


- [ ] 8. Checkpoint — MFE Transactions integrado con Shell
  - Verificar que el Shell carga MFE Transactions via Module Federation
  - Verificar flujo completo: login → lista transacciones → filtrar → ver detalle
  - Verificar navegación a disputas y payment links
  - Verificar que cambio de merchant/ambiente recarga datos en el MFE
  - Asegurar que todos los tests pasan, preguntar al usuario si surgen dudas

- [ ] 9. Implementar MFE Payouts
  - [ ] 9.1 Crear app MFE Payouts con Nuxt 3 y Module Federation remote
    - Crear `apps/mfe-payouts/` con `nuxt.config.ts` configurando Module Federation remote exponiendo `./PayoutsApp`
    - Crear `apps/mfe-payouts/package.json` y `apps/mfe-payouts/app.vue` con suscripción a eventos del Shell
    - _Requerimientos: 5.1, 5.3_

  - [ ] 9.2 Implementar stores y composable de payouts
    - Crear `apps/mfe-payouts/stores/balances.ts` con Pinia store para balances y auto-pago
    - Crear `apps/mfe-payouts/stores/payments.ts` con Pinia store para creación de pagos
    - Crear `apps/mfe-payouts/stores/approvals.ts` con Pinia store para aprobaciones
    - Crear `apps/mfe-payouts/composables/usePayoutsApi.ts` con funciones: `getBalance()`, `updateAutoPayment()`, `createPayout()`, `getLimits()`
    - _Requerimientos: 9.1, 10.1, 11.1, 19.1, 19.3_

  - [ ] 9.3 Implementar página de balances y dashboard
    - Crear `apps/mfe-payouts/pages/balances.vue` con dashboard de balance disponible, estado de auto-pago con toggle para activar/desactivar
    - _Requerimientos: 9.1, 9.2, 9.3_

  - [ ] 9.4 Implementar página de crear pago / dispersión
    - Crear `apps/mfe-payouts/pages/create-payment.vue` con formulario: destinatario (cuenta bancaria o favorito), monto, concepto, tipo de pago. Confirmación con ID de transacción. Notificación si requiere aprobación.
    - _Requerimientos: 10.1, 10.2, 10.3_

  - [ ]* 9.5 Escribir property test para creación de pago
    - **Property 6: Creación de pago con datos válidos invoca API correctamente**
    - Para cualquier payload válido (monto > 0, cuenta bancaria válida, concepto no vacío), el formulario invoca el endpoint correctamente
    - **Valida: Requerimiento 10.2**

  - [ ] 9.6 Implementar páginas de transacciones payouts, aprobaciones, favoritos, límites y reportes
    - Crear `apps/mfe-payouts/pages/transactions.vue` con lista paginada de transacciones payouts con filtros por estado, fecha y monto
    - Crear `apps/mfe-payouts/pages/approvals.vue` con vista de pagos pendientes de aprobación (aprobar/rechazar)
    - Crear `apps/mfe-payouts/pages/favorites.vue` con lista de destinatarios favoritos (agregar, editar, eliminar)
    - Crear `apps/mfe-payouts/pages/limits.vue` con configuración de límites de dispersión
    - Crear `apps/mfe-payouts/pages/reports.vue` con descarga de reportes según filtros
    - _Requerimientos: 11.1, 11.2, 11.3, 11.4, 11.5_

- [ ] 10. Checkpoint — MFE Payouts integrado con Shell
  - Verificar que el Shell carga MFE Payouts via Module Federation
  - Verificar flujo: balances → crear pago → ver transacciones → aprobar
  - Verificar que cambio de merchant/ambiente recarga datos
  - Asegurar que todos los tests pasan, preguntar al usuario si surgen dudas


- [ ] 11. Implementar MFE Settings
  - [ ] 11.1 Crear app MFE Settings con Nuxt 3 y Module Federation remote
    - Crear `apps/mfe-settings/` con `nuxt.config.ts` configurando Module Federation remote exponiendo `./SettingsApp`
    - Crear `apps/mfe-settings/package.json` y `apps/mfe-settings/app.vue` con suscripción a eventos del Shell
    - _Requerimientos: 5.1, 5.3_

  - [ ] 11.2 Implementar stores de usuarios y roles
    - Crear `apps/mfe-settings/stores/users.ts` con Pinia store para CRUD de usuarios
    - Crear `apps/mfe-settings/stores/roles.ts` con Pinia store para CRUD de roles
    - _Requerimientos: 12.1, 13.1, 19.1, 19.3_

  - [ ] 11.3 Implementar páginas de gestión de usuarios
    - Crear `apps/mfe-settings/pages/users/index.vue` con lista paginada de usuarios (nombre, email, rol, estado)
    - Crear `apps/mfe-settings/pages/users/create.vue` con formulario de creación (nombre, email, teléfono, rol)
    - Crear `apps/mfe-settings/pages/users/[id].vue` con detalle del usuario, edición de datos y opción de deshabilitar cuenta
    - _Requerimientos: 12.1, 12.2, 12.3, 12.4, 12.5_

  - [ ]* 11.4 Escribir property test para creación de usuario
    - **Property 7: Creación de usuario con datos válidos invoca API correctamente**
    - Para cualquier payload válido (nombre no vacío, email válido, teléfono, rol_id existente), el formulario invoca el endpoint correctamente
    - **Valida: Requerimiento 12.3**

  - [ ] 11.5 Implementar páginas de gestión de roles
    - Crear `apps/mfe-settings/pages/roles/index.vue` con lista de roles (nombre, cantidad de usuarios)
    - Crear `apps/mfe-settings/pages/roles/create.vue` con formulario (nombre del rol, selector de permisos agrupados por módulo)
    - Crear `apps/mfe-settings/pages/roles/[id].vue` con vista de edición con permisos actuales seleccionados
    - _Requerimientos: 13.1, 13.2, 13.3, 13.4_

  - [ ] 11.6 Implementar páginas de llaves API, mi cuenta y developers
    - Crear `apps/mfe-settings/pages/keys.vue` con llaves de API (pública y privada) diferenciando sandbox/producción, botón copiar al portapapeles
    - Crear `apps/mfe-settings/pages/my-account.vue` con formulario para actualizar contraseña, email y teléfono
    - Crear `apps/mfe-settings/pages/developers.vue` con información de integración y configuración de webhooks
    - _Requerimientos: 14.1, 14.2, 14.3_

- [ ] 12. Checkpoint — MFE Settings integrado con Shell
  - Verificar que el Shell carga MFE Settings via Module Federation
  - Verificar flujo: lista usuarios → crear usuario → editar → deshabilitar
  - Verificar gestión de roles y llaves API
  - Asegurar que todos los tests pasan, preguntar al usuario si surgen dudas

- [ ] 13. Verificación de seguridad y polish final
  - [ ] 13.1 Verificar 0 vulnerabilidades en el stack nuevo
    - Ejecutar escaneo de dependencias y verificar que no hay vulnerabilidades críticas o altas
    - Verificar que todas las dependencias son versiones actuales y soportadas: Nuxt 3.x, Vue 3.x, TypeScript 5.x, Node 20 LTS, Vite 5.x+
    - Verificar que no se requiere archivo `.snyk` con vulnerabilidades ignoradas
    - _Requerimientos: 18.1, 18.2, 18.3, 18.4_

  - [ ] 13.2 Integración final y wiring cross-MFE
    - Verificar que los 3 MFEs cargan correctamente bajo el Shell con Module Federation
    - Verificar que el deploy independiente funciona (cambiar un MFE sin rebuild del Shell)
    - Verificar flujo E2E completo: login → transacciones → crear payout → gestionar usuarios
    - Verificar que el cambio de merchant y ambiente se propaga correctamente a todos los MFEs
    - _Requerimientos: 5.2, 5.5, 4.2, 4.4, 20.3_

  - [ ]* 13.3 Escribir unit tests para flujos críticos
    - Test: auth middleware redirige a `/login` sin sesión activa
    - Test: HTTP 401 dispara logout automático
    - Test: MfeLoader muestra error cuando remote no está disponible
    - Test: toggle de ambiente cambia base URL correctamente
    - _Requerimientos: 2.5, 3.6, 5.4, 4.3_

- [ ] 14. Checkpoint final — MVP completo
  - Verificar que todos los tests pasan
  - Verificar flujo E2E: login → transacciones → crear payout → gestionar usuarios
  - Verificar 0 vulnerabilidades en el stack
  - Verificar TypeScript strict en todo el código nuevo
  - Asegurar que todos los tests pasan, preguntar al usuario si surgen dudas

## Notas

- Las tareas marcadas con `*` son opcionales y pueden omitirse para un MVP más rápido
- Cada tarea referencia requerimientos específicos para trazabilidad
- Los checkpoints aseguran validación incremental en cada fase
- Los property tests validan propiedades universales de correctitud definidas en el diseño
- Los unit tests validan ejemplos específicos y edge cases
- El orden de prioridad es: Shell + packages → MFE Transactions → MFE Payouts → MFE Settings → Polish
- Para el hackathon: si el tiempo apremia, priorizar Shell + Transactions funcional sobre completar los 3 MFEs
