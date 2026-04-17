---
inclusion: always
---
# Steering Global — Merchants Dashboard (Wompi)

## Resumen del Proyecto
Dashboard SPA para comercios de Wompi (pasarela de pagos). Construido con Nuxt 1.x (modo SPA) + Vue 2.7 (Options API). El proyecto es legacy con dependencias EOL — cualquier cambio debe ser conservador y compatible con el stack existente.

## Comandos Principales
- `npm run dev` — Servidor de desarrollo (requiere `.env`, usar `dotenv`)
- `npm run build` — Build de producción (`dotenv nuxt build`)
- `npm run lint` — Lint con ESLint (`eslint --ext .js,.vue`)
- `npm run lint:fix` — Autofix de lint
- `npm test` — Ejecutar tests con Jest
- `npm run coverage` — Tests con reporte de cobertura
- Node requerido: **12.22.12** (ver `.nvmrc`)

## Reglas de Código

### Estilo y Formato
- Prettier: semicolons habilitados, single quotes, printWidth 120 (ver `.prettierrc`)
- ESLint: `plugin:vue/recommended` + `standard` + `prettier` (ver `.eslintrc.js`)
- Siempre ejecutar `npm run lint` antes de considerar un cambio terminado
- No usar `var`, preferir `const` sobre `let`

### Vue / Componentes
- Usar exclusivamente **Options API** (NO Composition API, NO `<script setup>`)
- Templates en **Pug** (no HTML plano) — respetar la indentación Pug existente
- Componentes `.vue` con estructura: `<template lang="pug">`, `<script>`, `<style>`
- UI con **Element UI 2.x** — usar sus componentes (`el-button`, `el-table`, `el-dialog`, etc.)
- Estilos en **SASS** (indentado, no SCSS) — archivos parciales en `styles/`
- Variables globales de estilo en `styles/_variables.sass`
- Mixins SASS en `styles/_mixins.sass`
- Overrides de Element UI en `styles/_element-overrides.sass`

### Estado (Vuex)
- Store principal en `store/index.js` con `state.js`, `mutations.js`, `actions.js`, `getters.js`
- Módulos Vuex en `store/modules/` — cada módulo es un archivo independiente
- Side effects con **vuex-saga** (generators) — NO usar async/await directamente en actions del store principal
- Persistencia con `vuex-persistedstate` (configurado en `plugins/persisted-state.js`)
- Para nuevos módulos, seguir el patrón existente en `store/modules/`

### API / HTTP
- Cliente HTTP: **Axios 0.x** — instancia centralizada en `api/api.js`
- Interceptors en `api/interceptors.js` (manejo de tokens, errores)
- Cada dominio de API tiene su propio archivo en `api/` (ej: `api/transactions.js`, `api/payouts.js`)
- Server middleware Express en `serverMiddleware/api/` actúa como proxy al API Gateway
- Para nuevos endpoints, crear o extender archivos en `api/`

### Autenticación
- AWS Cognito via `amazon-cognito-identity-js`
- Lógica en `utils/awsCognito.js` y `utils/auth.js`
- Dos user pools: actual y legacy (migración en curso)
- Middlewares de auth en `middleware/auth.js`, `middleware/confirmCode.js`, etc.
- Feature flags controlan flujos de autenticación (`FLAG_AUTHENTICATION_BY_NICKNAME`, etc.)

### Permisos / Roles
- CASL (`@casl/ability` + `@casl/vue`) para control de acceso
- Plugin en `plugins/ability.js`
- Helpers en `utils/userPermissionsHelpers.js`
- Mixins: `mixins/permissionMixin.js`, `mixins/permissionFormMixin.js`

### Internacionalización (i18n)
- `vue-i18n 8.x` configurado en `plugins/i18n.js`
- Archivos de traducción en `locales/`: `es_CO.json`, `es_GT.json`, `es_PA.json`
- Usar `$t('key')` en templates, nunca strings hardcodeados para texto visible al usuario
- Nuevas traducciones deben agregarse en TODOS los archivos de locale

