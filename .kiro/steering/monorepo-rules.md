---
inclusion: always
---

# Monorepo — Reglas y Estructura

## Estructura

```
merchants-dashboard-mfe/
├── apps/           # Aplicaciones Nuxt 4
│   ├── shell/      # Host (Module Federation)
│   ├── mfe-transactions/
│   ├── mfe-payouts/
│   └── mfe-settings/
├── packages/       # Módulos compartidos
│   ├── types/      # @wompi/types — interfaces TypeScript
│   ├── auth/       # @wompi/auth — Cognito composable
│   ├── api-client/ # @wompi/api-client — ofetch + interceptors
│   ├── ui/         # @wompi/ui — componentes compartidos
│   ├── i18n/       # @wompi/i18n — traducciones
│   └── event-bus/  # @wompi/event-bus — comunicación cross-MFE
└── turbo.json
```

## Reglas

1. **Dependencias entre packages**: Los packages NO deben depender de las apps. Las apps dependen de los packages.
2. **Imports internos**: Usar el nombre del package (`@wompi/types`) en imports, NUNCA rutas relativas entre workspaces.
3. **Tipos compartidos**: Toda interfaz usada por más de un MFE va en `@wompi/types`.
4. **Componentes compartidos**: Todo componente usado por más de un MFE va en `@wompi/ui`.
5. **Estado compartido**: Auth y merchant context se comparten via `@wompi/auth` y `@wompi/event-bus`. NUNCA duplicar estado entre MFEs.
6. **Builds**: `turbo run build` compila packages antes que apps. Respetar el grafo de dependencias.
7. **Node version**: 20 LTS. Verificar con `.nvmrc`.
8. **Package manager**: pnpm. NUNCA usar npm o yarn en este proyecto.

## Agregar una dependencia

```bash
# A un package específico
pnpm add axios --filter @wompi/api-client

# A la raíz (dev dependency compartida)
pnpm add -D vitest -w

# Dependencia interna entre workspaces
pnpm add @wompi/types --filter @wompi/auth --workspace
```
