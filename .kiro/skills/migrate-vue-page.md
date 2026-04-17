# Skill: Migrar Página Vue 2 a Nuxt 4

## Descripción
Migra una página Vue 2 (Options API + Element UI + Pug templates) del legacy a Nuxt 4 (Composition API + Nuxt UI + TypeScript).

## Cuándo usarlo
- "Migra la página de transacciones"
- "Convierte esta página Vue 2 a Nuxt 4"
- "Crea la página de payouts basada en el legacy"

## Instrucciones

### Entrada
El usuario proporciona un archivo de `merchants-dashboard/pages/{ruta}/index.vue` del legacy.

### Proceso

1. **Leer la página completa** (template, script, style)
2. **Identificar**:
   - Qué store modules usa (`mapState`, `mapActions`, `mapSagas`)
   - Qué componentes Element UI usa
   - Qué datos muestra y qué acciones permite
   - Qué middleware o guards tiene

3. **Convertir el script**:
   - Options API → `<script setup lang="ts">`
   - `data()` → `ref()`
   - `computed` → `computed()`
   - `methods` → funciones
   - `mounted/created` → `onMounted()`
   - `watch` → `watch()`
   - `this.$store` → `const store = useXxxStore()`
   - `this.$router.push` → `navigateTo()`
   - `this.$t('key')` → `const { t } = useI18n()`

4. **Convertir el template**:
   - Pug → HTML (si usa Pug)
   - `<el-table>` → `<UTable>` con columns adaptadas
   - `<el-button>` → `<UButton>`
   - `<el-input>` → `<UInput>`
   - `<el-select>` → `<USelectMenu>`
   - `<el-dialog>` → `<UModal>`
   - `<el-form>` → `<UForm>` con schema Zod
   - `<el-pagination>` → integrada en `<UTable>`
   - Filtros `{{ val | filter }}` → `{{ formatFn(val) }}`

5. **Convertir estilos**:
   - SASS/SCSS → Tailwind CSS classes
   - Eliminar estilos de Element UI
   - Usar clases de Tailwind directamente en el template

6. **Ubicar el archivo** en la estructura Nuxt 4:
   - `apps/mfe-{domain}/app/pages/{ruta}.vue`

### Reglas
- SIEMPRE usar `<script setup lang="ts">`
- SIEMPRE tipar props con `defineProps<T>()`
- NUNCA usar Options API
- NUNCA importar Element UI
- Usar auto-imports de Nuxt 4 (ref, computed, navigateTo, etc.)
- Usar `useApiClient()` de `@wompi/api-client` para llamadas HTTP
