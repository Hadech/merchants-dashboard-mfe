# Skill: UI Polish Rápido

## Descripción
Mejora rápida de la interfaz visual para que el proyecto se vea profesional en la demo, sin invertir demasiado tiempo.

## Cuándo usarlo
Cuando el usuario diga cosas como:
- "Mejora el diseño"
- "Que se vea mejor"
- "Polish de UI"
- "Prepara para la demo"
- "Que se vea profesional"

## Instrucciones

### Prioridades de polish (en orden)
1. **Layout consistente**: Centrar contenido, espaciado uniforme, responsive básico
2. **Colores y tipografía**: Usar una paleta de 2-3 colores máximo, fuente legible
3. **Estados de carga**: Spinners o skeletons en lugar de pantallas en blanco
4. **Transiciones suaves**: Fade-in básico en cambios de vista
5. **Empty states**: Mensajes amigables cuando no hay datos

### Quick wins de alto impacto
- Agregar un header/navbar con el nombre del proyecto
- Usar bordes redondeados y sombras sutiles
- Agregar iconos (lucide-react, heroicons, o emojis como fallback)
- Gradientes sutiles en backgrounds
- Hover effects en botones y cards

### Librerías recomendadas (si no hay una ya)
- **Tailwind CSS**: Rápido para estilar sin escribir CSS
- **shadcn/ui**: Componentes bonitos y copy-paste
- **Chakra UI**: Componentes listos con buen diseño
- **DaisyUI**: Plugin de Tailwind con componentes

### NO hacer
- Animaciones complejas
- Responsive perfecto para todos los breakpoints
- Dark mode (a menos que sea trivial)
- Accesibilidad completa (solo lo básico: contraste, alt text)
- Diseño pixel-perfect

### Checklist pre-demo visual
- [ ] La app no tiene pantallas en blanco/rotas
- [ ] Los textos son legibles (tamaño y contraste)
- [ ] Los botones parecen botones (hover, cursor pointer)
- [ ] Hay feedback visual en acciones (loading, success)
- [ ] El nombre del proyecto es visible
