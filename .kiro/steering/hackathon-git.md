---
inclusion: always
---

# Hackathon - Convenciones Git

## Commits

Formato: `[tipo] descripción corta`

Tipos:
- `[feat]` nueva funcionalidad
- `[fix]` corrección de bug
- `[setup]` configuración inicial o de herramientas
- `[ui]` cambios visuales
- `[docs]` documentación
- `[hack]` solución rápida/temporal (marcar para revisar después)

Ejemplos:
```
[setup] init proyecto con Vite + React
[feat] agregar pantalla de login
[ui] mejorar layout del dashboard
[hack] hardcodear API key temporalmente
[fix] corregir crash al cargar datos vacíos
```

## Branches

- `main` → siempre funcional, siempre desplegable
- `feat/nombre-corto` → para features en paralelo
- Merge directo a main si trabajas solo. PRs rápidos si son equipo.

## Reglas

- Commit cada vez que algo funcione, por pequeño que sea
- Push frecuente para no perder trabajo
- No hacer squash en hackathon, el historial ayuda a debuggear
