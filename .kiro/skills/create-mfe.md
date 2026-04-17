# Skill: Crear un nuevo MFE

## Descripción
Genera la estructura completa de un nuevo micro-frontend con Nuxt 4 + Module Federation.

## Cuándo usarlo
- "Crea el MFE de transactions"
- "Scaffold del micro-frontend de payouts"
- "Inicializa el MFE de settings"

## Instrucciones

### Entrada
Nombre del MFE y dominio funcional que cubre.

### Proceso

1. **Crear la estructura de directorios**:

```
apps/mfe-{nombre}/
├── app/
│   ├── app.vue
│   ├── pages/
│   ├── components/
│   ├── composables/
│   └── stores/
├── nuxt.config.ts
├── package.json
└── tsconfig.json
```

2. **Crear `package.json`**:
```json
{
  "name": "@wompi/mfe-{nombre}",
  "private": true,
  "scripts": {
    "dev": "nuxt dev --port {puerto}",
    "build": "nuxt build",
    "lint": "eslint .",
    "test": "vitest run"
  },
  "dependencies": {
    "@wompi/auth": "workspace:*",
    "@wompi/api-client": "workspace:*",
    "@wompi/ui": "workspace:*",
    "@wompi/i18n": "workspace:*",
    "@wompi/event-bus": "workspace:*",
    "@wompi/types": "workspace:*"
  }
}
```

Puertos: shell=3000, mfe-transactions=3001, mfe-payouts=3002, mfe-settings=3003

3. **Crear `nuxt.config.ts`** con Module Federation remote:
```typescript
import { createModuleFederationConfig } from '@module-federation/vite'

export default defineNuxtConfig({
  ssr: false,
  modules: ['@nuxt/ui'],
  vite: {
    plugins: [
      createModuleFederationConfig({
        name: 'mfe-{nombre}',
        filename: 'remoteEntry.js',
        exposes: {
          './{Nombre}App': './app/app.vue',
        },
        shared: {
          vue: { singleton: true },
          pinia: { singleton: true },
          'vue-i18n': { singleton: true },
          ofetch: { singleton: true },
        },
      }),
    ],
  },
})
```

4. **Crear `app/app.vue`** con suscripción a eventos:
```vue
<template>
  <NuxtPage />
</template>

<script setup lang="ts">
import { useEventBus } from '@wompi/event-bus'

const { on, cleanup } = useEventBus('mfe-{nombre}')

on('merchant:changed', () => {
  // Recargar datos
})

on('environment:changed', () => {
  // Recargar datos
})

onUnmounted(cleanup)
</script>
```

5. **Crear página index** básica como placeholder

### Salida
MFE funcional que arranca con `pnpm dev --filter @wompi/mfe-{nombre}` y se registra como remote en Module Federation.
