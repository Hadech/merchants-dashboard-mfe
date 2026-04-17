---
inclusion: always
---

# Nuxt 4 — Convenciones del Proyecto

## Estructura de Directorios (Nuxt 4)

Nuxt 4 usa la estructura `app/` por defecto. Todo el código de la aplicación vive dentro de `app/`:

```
apps/{nombre}/
├── app/
│   ├── app.vue              # Entry point
│   ├── pages/               # Rutas
│   ├── components/          # Componentes Vue
│   ├── composables/         # Lógica reutilizable
│   ├── layouts/             # Layouts
│   ├── middleware/           # Route middleware
│   ├── plugins/             # Plugins
│   └── utils/               # Utilidades
├── shared/                  # Código compartido app/server
├── server/                  # Server routes (si aplica)
└── nuxt.config.ts           # Configuración
```

NUNCA poner pages, components, composables, layouts o middleware en la raíz de la app. Siempre dentro de `app/`.

## Convenciones de Código

### Composables
- Prefijo `use` siempre: `useAuth`, `useApiClient`, `useMerchantContext`
- Retornar objetos con propiedades reactivas y funciones
- Tipar todo con TypeScript

### Stores (Pinia)
- Usar setup stores (Composition API), NO option stores
- Prefijo `use` + sufijo `Store`: `useTransactionsStore`
- Ubicar en `app/stores/` dentro de cada MFE

### Componentes
- PascalCase para nombres: `AppSidebar.vue`, `WDataTable.vue`
- Componentes compartidos prefijados con `W`: `WDataTable`, `WFilterPanel`, `WStatusBadge`
- Props tipadas con `defineProps<T>()`
- Emits tipados con `defineEmits<T>()`

### API Calls
- Usar `useApiClient()` del package `@wompi/api-client`, NUNCA axios directo
- Composables de API por dominio: `useTransactionsApi()`, `usePayoutsApi()`
- Tipar todas las respuestas con interfaces de `@wompi/types`

### Imports
- Usar auto-imports de Nuxt 4 (ref, computed, watch, navigateTo, etc.)
- Imports explícitos para packages compartidos: `import { useAuth } from '@wompi/auth'`
- NUNCA usar `require()`, solo ESM imports

## SPA Mode

Este proyecto corre en modo SPA (`ssr: false`), igual que el legacy. No usar features de SSR como `useRequestHeaders`, `setCookie` del server, etc.

## Module Federation

- El Shell es el HOST, los MFEs son REMOTES
- Cada MFE expone su `app/app.vue` como entry point
- Shared dependencies (vue, pinia, vue-i18n, ofetch) son SINGLETON
- Si un MFE necesita comunicarse con el Shell, usar `@wompi/event-bus`
