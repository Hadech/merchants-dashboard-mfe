# Documento de Requerimientos — Migración Merchants Dashboard a Micro-Frontends

## Introducción

Migración del Merchants Dashboard de Wompi desde un monolito SPA legacy (Nuxt 1.0.0, Vue 2.7, Webpack 3, Node 12, sin TypeScript, 60+ vulnerabilidades ignoradas en Snyk) hacia una arquitectura de micro-frontends moderna con Nuxt 4, Vue 3, TypeScript, Module Federation y Turborepo. El objetivo es eliminar la deuda técnica acumulada, resolver todas las vulnerabilidades de seguridad, y habilitar el despliegue independiente por dominio funcional.

## Glosario

- **Shell**: Aplicación Nuxt 4 host que provee layout global (sidebar, header, content area), autenticación, navegación, selector de merchant y configuración de Module Federation como host. Usa la nueva estructura `app/` de Nuxt 4.
- **MFE**: Micro-Frontend — aplicación Nuxt 4 independiente que se registra como remote en Module Federation y se carga dinámicamente dentro del Shell.
- **MFE_Transactions**: Micro-frontend que encapsula el dominio de transacciones, disputas y payment links.
- **MFE_Payouts**: Micro-frontend que encapsula el dominio de payouts (balances, pagos, aprobaciones, favoritos, límites, reportes).
- **MFE_Settings**: Micro-frontend que encapsula el dominio de configuración (usuarios, roles, llaves API, mi cuenta, developers).
- **Monorepo**: Repositorio único gestionado con Turborepo que contiene todas las apps (Shell, MFEs) y packages compartidos.
- **Module_Federation**: Plugin de Vite (`@module-federation/vite`) que permite cargar módulos remotos en runtime, habilitando despliegue independiente de cada MFE. Compatible con Nuxt 4 y su bundler Vite 6.
- **Pinia**: Librería de manejo de estado para Vue 3 que reemplaza a Vuex, con API basada en Composition API.
- **Cognito**: Servicio AWS de autenticación usado para login, logout, refresh de sesión y gestión de tokens.
- **API_Client**: Package compartido que encapsula la comunicación HTTP con los backends de Wompi, incluyendo interceptors de autenticación y manejo de errores.
- **Nuxt_UI**: Librería de componentes UI basada en Radix Vue + Tailwind CSS, usada como reemplazo de Element UI.
- **Event_Bus**: Mecanismo de comunicación tipado entre MFEs para eventos cross-domain (merchant seleccionado, estado de auth, cambio de ambiente).
- **Sandbox**: Ambiente de pruebas de Wompi con API y datos separados del ambiente de producción.

## Requerimientos

### Requerimiento 1: Estructura del Monorepo

**User Story:** Como desarrollador del equipo, quiero un monorepo con Turborepo que organice las apps y packages compartidos, para que cada MFE pueda desarrollarse, probarse y desplegarse de forma independiente.

#### Criterios de Aceptación

1. THE Monorepo SHALL organizar el código en dos directorios raíz: `apps/` para las aplicaciones (Shell, MFE_Transactions, MFE_Payouts, MFE_Settings) y `packages/` para los módulos compartidos (ui, auth, api-client, i18n, types).
2. THE Monorepo SHALL incluir un archivo `turbo.json` que defina los pipelines de `build`, `dev`, `lint` y `test` con las dependencias correctas entre tasks.
3. THE Monorepo SHALL incluir un `tsconfig.base.json` en la raíz que extienda cada app y package para garantizar configuración TypeScript consistente.
4. WHEN se ejecuta `turbo run build`, THE Monorepo SHALL compilar los packages compartidos antes que las apps, respetando el grafo de dependencias.
5. THE Monorepo SHALL usar workspaces de npm o pnpm para resolver dependencias internas entre apps y packages sin publicar a un registry.

---

### Requerimiento 2: Shell App — Layout y Navegación

**User Story:** Como comerciante de Wompi, quiero acceder al dashboard con un layout consistente (sidebar, header, área de contenido), para que pueda navegar entre los diferentes módulos del sistema.

#### Criterios de Aceptación

