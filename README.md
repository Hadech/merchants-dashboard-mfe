# Merchants Dashboard MFE — Wompi Hackathon 2026

Migración del Merchants Dashboard de Wompi a arquitectura micro-frontend con Nuxt 3 + Module Federation + TypeScript + Turborepo.

## Documentación

| Documento | Descripción |
|-----------|-------------|
| [SPEC_DESCRIPTION.md](docs/SPEC_DESCRIPTION.md) | Visión general del proyecto, decisiones técnicas, stack, scope del MVP |
| [requirements.md](docs/requirements.md) | 20 requerimientos funcionales y no funcionales |
| [design.md](docs/design.md) | Arquitectura técnica, diagramas, código de ejemplo, correctness properties |
| [tasks.md](docs/tasks.md) | Plan de implementación con 14 tareas priorizadas para 48h |

## Equipo

- **Líder**: Monorepo + Shell + Module Federation + Demo
- **Dev 1**: Shared packages (auth, api) + MFE Transactions
- **Dev 2**: Shared packages (ui, i18n) + MFE Payouts
- **QA**: Testing E2E + datos de prueba + demo rehearsal

## Stack

- Nuxt 3 (Vue 3) · TypeScript · Pinia · Tailwind CSS · Nuxt UI
- Module Federation (Vite) · Turborepo · pnpm workspaces
- Node 20 LTS · Vitest · Playwright