### Routing / Páginas
- Routing basado en filesystem de Nuxt — carpetas en `pages/`
- Rutas dinámicas con `_param.vue` (ej: `pages/transactions/_id.vue`)
- Layouts en `layouts/` — `default.vue` es el principal con sidebar
- Middlewares de ruta en `middleware/`

### Mixins
- Mixins compartidos en `mixins/` — reutilizar los existentes antes de crear nuevos
- Mixins principales: `formMixin`, `loginMixin`, `permissionMixin`, `roleMixin`, `otpMixin`

### Utilidades
- Helpers en `utils/` — verificar si ya existe una utilidad antes de crear una nueva
- Configuración por país en `utils/countryConfig.js`
- Validadores custom en `utils/customValidators.js`
- Expresiones regulares en `utils/regularExpressions.js`
- Formatters en `utils/formatter.js` y `utils/filters.js`

## Tests
- Framework: **Jest 23** con `@vue/test-utils 1.x`
- Tests en `tests/unit/` — estructura espejo del proyecto
- Archivos de test: `*.spec.js`
- Mocks en `tests/unit/__mocks__/`
- Alias `@/` y `~/` mapeados a la raíz del proyecto
- Ejecutar: `npm test` (single run, no watch)
- Al crear o modificar un componente/utilidad, verificar si existe un test asociado

## Estructura de Directorios (Referencia Rápida)
```
pages/          → Rutas (filesystem-based routing de Nuxt)
components/     → Componentes Vue (carpeta por feature + componentes sueltos)
store/          → Vuex store principal + modules/
api/            → Módulos de API (axios)
middleware/     → Middlewares de ruta (auth, redirects)
plugins/        → Plugins Nuxt (element-ui, i18n, ability, etc.)
mixins/         → Mixins Vue compartidos
utils/          → Utilidades y helpers
layouts/        → Layouts de Nuxt
locales/        → Archivos de traducción (es_CO, es_GT, es_PA)
styles/         → SASS global (variables, mixins, overrides)
serverMiddleware/ → Express proxy middleware
assets/         → Imágenes, iconos, fuentes
static/         → Archivos estáticos (JS tracking, favicon)
tests/unit/     → Tests unitarios (Jest)
```

## Variables de Entorno
- Configuradas via `.env` (ver `.env.example` para referencia)
- Inyectadas al cliente via `nuxt.config.js > env`
- Feature flags importantes: `FLAG_AUTHENTICATION_BY_NICKNAME`, `PAYOUTS_AVAILABLE`, `TWO_FACTOR_AUTHENTICATION_ENABLED`, `VP_ROLE_PERMISSION_ACTIVE`, `ENABLE_DATAPHONE_PURCHASE_FLOW`
- Nunca commitear valores reales de `.env`

## Infraestructura
- Deploy: S3 + CloudFront (SPA estático generado con `nuxt generate`)
- Auth: AWS Cognito (2 user pools: actual + legacy en migración)
- Backend: API Gateway → Lambda (el dashboard consume vía serverMiddleware proxy)
- Tracking: GTM, Google Analytics, Hotjar, Salesforce Evergage, Zendesk, Facebook Pixel, Twitter, LinkedIn

## Restricciones Importantes
- NO actualizar dependencias mayores sin validación explícita (proyecto legacy con 85+ vulnerabilidades conocidas)
- NO introducir Composition API, `<script setup>`, TypeScript ni Vue 3
- NO reemplazar Element UI por otra librería de componentes
- NO cambiar de SASS indentado a SCSS en archivos existentes
- NO usar `async/await` en el store principal (usar vuex-saga generators)
- Mantener compatibilidad con Node 12.22.12
- Cualquier nuevo paquete npm debe ser compatible con Webpack 3