1. THE Shell SHALL renderizar un layout con tres zonas: sidebar de navegación lateral, header superior con información del usuario y merchant activo, y área de contenido principal donde se montan los MFEs.
2. THE Shell SHALL mostrar en el sidebar los enlaces de navegación agrupados por dominio: Transacciones (transacciones, disputas, payment links), Payouts (balances, crear pago, transacciones payouts, aprobaciones, favoritos, límites, reportes) y Configuración (usuarios, roles, llaves API, mi cuenta, developers).
3. WHEN el usuario hace clic en un enlace del sidebar, THE Shell SHALL cargar el MFE correspondiente en el área de contenido sin recargar la página completa.
4. THE Shell SHALL mostrar en el header el nombre del comerciante activo, el ambiente actual (sandbox o producción) y el menú desplegable del usuario.
5. WHILE el usuario no tiene sesión activa, THE Shell SHALL redirigir todas las rutas protegidas a la página de login.

---

### Requerimiento 3: Shell App — Autenticación con Cognito

**User Story:** Como comerciante, quiero autenticarme con mis credenciales de Cognito existentes, para que pueda acceder al dashboard de forma segura sin necesidad de crear una cuenta nueva.

#### Criterios de Aceptación

1. WHEN el usuario envía credenciales válidas en el formulario de login, THE Shell SHALL autenticar al usuario contra AWS Cognito usando la librería `amazon-cognito-identity-js` y almacenar los tokens de sesión (idToken, accessToken, refreshToken).
2. WHEN un token de acceso está próximo a expirar, THE Shell SHALL ejecutar un refresh de sesión automático usando el refreshToken de Cognito antes de cada request HTTP.
3. WHEN el refresh de sesión falla dos veces consecutivas, THE Shell SHALL cerrar la sesión del usuario, limpiar los tokens almacenados y redirigir a la página de login.
4. WHEN el usuario hace clic en "Cerrar sesión", THE Shell SHALL invalidar la sesión local, limpiar todos los tokens almacenados y redirigir a la página de login.
5. THE Shell SHALL exponer el estado de autenticación (usuario, tokens, isAuthenticated) a todos los MFEs a través del package compartido `auth`.
6. IF el backend responde con código HTTP 401, THEN THE Shell SHALL cerrar la sesión del usuario y redirigir a la página de login.

---

### Requerimiento 4: Shell App — Selector de Merchant y Ambiente

**User Story:** Como comerciante con múltiples comercios, quiero seleccionar el comercio activo y alternar entre sandbox y producción, para que pueda gestionar cada comercio y ambiente de forma independiente.

#### Criterios de Aceptación

1. THE Shell SHALL mostrar un selector de merchant que permita al usuario cambiar entre los comercios asociados a su cuenta.
2. WHEN el usuario selecciona un merchant diferente, THE Shell SHALL actualizar el header `User-Principal-Id` en todas las peticiones HTTP subsiguientes y notificar a los MFEs del cambio mediante el Event_Bus.
3. THE Shell SHALL permitir al usuario alternar entre ambiente sandbox y producción, cambiando la base URL del API_Client entre `API_GW_BASE_URL` y `API_GW_BASE_URL_SANDBOX`.
4. WHEN el usuario cambia de ambiente, THE Shell SHALL notificar a todos los MFEs del cambio mediante el Event_Bus para que recarguen sus datos con el nuevo contexto.

---

### Requerimiento 5: Module Federation — Configuración Host/Remote

**User Story:** Como líder técnico, quiero que el Shell cargue los MFEs como módulos remotos via Module Federation, para que cada MFE pueda desplegarse de forma independiente sin requerir rebuild del Shell.

#### Criterios de Aceptación

1. THE Shell SHALL configurar `@module-federation/vite` como host, declarando los remotes: MFE_Transactions, MFE_Payouts y MFE_Settings con sus URLs de carga.
2. WHEN el usuario navega a una ruta de un MFE, THE Shell SHALL cargar dinámicamente el módulo remoto correspondiente y montarlo en el área de contenido.
3. THE Module_Federation SHALL declarar Vue 3, Pinia, vue-i18n y ofetch como dependencias compartidas en modo singleton para evitar duplicación en runtime.
4. IF un MFE remoto no está disponible al momento de carga, THEN THE Shell SHALL mostrar un mensaje de error descriptivo al usuario en lugar de una pantalla en blanco.
5. WHEN se despliega una nueva versión de un MFE, THE Shell SHALL cargar la versión actualizada sin necesidad de redesplegar el Shell.

