# Skill: Scaffold de Proyecto Hackathon

## Descripción
Genera la estructura inicial de un proyecto para hackathon de forma rápida. Crea el boilerplate mínimo necesario para empezar a desarrollar features inmediatamente.

## Cuándo usarlo
Cuando el usuario diga cosas como:
- "Inicia el proyecto"
- "Crea el scaffold"
- "Setup inicial"
- "Arranca el proyecto con [tecnología]"

## Instrucciones

1. Preguntar al usuario:
   - ¿Qué tipo de proyecto? (web app, API, mobile, fullstack, CLI)
   - ¿Qué stack? (React, Next.js, Express, FastAPI, etc.)
   - ¿Nombre del proyecto?

2. Crear la estructura mínima:
   - `README.md` con: nombre, descripción de una línea, cómo correr el proyecto
   - Archivos de configuración del framework elegido
   - Una página/endpoint "Hello World" funcional
   - `.gitignore` apropiado
   - `package.json` o equivalente con scripts básicos (dev, build, start)

3. NO incluir:
   - Tests
   - CI/CD
   - Docker (a menos que sea necesario para el stack)
   - Linters/formatters elaborados
   - Documentación extensa

4. Después del scaffold, verificar que el proyecto arranca correctamente.

5. Sugerir el primer commit: `[setup] init proyecto con {stack}`

## Ejemplo de estructura para React + Vite
```
├── README.md
├── package.json
├── vite.config.ts
├── index.html
├── src/
│   ├── main.tsx
│   ├── App.tsx
│   └── App.css
└── .gitignore
```

## Ejemplo de estructura para API con Express
```
├── README.md
├── package.json
├── src/
│   ├── index.ts
│   └── routes/
│       └── health.ts
└── .gitignore
```
