# Skill: Decisiones Arquitectónicas — Reto 2 Hackathon

Cuando se invoque este skill, el agente debe considerar todas las decisiones registradas aquí como contexto vinculante para las tareas en curso. Cada decisión es un ADR (Architecture Decision Record) que justifica una elección técnica frente a los criterios de evaluación del hackathon.

## Cómo usar este archivo
- El equipo y el agente registran aquí cada decisión técnica relevante
- Formato: ADR ligero (contexto → decisión → justificación → consecuencias)
- Referenciar el criterio de evaluación que la decisión impacta
- Las specs pueden referenciar este archivo con `#[[file:.kiro/skills/reto2-decisiones.md]]`

## Instrucciones para el Agente
Cuando este skill esté activo:
1. Consulta las decisiones ya tomadas ANTES de proponer cambios
2. No contradigas una decisión existente sin justificación explícita
3. Si una tarea requiere una nueva decisión arquitectónica, regístrala aquí
4. Vincula cada decisión al criterio de evaluación que impacta (ver `.kiro/steering/reto2-hackathon.md`)

---

## Decisiones Registradas

### ADR-001: Estrategia de migración — Incremental por módulo
- **Estado:** Propuesta (pendiente validación del equipo)
- **Contexto:** El dashboard tiene 31 páginas, 31 carpetas de componentes, 23 archivos de store y 20 módulos de API. Un rewrite completo en 48h es inviable.
- **Decisión:** Migración incremental módulo a módulo. Empezar por el Shell + 1 MFE funcional (Transacciones o Dashboard Principal).
- **Justificación:** El jurado evalúa production-readiness (Funcional) y migración incremental es un beneficio explícito de la arquitectura MFE propuesta. Además, el criterio de Técnicas IA penaliza la sobre-ingeniería.
- **Criterio impactado:** Arquitectura MFE (25%) + Funcional (demo E2E sin fallos)
- **Consecuencias:** El monolito legacy coexiste con los MFEs nuevos durante la demo. Necesitamos un proxy o routing que maneje ambos.

### ADR-002: Prioridad de cierre de vulnerabilidades
- **Estado:** Propuesta (pendiente validación del equipo)
- **Contexto:** 85 issues en 13 paquetes. El criterio de vulnerabilidades pesa 30% — el más alto.
- **Decisión:** Cerrar primero nuxt (66 CVEs) y axios (3 CVEs) porque representan el 81% de los issues y son los dos únicos con severidad Critical.
- **Justificación:** Máximo impacto en el criterio de mayor peso. La migración a Nuxt 3 resuelve automáticamente nuxt + express + los 3 SVG loaders (76 issues de golpe).
- **Criterio impactado:** Vulnerabilidades cerradas (30%)
- **Consecuencias:** La migración a Nuxt 3 es prerequisito de casi todo lo demás. Debe ser la primera tarea.

### ADR-003: UI Library — Nuxt UI v3
- **Estado:** Aceptada
- **Contexto:** element-ui@2.x no soporta Vue 3. Necesitamos una alternativa compatible.
- **Decisión:** Nuxt UI v3 (basado en Radix Vue + Tailwind CSS 4).
- **Justificación:** Integración nativa con Nuxt 4, componentes accesibles out-of-the-box, Tailwind elimina la dependencia de SASS/node-sass (CVEs). Componentes compartidos en `@wompi/ui`: WDataTable, WFilterPanel, WStatusBadge, WCopyButton.
- **Alternativas descartadas:** PrimeVue (más pesado, menos integración Nuxt), Radix Vue directo (más trabajo manual).
- **Criterio impactado:** Modernización del stack (20%) + Funcional (operable sin guía)
- **Consecuencias:** No habrá pixel-perfect con el legacy. Si falta un componente de Element UI, se construye básico con Tailwind.

### ADR-004: HTTP Client — ofetch (reemplaza axios)
- **Estado:** Aceptada
- **Contexto:** axios 0.x tiene 3 CVEs (1 Critical + 2 High). Necesitamos reemplazo seguro.
- **Decisión:** `ofetch` (nativo de Nuxt 4), 0 CVEs. Interceptors migrados a `onRequest`/`onResponseError`.
- **Justificación:** Viene incluido en Nuxt 4, API más limpia, tipado nativo, elimina 3 CVEs de golpe. La lógica de interceptors (refresh token, Bearer header, User-Principal-Id, 401→logout) se preserva 1:1.
- **Alternativas descartadas:** axios@1.15.0 (funciona pero agrega dependencia innecesaria cuando ofetch ya está incluido).
- **Criterio impactado:** Vulnerabilidades cerradas (30%) + Modernización (20%)
- **Consecuencias:** Cada MFE usa `useApiClient()` de `@wompi/api-client` que conecta auth con ofetch.

### ADR-005: Framework — Nuxt 4 (no Nuxt 3, no Next.js)
- **Estado:** Aceptada
- **Contexto:** El legacy es Nuxt 1/Vue 2. Necesitamos framework moderno con 0 CVEs.
- **Decisión:** Nuxt 4 con estructura `app/`, Vue 3.5, Vite 6, SPA mode (`ssr: false`).
- **Justificación:** Mismo ecosistema Vue → curva de aprendizaje mínima. Nuxt 4 trae mejor TypeScript, estructura `app/` con separación app/server/shared, y route groups nativos. Next.js requeriría reescribir todo de Vue a React — inviable en 48h.
- **Criterio impactado:** Vulnerabilidades (30%) + Modernización (20%) + Arquitectura MFE (25%)
- **Consecuencias:** Toda la estructura usa `app/` directory (no `pages/` en raíz como Nuxt 2/3).

### ADR-006: Comunicación Cross-MFE — Event Bus tipado (mitt)
- **Estado:** Aceptada
- **Contexto:** Los MFEs necesitan comunicarse (cambio de merchant, cambio de ambiente, logout).
- **Decisión:** `@wompi/event-bus` basado en `mitt` con tipos estrictos (`EventMap`). Singleton compartido via Module Federation.
- **Justificación:** Ligero (~200 bytes), tipado completo, cleanup automático por mfeId al desmontar. Sin dependencias pesadas como Redux o Vuex global.
- **Criterio impactado:** Arquitectura MFE (25%)
- **Consecuencias:** Cada MFE se registra con `useEventBus('mfe-name')` y limpia en `onUnmounted`.

### ADR-007: Monorepo — Turborepo + pnpm workspaces
- **Estado:** Aceptada
- **Contexto:** Necesitamos gestionar 4 apps + 6 packages con builds paralelos.
- **Decisión:** Turborepo para orquestación de tasks + pnpm workspaces para resolución de dependencias.
- **Justificación:** Build caching reduce tiempos, paralelismo nativo, resolución de deps interna entre packages. pnpm es más rápido y estricto que npm/yarn.
- **Criterio impactado:** Arquitectura MFE (25%) + Modernización (20%)
- **Consecuencias:** `pnpm-workspace.yaml` define `apps/*` y `packages/*`. Scripts globales via `turbo run`.

---

## Plantilla para nuevas decisiones

```
### ADR-XXX: [Título corto]
- **Estado:** Propuesta | Aceptada | Rechazada | Reemplazada por ADR-YYY
- **Contexto:** [Qué problema o necesidad motiva esta decisión]
- **Decisión:** [Qué se decidió hacer]
- **Justificación:** [Por qué esta opción y no las alternativas]
- **Criterio impactado:** [Qué criterio de evaluación del hackathon impacta y con qué peso]
- **Consecuencias:** [Qué implica esta decisión — positivo y negativo]
```