---

### Requerimiento 6: MFE Transactions — Lista y Detalle de Transacciones

**User Story:** Como comerciante, quiero ver la lista de transacciones con filtros y acceder al detalle de cada transacción, para que pueda monitorear los pagos recibidos.

#### Criterios de Aceptación

1. THE MFE_Transactions SHALL mostrar una tabla paginada de transacciones con las columnas: ID, referencia, monto, método de pago, estado, canal de origen y fecha.
2. THE MFE_Transactions SHALL proveer filtros por: ID de transacción, referencia, email del cliente, estado, tipo de método de pago y canal de origen.
3. WHEN el usuario aplica filtros, THE MFE_Transactions SHALL consultar el API con los filtros activos y actualizar la tabla de resultados.
4. WHEN el usuario hace clic en una transacción de la lista, THE MFE_Transactions SHALL navegar a la vista de detalle mostrando toda la información de la transacción incluyendo método de pago, timeline de estados y datos del cliente.
5. THE MFE_Transactions SHALL permitir la descarga de reportes de transacciones en formato CSV o Excel según los filtros aplicados.

---

### Requerimiento 7: MFE Transactions — Disputas

**User Story:** Como comerciante, quiero gestionar las disputas de transacciones, para que pueda responder a contracargos y reclamaciones.

#### Criterios de Aceptación

1. THE MFE_Transactions SHALL mostrar una lista paginada de disputas con estado, monto, fecha de creación y transacción asociada.
2. WHEN el usuario hace clic en una disputa, THE MFE_Transactions SHALL navegar a la vista de detalle mostrando la información completa de la disputa, la transacción asociada y el timeline de estados.

---

### Requerimiento 8: MFE Transactions — Payment Links

**User Story:** Como comerciante, quiero crear y gestionar links de pago, para que pueda compartir enlaces de cobro con mis clientes.

#### Criterios de Aceptación

1. THE MFE_Transactions SHALL mostrar una lista paginada de payment links con nombre, monto, estado y fecha de creación.
2. WHEN el usuario hace clic en "Crear link de pago", THE MFE_Transactions SHALL mostrar un formulario para configurar el link con: nombre, descripción, monto, moneda y fecha de expiración.
3. WHEN el usuario envía el formulario de creación con datos válidos, THE MFE_Transactions SHALL crear el payment link via API y mostrar el link generado con opción de copiar al portapapeles.
4. WHEN el usuario hace clic en un payment link de la lista, THE MFE_Transactions SHALL navegar a la vista de detalle mostrando la configuración del link, su URL y las transacciones asociadas.

---

### Requerimiento 9: MFE Payouts — Balances y Dashboard

**User Story:** Como comerciante, quiero ver el balance disponible de mi cuenta, para que pueda conocer los fondos disponibles para dispersiones.

#### Criterios de Aceptación

1. THE MFE_Payouts SHALL mostrar un dashboard con el balance disponible del merchant activo, consultando el endpoint `/transversal/balances/{merchantId}`.
2. THE MFE_Payouts SHALL mostrar el estado del auto-pago (activado/desactivado) y permitir al usuario cambiar su estado.
3. WHEN el usuario activa o desactiva el auto-pago, THE MFE_Payouts SHALL actualizar el estado via API y reflejar el cambio en la interfaz.

---

### Requerimiento 10: MFE Payouts — Crear Pago / Dispersión

**User Story:** Como comerciante, quiero crear pagos y dispersiones a terceros, para que pueda transferir fondos desde mi balance.

#### Criterios de Aceptación

1. THE MFE_Payouts SHALL proveer un formulario de creación de pago con: destinatario (cuenta bancaria o favorito), monto, concepto y tipo de pago.
2. WHEN el usuario envía el formulario con datos válidos, THE MFE_Payouts SHALL crear el pago via API y mostrar la confirmación con el ID de la transacción.
3. WHEN el pago requiere aprobación según las reglas del merchant, THE MFE_Payouts SHALL notificar al usuario que el pago queda pendiente de aprobación.

---

### Requerimiento 11: MFE Payouts — Transacciones, Aprobaciones y Favoritos

**User Story:** Como comerciante, quiero consultar las transacciones de payouts, aprobar pagos pendientes y gestionar destinatarios favoritos, para que pueda administrar mis dispersiones eficientemente.

