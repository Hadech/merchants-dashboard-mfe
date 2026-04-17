# Skill: Debug Rápido en Hackathon

## Descripción
Estrategia de debugging optimizada para hackathon. Encontrar y arreglar bugs lo más rápido posible sin rabbit holes.

## Cuándo usarlo
Cuando el usuario diga cosas como:
- "No funciona"
- "Hay un error"
- "Se crashea"
- "No carga"
- "Ayuda con este bug"

## Instrucciones

### Regla de los 10 minutos
Si un bug lleva más de 10 minutos sin resolverse:
1. ¿Se puede evitar el bug con un workaround? → Hacerlo y seguir
2. ¿La feature es crítica para la demo? → Si no, comentar el código y seguir
3. ¿Es un problema de configuración? → Buscar en Stack Overflow/docs rápido

### Proceso de debug (máximo 10 min)
1. **Leer el error** completo (no asumir)
2. **Reproducir** el bug de forma consistente
3. **Aislar**: ¿Es frontend, backend, o datos?
4. **Verificar lo obvio**:
   - ¿El servidor está corriendo?
   - ¿Las variables de entorno están configuradas?
   - ¿La URL/puerto es correcto?
   - ¿Hay typos en nombres de variables/funciones?
   - ¿Se guardaron los archivos?
5. **Arreglar** con la solución más simple posible
6. **Verificar** que el fix no rompe otra cosa

### Errores comunes en hackathon
- CORS → Agregar middleware de CORS o usar proxy
- Puerto en uso → Cambiar puerto o matar proceso
- Módulo no encontrado → `npm install` / verificar imports
- Tipo undefined → Agregar optional chaining (`?.`)
- API no responde → Verificar URL, usar mock data como fallback
- Build falla → Limpiar cache (`rm -rf node_modules && npm install`)

### Herramientas de debug rápido
- `console.log` estratégico (no vergüenza, es hackathon)
- Network tab del browser para problemas de API
- React DevTools / Vue DevTools para estado
- `JSON.stringify(data, null, 2)` para inspeccionar objetos

### Después del fix
- Commit inmediato: `[fix] {qué se arregló}`
- NO refactorizar el fix, si funciona, se queda
