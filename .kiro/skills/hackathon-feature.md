# Skill: Desarrollo Rápido de Features

## Descripción
Guía para implementar features de forma rápida y demo-friendly durante una hackathon. Prioriza funcionalidad visible sobre arquitectura perfecta.

## Cuándo usarlo
Cuando el usuario diga cosas como:
- "Agrega [funcionalidad]"
- "Necesito que haga [algo]"
- "Implementa [feature]"
- "Crea la pantalla de [algo]"

## Instrucciones

### Antes de codear (máximo 2 minutos de análisis)
1. Identificar el resultado visible/demostrable de la feature
2. Definir el flujo mínimo (happy path solamente)
3. Identificar si necesita datos mock o API real

### Durante el desarrollo
1. Implementar el happy path completo primero
2. Usar datos hardcodeados/mock si la API no está lista
3. Agregar UI básica pero presentable (usar componentes de librería si hay)
4. NO manejar todos los edge cases, solo los que crashean la app
5. Agregar un `console.log` o indicador visual de que la feature funciona

### Después de implementar
1. Verificar que funciona de inicio a fin
2. Verificar que no rompe features existentes
3. Sugerir commit con formato: `[feat] {descripción corta}`

### Manejo de errores (mínimo viable)
- Try/catch en llamadas a APIs
- Mostrar "Algo salió mal" genérico en UI si falla
- NO crear sistema de error handling elaborado
- `console.error` para debugging rápido

### Si la feature se complica
- Si lleva más de 30 minutos sin avance → sugerir simplificar el scope
- Si depende de otra feature no terminada → usar mocks y seguir
- Si hay un bug bloqueante → arreglar el bug primero, luego continuar