#### Criterios de Aceptación

1. THE MFE_Payouts SHALL mostrar una lista paginada de transacciones de payouts con filtros por estado, fecha y monto.
2. THE MFE_Payouts SHALL mostrar una vista de aprobaciones con los pagos pendientes de aprobación, permitiendo aprobar o rechazar cada uno.
3. THE MFE_Payouts SHALL mostrar una lista de destinatarios favoritos con opción de agregar, editar y eliminar favoritos.
4. THE MFE_Payouts SHALL mostrar la configuración de límites de dispersión del merchant.
5. THE MFE_Payouts SHALL permitir la descarga de reportes de payouts según los filtros aplicados.

---

### Requerimiento 12: MFE Settings — Gestión de Usuarios

**User Story:** Como administrador del comercio, quiero gestionar los usuarios del dashboard (crear, ver, editar, deshabilitar), para que pueda controlar quién tiene acceso al sistema.

#### Criterios de Aceptación

1. THE MFE_Settings SHALL mostrar una lista paginada de usuarios del merchant con nombre, email, rol y estado.
2. WHEN el administrador hace clic en "Crear usuario", THE MFE_Settings SHALL mostrar un formulario con: nombre, email, teléfono y rol asignado.
3. WHEN el administrador envía el formulario de creación con datos válidos, THE MFE_Settings SHALL crear el usuario via API y actualizar la lista.
4. WHEN el administrador hace clic en un usuario de la lista, THE MFE_Settings SHALL navegar al detalle del usuario con opción de editar datos o deshabilitar la cuenta.
5. WHEN el administrador deshabilita un usuario, THE MFE_Settings SHALL actualizar el estado del usuario via API y reflejar el cambio en la lista.

---

### Requerimiento 13: MFE Settings — Gestión de Roles

**User Story:** Como administrador del comercio, quiero gestionar los roles y permisos del dashboard, para que pueda definir qué acciones puede realizar cada tipo de usuario.

#### Criterios de Aceptación

1. THE MFE_Settings SHALL mostrar una lista de roles del merchant con nombre y cantidad de usuarios asignados.
2. WHEN el administrador hace clic en "Crear rol", THE MFE_Settings SHALL mostrar un formulario con nombre del rol y selector de permisos agrupados por módulo.
3. WHEN el administrador envía el formulario de creación con datos válidos, THE MFE_Settings SHALL crear el rol via API y actualizar la lista.
4. WHEN el administrador hace clic en un rol existente, THE MFE_Settings SHALL navegar a la vista de edición con los permisos actuales seleccionados.

---

### Requerimiento 14: MFE Settings — Llaves API, Mi Cuenta y Developers

**User Story:** Como comerciante o desarrollador, quiero acceder a las llaves de API, gestionar mi cuenta y ver la documentación de integración, para que pueda integrar Wompi en mis aplicaciones.

#### Criterios de Aceptación

1. THE MFE_Settings SHALL mostrar las llaves de API del merchant (pública y privada) con opción de copiar al portapapeles, diferenciando entre llaves de sandbox y producción.
2. THE MFE_Settings SHALL proveer una vista de "Mi cuenta" donde el usuario pueda actualizar su contraseña, email y teléfono.
3. THE MFE_Settings SHALL proveer una sección de "Developers" con información de integración y configuración de webhooks.

---

### Requerimiento 15: Package Compartido — API Client

**User Story:** Como desarrollador, quiero un API client compartido y tipado que encapsule la autenticación y el manejo de errores, para que todos los MFEs consuman los APIs de Wompi de forma consistente.

#### Criterios de Aceptación

1. THE API_Client SHALL proveer una función factory que cree instancias HTTP configuradas con la base URL del ambiente activo (sandbox o producción) y el prefijo `/dashboard`.
2. THE API_Client SHALL incluir un interceptor de request que ejecute refresh de sesión Cognito antes de cada petición y adjunte los headers `Authorization: Bearer {token}` y `User-Principal-Id: {merchantUserId}`.
3. IF una petición HTTP falla con código 401, THEN THE API_Client SHALL invocar el flujo de cierre de sesión del package `auth`.
4. THE API_Client SHALL exponer tipos TypeScript para todas las respuestas de API usadas por los MFEs.
5. THE API_Client SHALL soportar timeout configurable con un valor por defecto de 60 segundos, consistente con el cliente legacy.

