---
inclusion: always
---

# Hackathon - Reglas Generales

## Contexto
Estamos en una hackathon. El tiempo es limitado, la velocidad es prioridad, pero sin sacrificar calidad mínima viable.

## Principios de desarrollo

1. **Velocidad sobre perfección**: MVP funcional primero, pulir después. No sobre-engineerear.
2. **Código que funcione**: Priorizar features que se puedan demostrar en vivo.
3. **Commits frecuentes**: Commits pequeños y descriptivos. Nunca perder trabajo.
4. **Demo-driven development**: Cada feature debe ser demostrable visualmente o con output claro.
5. **Documentación mínima**: Solo README con setup y descripción del proyecto. Nada más.

## Stack y decisiones técnicas

- Elegir tecnologías que el equipo ya conozca. No es momento de aprender algo nuevo.
- Preferir librerías con buena documentación y ejemplos.
- Usar templates/boilerplates cuando sea posible.
- Evitar configuraciones complejas de infraestructura.

## Estructura de trabajo

- Dividir el proyecto en features independientes que se puedan desarrollar en paralelo.
- Cada feature debe tener un criterio claro de "terminado".
- Si algo toma más de 30 minutos sin avance, pivotar o pedir ayuda.

## Prioridades de código

1. Que funcione (funcionalidad)
2. Que se vea bien (UI/UX básica)
3. Que sea mantenible (refactor si hay tiempo)
4. Que tenga tests (solo si sobra tiempo)

## Anti-patrones en hackathon

- NO hacer setup de CI/CD elaborado
- NO escribir tests unitarios extensivos (solo los críticos)
- NO optimizar prematuramente
- NO discutir arquitectura más de 15 minutos
- NO refactorizar código que ya funciona (a menos que bloquee otra feature)
