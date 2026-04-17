# Skill: Crear Package Compartido

## Descripción
Genera un nuevo package compartido para el monorepo con la estructura correcta de TypeScript y exports.

## Cuándo usarlo
- "Crea el package de auth"
- "Inicializa @wompi/api-client"
- "Crea el package de event-bus"

## Instrucciones

### Entrada
Nombre del package y su propósito.

### Proceso

1. **Crear estructura**:
```
packages/{nombre}/
├── src/
│   └── index.ts
├── package.json
└── tsconfig.json
```

2. **Crear `package.json`**:
```json
{
  "name": "@wompi/{nombre}",
  "version": "0.0.1",
  "private": true,
  "main": "./src/index.ts",
  "types": "./src/index.ts",
  "scripts": {
    "build": "tsc",
    "test": "vitest run",
    "clean": "rm -rf dist"
  },
  "devDependencies": {
    "typescript": "^5.8.0"
  }
}
```

3. **Crear `tsconfig.json`**:
```json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src"]
}
```

4. **Crear `src/index.ts`** con exports del package

### Reglas
- Nombre del package: `@wompi/{nombre}`
- Siempre TypeScript
- Exports explícitos en index.ts
- Si depende de otro package interno, usar `"@wompi/types": "workspace:*"`
- NO incluir dependencias de Vue/Nuxt a menos que sea necesario (mantener packages framework-agnostic cuando sea posible)