---

### Requerimiento 16: Package Compartido — UI Components

**User Story:** Como desarrollador, quiero un package de componentes UI compartidos basado en Nuxt UI + Tailwind, para que todos los MFEs tengan una apariencia visual consistente.

#### Criterios de Aceptación

1. THE Package_UI SHALL proveer componentes reutilizables basados en Nuxt UI para: tablas con paginación, formularios con validación, modales, alertas, date pickers, selectores y botones.
2. THE Package_UI SHALL configurar Tailwind CSS con los colores, tipografía y espaciado del design system de Wompi.
3. THE Package_UI SHALL exportar los componentes como módulo consumible por cada MFE y el Shell sin duplicación de estilos.

---

### Requerimiento 17: Package Compartido — i18n

**User Story:** Como comerciante en Colombia, Panamá o Guatemala, quiero ver el dashboard en mi idioma y formato regional, para que la experiencia sea localizada.

#### Criterios de Aceptación

1. THE Package_i18n SHALL proveer archivos de traducción para los locales `es_CO`, `es_PA` y `es_GT`, migrando las traducciones existentes del legacy.
2. THE Package_i18n SHALL configurar `@nuxtjs/i18n` (vue-i18n 9) con detección automática del locale basada en la variable de entorno `I18N_LOCALE`.
3. THE Package_i18n SHALL exponer un composable `useI18n()` que cada MFE pueda usar para acceder a las traducciones sin configuración adicional.

---

### Requerimiento 18: Seguridad — Eliminación de Vulnerabilidades

**User Story:** Como equipo de ciberseguridad, quiero que el nuevo stack tenga 0 vulnerabilidades conocidas, para que el dashboard cumpla con los estándares de seguridad de Wompi.

#### Criterios de Aceptación

1. THE Monorepo SHALL usar versiones actuales y soportadas de todas las dependencias: Nuxt 4.x, Vue 3.x, TypeScript 5.x, Node 20 LTS, Vite 6.x.
2. THE Monorepo SHALL eliminar todas las dependencias legacy con vulnerabilidades conocidas: nuxt 1.x, vue 2.x, webpack 3.x, axios 0.x, element-ui 2.x, node-sass, babel 6.x.
3. WHEN se ejecuta un escaneo de Snyk sobre el Monorepo, THE Monorepo SHALL reportar 0 vulnerabilidades de severidad crítica o alta.
4. THE Monorepo SHALL no requerir un archivo `.snyk` con vulnerabilidades ignoradas.

---

### Requerimiento 19: Migración de Estado — Vuex a Pinia

**User Story:** Como desarrollador, quiero que el estado de la aplicación se gestione con Pinia en lugar de Vuex + vuex-saga, para que el código sea más simple, tipado y compatible con Vue 3.

#### Criterios de Aceptación

1. WHEN se migra un módulo Vuex, THE Pinia_Store SHALL convertir `state` a `ref()`, eliminar `mutations`, convertir `actions` generadoras (vuex-saga) a funciones `async` y convertir `getters` a `computed()`.
2. THE Pinia_Store SHALL mantener la misma estructura de datos y lógica de negocio que el módulo Vuex original para garantizar paridad funcional.
3. THE Pinia_Store SHALL tipar todos los estados, acciones y getters con interfaces TypeScript.

---

### Requerimiento 20: Comunicación Cross-MFE

**User Story:** Como desarrollador, quiero un mecanismo tipado de comunicación entre MFEs, para que los eventos globales (cambio de merchant, cambio de ambiente, estado de auth) se propaguen correctamente.

#### Criterios de Aceptación

1. THE Event_Bus SHALL proveer un sistema de publicación/suscripción tipado con TypeScript para eventos cross-MFE.
2. THE Event_Bus SHALL soportar los eventos: `merchant:changed`, `environment:changed`, `auth:logout` y `auth:session-refreshed`.
3. WHEN el Shell emite un evento `merchant:changed`, THE Event_Bus SHALL notificar a todos los MFEs suscritos con el nuevo ID de merchant.
4. WHEN un MFE se desmonta, THE Event_Bus SHALL limpiar automáticamente las suscripciones de ese MFE para evitar memory leaks.
