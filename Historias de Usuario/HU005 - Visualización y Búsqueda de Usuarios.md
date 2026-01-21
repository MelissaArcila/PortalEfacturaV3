# HU005 - Visualización y Búsqueda de Usuarios

## Información General

| Campo | Descripción |
|-------|-------------|
| **Release objetivo** | Release 1 |
| **Epic** | Gestión de Usuarios y Permisos |
| **Estado** | BORRADOR |
| **Propietario** | Por definir |
| **Arquitecto** | Por definir |
| **Desarrolladores** | Por definir |
| **Tester** | Por definir |
| **Incidentes relacionados** | N/A |
| **Arquitectura relacionada** | Portal Unificado - Módulo de Administración, Gestión de Usuarios y Permisos |

---

## Contexto

El Portal Unificado de CDN Facturación requiere que los Administradores de Portal puedan visualizar, buscar y consultar información de todos los Usuarios registrados en el sistema, tanto Usuarios Internos (personal de CDN) como Usuarios de Cliente (personal de empresas contratantes).

Esta funcionalidad es fundamental para gestionar la base de usuarios de la plataforma, identificar usuarios específicos, verificar sus permisos asignados, revisar su estado (activo, inactivo, bloqueado), y acceder a acciones de administración como edición, inactivación, o reseteo de contraseña.

La pantalla de visualización de usuarios debe proporcionar capacidades de búsqueda y filtrado eficientes para localizar usuarios en una base que puede contener miles de registros, mostrar información relevante de cada usuario de forma clara y estructurada, y permitir navegación fluida hacia otras acciones de gestión de usuarios.

---

**Notas:**
- Solo Usuarios con rol de Administrador del Portal pueden acceder a esta funcionalidad
- La pantalla muestra tanto Usuarios Internos como Usuarios de Cliente en una vista unificada
- Los permisos asignados a cada usuario deben ser visibles de forma resumida
- Se debe mostrar el estado actual de cada usuario (Activo, Inactivo, Bloqueado)
- La búsqueda y filtros deben funcionar en tiempo real sin recargar la página completa
- La información sensible (contraseñas) NUNCA se muestra en esta pantalla
- Todo el acceso a esta pantalla debe ser auditable siguiendo el Estándar de Auditoría Funcional

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Administrador del Portal** | Quiero acceder a una pantalla de visualización de todos los usuarios del sistema | Para tener una vista centralizada de la base de usuarios y poder gestionarlos eficientemente |
| Como **Administrador del Portal** | Quiero poder buscar usuarios por número de identificación, nombre, apellido o correo electrónico | Para localizar rápidamente un usuario específico sin tener que navegar por toda la lista |
| Como **Administrador del Portal** | Quiero filtrar usuarios por estado, tipo (interno/cliente), y cliente asociado aplicando los filtros manualmente | Para analizar subconjuntos específicos de usuarios de forma controlada |
| Como **Administrador del Portal** | Quiero ver en la tabla de usuarios su información básica y permisos asignados | Para tener contexto completo de cada usuario sin necesidad de abrir detalles individuales |
| Como **Administrador del Portal** | Quiero poder acceder rápidamente a acciones sobre cada usuario (editar, inactivar, resetear contraseña, ver detalle) | Para realizar gestión eficiente sin navegar por múltiples pantallas |
| Como **Auditor** | Quiero que todos los accesos a la pantalla de usuarios sean registrados | Para garantizar trazabilidad de quién consulta información de usuarios y detectar accesos no autorizados |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Acceso al módulo de Gestión de Usuarios | Administrador del Portal autenticado en el sistema | Administrador navega al menú principal | Sistema muestra opción "Gestión de Usuarios" en el menú de Administración. Solo visible para Usuarios con rol de Administrador del Portal | ☐ | ☐ | ☐ |
| 2 | Visualización de pantalla principal de usuarios | Administrador hace clic en "Gestión de Usuarios" | Sistema carga la pantalla | Sistema muestra pantalla con: Título "Gestión de Usuarios", buscador general (campo de texto), filtros (Estado, Tipo de Usuario, Cliente), botón "Crear Nuevo Usuario", tabla de usuarios con columnas: Número ID, Nombre Completo, Correo Electrónico, Tipo (Interno/Cliente), Estado, Permisos Asignados, Acciones. Paginación habilitada (20 registros por página por defecto) | ☐ | ☐ | ☐ |
| 3 | Listado inicial de usuarios con paginación | Administrador accede a pantalla de Gestión de Usuarios y existen usuarios registrados | Sistema carga datos | Sistema muestra los primeros 20 usuarios ordenados por fecha de creación descendente (más recientes primero). Muestra paginación inferior con: total de registros, número de página actual, botones "Anterior", "Siguiente", selector de registros por página (10, 20, 50, 100). Contador muestra "Mostrando 1-20 de X usuarios" | ☐ | ☐ | ☐ |
| 4 | Búsqueda de usuarios por texto general | Administrador en pantalla de usuarios, escribe en el buscador general | Administrador ingresa texto y presiona Enter o hace clic en ícono de búsqueda | Sistema busca coincidencias en: Número de Identificación, Primer Nombre, Segundo Nombre, Primer Apellido, Segundo Apellido, Correo Electrónico (búsqueda parcial, case-insensitive). Actualiza tabla mostrando solo usuarios que coinciden. Muestra contador "Mostrando X resultados para '[texto]'". Si no hay resultados, muestra mensaje: "No se encontraron usuarios que coincidan con la búsqueda" | ☐ | ☐ | ☐ |
| 5 | Selección de filtro por Estado de usuario | Administrador hace clic en filtro "Estado" | Administrador selecciona una opción del dropdown | Sistema muestra dropdown con opciones: "Todos", "Activo", "Inactivo", "Bloqueado". Administrador selecciona opción deseada. Filtro NO se aplica automáticamente hasta hacer clic en botón "Aplicar Filtros" | ☐ | ☐ | ☐ |
| 6 | Selección de filtro por Tipo de Usuario | Administrador hace clic en filtro "Tipo de Usuario" | Administrador selecciona una opción del dropdown | Sistema muestra dropdown con opciones: "Todos", "Usuario Interno", "Usuario de Cliente". Administrador selecciona opción deseada. Filtro NO se aplica automáticamente hasta hacer clic en botón "Aplicar Filtros" | ☐ | ☐ | ☐ |
| 7 | Selección de filtro por Cliente | Administrador hace clic en filtro "Cliente" | Administrador selecciona un cliente del dropdown | Sistema muestra dropdown con lista de todas las empresas/clientes registrados (ordenados alfabéticamente). Incluye opción "Todos" y "Solo Internos (Sin Cliente)". Administrador selecciona opción deseada. Filtro NO se aplica automáticamente hasta hacer clic en botón "Aplicar Filtros" | ☐ | ☐ | ☐ |
| 7a | Aplicar filtros seleccionados | Administrador ha seleccionado uno o más filtros (Estado, Tipo, Cliente) y/o ingresado texto en búsqueda | Administrador hace clic en botón "Aplicar Filtros" | Sistema aplica todos los filtros seleccionados y búsqueda. Actualiza tabla mostrando solo usuarios que coinciden con los criterios. Muestra contador actualizado "Mostrando X resultados". Filtros se pueden combinar entre sí. Muestra indicadores visuales de filtros activos | ☐ | ☐ | ☐ |
| 8 | Limpiar filtros y búsqueda | Administrador ha aplicado filtros y/o búsqueda | Administrador hace clic en botón "Limpiar Filtros" | Sistema remueve todos los filtros activos y búsqueda. Vuelve a mostrar todos los usuarios con ordenamiento predeterminado (fecha creación descendente). Resetea selectores de filtro a "Todos" y limpia campo de búsqueda | ☐ | ☐ | ☐ |
| 9 | Visualización de información básica en tabla | Administrador visualiza la tabla de usuarios | Usuario revisa información | Cada fila de la tabla muestra: **(1) Número de Identificación** (texto plano), **(2) Nombre Completo** (concatenación: Primer Nombre + Segundo Nombre + Primer Apellido + Segundo Apellido), **(3) Correo Electrónico** (siempre presente, campo obligatorio), **(4) Tipo** (badge: "Interno" en color primario o "Cliente" en color secundario), **(5) Estado** (badge: "Activo" verde, "Inactivo" gris, "Bloqueado" rojo), **(6) Permisos Asignados** (número, ej: "3 permisos" clickeable), **(7) Acciones** (iconos: Ver Detalle, Editar, Más Opciones) | ☐ | ☐ | ☐ |
| 10 | Ordenamiento por columnas | Administrador hace clic en encabezado de columna "Nombre Completo" | Sistema detecta clic | Sistema ordena la tabla alfabéticamente por Nombre Completo ascendente. Muestra indicador de ordenamiento (flecha arriba). Al hacer clic nuevamente, invierte el orden (descendente, flecha abajo). Permite ordenar por: Número ID, Nombre Completo, Estado, Fecha de Creación | ☐ | ☐ | ☐ |
| 11 | Visualización de permisos asignados resumidos | Administrador hace clic en enlace "X permisos" en columna "Permisos Asignados" | Sistema muestra popover o tooltip expandido | Sistema muestra lista resumida de permisos del usuario: Cliente (Nombre Empresa) - Rol. Ejemplo: "Empresa ABC - Administrador de Cliente", "Empresa XYZ - Gestor Emisión FE". Si usuario tiene más de 5 permisos, muestra los primeros 5 y enlace "Ver todos" que navega a pantalla de detalle. Si usuario es interno sin permisos de cliente, muestra "Solo roles internos" | ☐ | ☐ | ☐ |
| 12 | Navegación a detalle de usuario | Administrador hace clic en icono "Ver Detalle" (ojo) en columna Acciones | Sistema detecta clic | Sistema navega a pantalla de "Detalle de Usuario" mostrando información completa: Datos personales, historial de permisos (con fechas de asignación y usuario que asignó), historial de cambios de estado, último acceso al sistema, clientes asociados, auditoría de acciones del usuario | ☐ | ☐ | ☐ |
| 13 | Navegación a edición de usuario | Administrador hace clic en icono "Editar" (lápiz) en columna Acciones | Sistema detecta clic | Sistema navega a pantalla de "Editar Usuario" pre-poblada con datos actuales del usuario (similar a creación pero en modo edición). Permite modificar: nombres, apellidos, correo, estado, permisos | ☐ | ☐ | ☐ |
| 14 | Menú de más opciones por usuario | Administrador hace clic en icono "Más Opciones" (tres puntos verticales) en columna Acciones | Sistema muestra menú contextual | Sistema muestra menú desplegable con opciones: "Resetear Contraseña", "Inactivar Usuario" (si está activo) o "Activar Usuario" (si está inactivo). Cada opción requiere confirmación | ☐ | ☐ | ☐ |
| 15 | Cambio de cantidad de registros por página | Administrador hace clic en selector de registros por página | Administrador selecciona "50" | Sistema recarga la tabla mostrando hasta 50 usuarios por página. Actualiza paginación y contador "Mostrando 1-50 de X usuarios". Mantiene filtros y búsqueda activos | ☐ | ☐ | ☐ |
| 16 | Navegación entre páginas | Administrador en página 1 hace clic en botón "Siguiente" | Sistema detecta clic | Sistema carga la página 2 mostrando los siguientes registros. Actualiza contador "Mostrando 21-40 de X usuarios". Botón "Anterior" se habilita. Si es la última página, botón "Siguiente" se deshabilita | ☐ | ☐ | ☐ |
| 17 | Indicador visual de usuario bloqueado | Administrador visualiza tabla con usuario bloqueado | Sistema muestra datos | En columna "Estado", muestra badge "Bloqueado" en color rojo. Opcionalmente muestra tooltip con razón del bloqueo: "Bloqueado por X intentos fallidos. Desbloqueo automático: [fecha/hora]" o "Bloqueado manualmente por [admin] el [fecha]" | ☐ | ☐ | ☐ |
| 18 | Formato de visualización de correo electrónico | Administrador visualiza tabla de usuarios | Sistema muestra datos | En columna "Correo Electrónico", muestra el correo electrónico del usuario en texto plano. Todos los usuarios tienen correo electrónico ya que es campo obligatorio en la creación (HU006) | ☐ | ☐ | ☐ |
| 19 | Botón "Crear Nuevo Usuario" visible y accesible | Administrador en pantalla de Gestión de Usuarios | Administrador visualiza interfaz | Sistema muestra botón "Crear Nuevo Usuario" prominente en esquina superior derecha de la pantalla (estilo: contained, gradiente Primary → Secondary, ícono de +). Al hacer clic, navega a pantalla de creación de usuario (HU006) | ☐ | ☐ | ☐ |
| 20 | Sistema sin usuarios registrados | Administrador accede a pantalla de Gestión de Usuarios en sistema nuevo sin usuarios | Sistema carga datos | Sistema muestra mensaje amigable: "Aún no hay usuarios registrados en el sistema. Crea el primer usuario haciendo clic en 'Crear Nuevo Usuario'." con ilustración o ícono. Botón "Crear Nuevo Usuario" permanece visible y destacado | ☐ | ☐ | ☐ |
| 21 | Búsqueda sin resultados | Administrador busca "XYZ123" pero ningún usuario coincide | Sistema procesa búsqueda | Sistema muestra mensaje: "No se encontraron usuarios que coincidan con la búsqueda 'XYZ123'. Verifica el término de búsqueda o limpia los filtros." con ícono de búsqueda vacía. Botón "Limpiar Filtros" visible | ☐ | ☐ | ☐ |
| 22 | Validación de permiso de acceso | Usuario sin rol de Administrador del Portal intenta acceder mediante URL directa | Usuario no autorizado navega a /admin/usuarios | Sistema valida permisos. Muestra mensaje: "No tiene permisos para acceder a esta sección. Solo Administradores del Portal pueden gestionar usuarios." y redirige a pantalla de inicio del Portal | ☐ | ☐ | ☐ |
| 23 | Refresh manual de la lista | Administrador hace clic en botón "Actualizar" o ícono de refresh | Sistema detecta clic | Sistema recarga la lista de usuarios desde el servidor manteniendo filtros y búsqueda activos. Muestra indicador de carga breve. Útil para ver cambios recientes realizados por otros administradores | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 25 | Estructura obligatoria de registro de auditoría | Cualquier evento relacionado con visualización de usuarios (acceso a pantalla, búsqueda, filtrado, visualización de detalle) | Sistema registra evento | Registro contiene: ID Único del Evento (UUID), Tipo de Evento (formato MODULO_ENTIDAD_ACCION), Fecha y Hora (UTC con milisegundos, ISO 8601), Usuario (nombre del Administrador), Cliente (NULL), Cliente (Nombre - NULL), IP Local, IP Pública, Resultado (EXITOSO/FALLIDO), Descripción, Nivel de Severidad (INFO/WARNING), Datos Adicionales (JSON con contexto: filtros aplicados, término de búsqueda, usuario visualizado, etc.) | ☐ | ☐ | ☐ |
| 26 | Auditoría de acceso a pantalla de usuarios | Administrador del Portal accede a Gestión de Usuarios | Administrador hace clic en menú | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_USUARIOS_ACCESO", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Administrador {usuario} accedió a pantalla de Gestión de Usuarios", Datos Adicionales: {"total_usuarios_sistema": 150, "timestamp_acceso": "..."} | ☐ | ☐ | ☐ |
| 27 | Auditoría de búsqueda de usuarios | Administrador realiza búsqueda por término específico | Administrador busca "Juan Pérez" | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_USUARIOS_BUSQUEDA", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Administrador {usuario} buscó usuarios con término '{término}'", Datos Adicionales: {"termino_busqueda": "Juan Pérez", "resultados_encontrados": 3, "filtros_activos": {"estado": "Activo"}} | ☐ | ☐ | ☐ |
| 28 | Auditoría de aplicación de filtros | Administrador aplica filtros a la lista | Administrador selecciona Estado=Bloqueado, Cliente=Empresa ABC y hace clic en "Aplicar Filtros" | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_USUARIOS_FILTRADO", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Administrador {usuario} aplicó filtros a listado de usuarios", Datos Adicionales: {"filtros": {"estado": "Bloqueado", "cliente_id": "UUID", "cliente_nombre": "Empresa ABC"}, "resultados": 2} | ☐ | ☐ | ☐ |
| 29 | Auditoría de visualización de detalle de usuario | Administrador hace clic en "Ver Detalle" de un usuario | Administrador accede a detalle de usuario específico | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_USUARIOS_DETALLE_VISUALIZADO", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Administrador {usuario} visualizó detalle de usuario {usuario_id}", Datos Adicionales: {"usuario_visualizado_id": "UUID", "usuario_visualizado_numero_id": "123456789", "usuario_visualizado_nombre": "Juan Pérez Gómez"} | ☐ | ☐ | ☐ |
| 30 | Auditoría de acceso no autorizado | Usuario sin rol Administrador intenta acceder a gestión de usuarios | Usuario no autorizado navega a URL | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_USUARIOS_ACCESO_DENEGADO", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} sin permisos intentó acceder a Gestión de Usuarios", Datos Adicionales: {"rol_usuario": "Gestor Emisión FE", "url_intentada": "/admin/usuarios"} | ☐ | ☐ | ☐ |
| 31 | Inmutabilidad de registros de auditoría | Cualquier escenario de auditoría | Sistema genera registro | Registros de auditoría NO pueden ser modificados ni eliminados. Solo se permiten consultas de lectura | ☐ | ☐ | ☐ |
| 32 | Consulta de registros de auditoría de acceso a usuarios | Auditor requiere revisar quién ha consultado información de usuarios | Auditor accede a módulo de auditoría | Sistema permite filtrar por: rango de fechas, tipo de evento (ADMINISTRACION_USUARIOS_*), administrador que realizó consulta, término de búsqueda usado, usuario visualizado | ☐ | ☐ | ☐ |
| 33 | Retención de registros de auditoría | Registros de auditoría de visualización de usuarios | Paso del tiempo | Registros se conservan por tiempo mínimo de 7 años (política estándar). Después pueden ser archivados o eliminados según política de retención | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Completado**

Los mockups para esta historia de usuario están disponibles en la carpeta `/Mockups`:

#### 1. **HU005-GestionUsuarios.html** - Pantalla principal de gestión de usuarios
**Descripción:** Pantalla principal con tabla de usuarios, búsqueda, filtros y paginación.

**Elementos clave:**

**Barra superior:**
- Título: "Gestión de Usuarios"
- Botón "Crear Nuevo Usuario" (gradiente primary → secondary, ícono +)

**Panel de búsqueda y filtros:**
- Buscador general (TextField outlined, ícono de lupa, placeholder: "Buscar por nombre, apellido, número de identificación o correo...")
- Filtros en fila:
  - Estado (select: Todos, Activo, Inactivo, Bloqueado)
  - Tipo de Usuario (select: Todos, Usuario Interno, Usuario de Cliente)
  - Cliente (select con búsqueda, opciones: Todos, Solo Internos (Sin Cliente), Lista de empresas)
- Botones: "Aplicar Filtros" (contained primary, prominente), "Limpiar Filtros" (text)

**Contador de resultados:**
- Texto: "Mostrando 1-20 de 150 usuarios" o "Mostrando 5 resultados para 'Juan Pérez'"
- Botón "Actualizar" para refresh manual

**Tabla de usuarios:**
- Columnas:
  1. Número ID (sortable)
  2. Nombre Completo (sortable)
  3. Correo Electrónico
  4. Tipo (badge: "Interno" primary / "Cliente" secondary)
  5. Estado (chip: "Activo" success / "Inactivo" default / "Bloqueado" error)
  6. Permisos Asignados (enlace clickeable "X permisos", muestra popover al hacer clic)
  7. Acciones (iconos: Ver Detalle [ojo], Editar [lápiz], Más Opciones [tres puntos])
- Hover en fila: fondo gris claro
- Altura fija con scroll vertical si hay muchos registros

**Paginación:**
- Selector de registros por página (select: 10, 20, 50, 100)
- Botones: "Anterior" (disabled si página 1), "Siguiente" (disabled si última página)
- Indicador de página: "Página 1 de 8"

**Estado vacío:**
- Ilustración o ícono grande
- Mensaje: "Aún no hay usuarios registrados en el sistema"
- Botón "Crear Nuevo Usuario"

**Estado sin resultados:**
- Ícono de búsqueda vacía
- Mensaje: "No se encontraron usuarios que coincidan con la búsqueda"
- Botón "Limpiar Filtros"

**Popover de permisos:**
- Lista de permisos con formato: "Empresa ABC - Administrador de Cliente"
- Si >5 permisos, mostrar primeros 5 + enlace "Ver todos en detalle"

**Menú contextual "Más Opciones":**
- Opciones: Resetear Contraseña, Inactivar/Activar Usuario, Ver Auditoría, Eliminar Usuario
- Cada opción con ícono correspondiente

---

#### 2. **HU005-DetalleUsuario.html** - Pantalla de detalle de usuario
**Descripción:** Pantalla de visualización detallada de un usuario específico (Escenario 12).

**Elementos clave:**

**Header:**
- Breadcrumb: Gestión de Usuarios > Detalle de Usuario
- Nombre completo del usuario como título principal
- Badges de Tipo (Interno/Cliente) y Estado (Activo/Inactivo/Bloqueado)
- Número de identificación
- Botones: "Editar Usuario", "Volver"

**Sección: Datos Personales (card)**
- Número de Identificación
- Primer Nombre, Segundo Nombre
- Primer Apellido, Segundo Apellido
- Correo Electrónico (con ícono)
- Tipo de Usuario (badge)
- Estado (chip)
- Fecha de Creación
- Creado por

**Sección: Información de Sesión (card)**
- Último Acceso (fecha/hora con ícono de check)
- IP del Último Acceso (IP pública y local)
- Total de Accesos (número de sesiones)
- Intentos Fallidos
- Contraseña Actualizada (fecha/hora)

**Sección: Clientes Asociados y Permisos (card expandida)**
- Cards por cada cliente asociado mostrando:
  - Nombre de la empresa
  - Badge con número de permisos
  - Tags de roles asignados con íconos
- Card separada para "Roles Internos"

**Sección: Historial de Permisos (card con timeline)**
- Timeline vertical mostrando:
  - Fecha y hora de cada cambio
  - Tipo de acción (Permiso Asignado/Revocado)
  - Detalle del permiso (Cliente - Rol)
  - Usuario que realizó la acción

**Sección: Historial de Cambios de Estado (card con timeline)**
- Timeline vertical mostrando:
  - Fecha y hora de cada cambio
  - Cambio de estado (de X a Y)
  - Razón del cambio (si aplica)
  - Usuario que realizó la acción

**Sección: Auditoría de Acciones del Usuario (card con tabla)**
- Tabla con últimas 10 acciones mostrando:
  - Fecha y Hora
  - Acción realizada
  - Módulo
  - Resultado (chip success/error)
  - IP
- Botón "Ver Auditoría Completa"

---

#### 3. **HU005-CambiarEstadoUsuario.html** - Modales de cambio de estado
**Descripción:** Modales de confirmación para inactivar o activar usuarios con campo obligatorio de razón (Escenario 14).

**Elementos clave:**

**Modal: Inactivar Usuario**
- Header con ícono de warning (fondo amarillo claro)
- Título: "Inactivar Usuario"
- Párrafo descriptivo: "Está a punto de inactivar al siguiente usuario. El usuario no podrá acceder al sistema hasta que sea reactivado."
- Información del usuario en card destacado:
  - Nombre completo con ícono
  - Número ID
  - Correo electrónico
  - Tipo (badge)
  - Estado actual (chip)
- Campo de texto obligatorio (textarea):
  - Label: "Razón de la inactivación *"
  - Placeholder descriptivo
  - Contador de caracteres (máx 500)
  - Helper text: "Esta razón quedará registrada en la auditoría"
- Footer con botones:
  - "Cancelar" (text)
  - "Inactivar Usuario" (danger, rojo)

**Modal: Activar Usuario**
- Header con ícono de check_circle (fondo verde claro)
- Título: "Activar Usuario"
- Párrafo descriptivo: "Está a punto de activar al siguiente usuario. El usuario podrá acceder nuevamente al sistema con sus permisos previamente asignados."
- Información del usuario en card destacado (mismo formato)
- Campo de texto obligatorio (textarea):
  - Label: "Razón de la activación *"
  - Placeholder descriptivo
  - Contador de caracteres (máx 500)
  - Helper text: "Esta razón quedará registrada en la auditoría"
- Footer con botones:
  - "Cancelar" (text)
  - "Activar Usuario" (success, verde)

**Validaciones:**
- Campo de razón es obligatorio
- Mínimo 10 caracteres
- Máximo 500 caracteres
- Se muestra error si se intenta enviar sin completar

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Rendimiento con gran volumen de usuarios** | Si el sistema tiene miles de usuarios (5000+), la carga inicial de la tabla y búsquedas pueden ser lentas, afectando la experiencia del administrador | Medio | Media | Implementar paginación en backend (no cargar todos los registros). Búsqueda y filtros deben ejecutarse en servidor con índices en base de datos (numero_identificacion, nombre, apellido, correo). Lazy loading de tabla. Límite de 100 registros por página máximo. Considerar búsqueda con debounce (300ms) | Mitigado |
| **Exposición de información sensible en pantalla compartida** | Si un administrador está compartiendo pantalla en videollamada y visualiza la lista de usuarios, podría exponer números de identificación, correos, y permisos de todos los usuarios del sistema a personas no autorizadas | Medio | Baja | Agregar indicador visual cuando pantalla está siendo compartida (funcionalidad del navegador). Incluir watermark o banner: "Información Confidencial - No Compartir". Capacitar a administradores sobre privacidad de datos. NO mostrar contraseñas ni información financiera en ninguna vista | Aceptado con concientización |
| **Búsqueda SQL injection** | Si la búsqueda no está correctamente parametrizada, un administrador malicioso podría inyectar código SQL en el campo de búsqueda para acceder a datos no autorizados o corromper la base de datos | Alto | Baja | Usar consultas parametrizadas (prepared statements) en todas las búsquedas y filtros. Validar y sanitizar entrada del usuario en backend. Implementar ORM que prevenga inyección SQL. Auditar código de búsqueda en revisiones de seguridad | Mitigado |
| **Filtros que no persisten durante navegación** | Si un administrador aplica filtros (ej: solo usuarios bloqueados), navega a detalle de un usuario, y vuelve, espera que los filtros sigan activos. Si no persisten, debe re-aplicarlos, causando frustración | Bajo | Media | Persistir estado de filtros, búsqueda y paginación en URL (query parameters) o en localStorage. Al volver de detalle/edición, restaurar estado anterior. Botón "Volver" del navegador debe mantener contexto | Mitigado |
| **Información desactualizada si hay cambios concurrentes** | Si dos administradores están viendo la lista de usuarios simultáneamente y uno inactiva a un usuario, el otro no ve el cambio hasta que recargue manualmente, pudiendo tomar decisiones sobre datos obsoletos | Bajo | Media | Implementar botón "Actualizar" visible. Opcionalmente, implementar auto-refresh cada 30 segundos en background. Mostrar timestamp de última actualización: "Actualizado hace 2 minutos". Considerar notificaciones en tiempo real (WebSockets) para cambios importantes | Aceptado con refresh manual |
| **Usuarios con nombres muy largos rompen diseño de tabla** | Si un usuario tiene nombres o apellidos muy largos (ej: nombres compuestos de 50+ caracteres), puede romper el diseño de la tabla y dificultar lectura | Bajo | Baja | Aplicar truncamiento de texto con ellipsis (...) en columnas de nombre. Mostrar tooltip con nombre completo al pasar mouse. Límite de ancho de columna definido en CSS. Permitir redimensionar columnas opcionalmente | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance (Out of Scope)
- Edición de usuarios desde esta pantalla (se cubre en HU006 - Creación de Usuarios, pantalla de edición)
- Creación de usuarios desde esta pantalla (se cubre en HU006)
- Reseteo de contraseña (se cubrirá en HU007 - Gestión de Contraseñas de Usuarios)
- Inactivación/Activación masiva de usuarios (se evaluará en Release 2)
- Importación de usuarios desde archivo CSV/Excel (se evaluará en Release 2)
- Exportación de listado de usuarios a CSV/Excel (se evaluará en Release 2 si es necesario)
- Aplicación automática de filtros en tiempo real - se requiere hacer clic explícito en botón "Aplicar Filtros"
- Notificaciones en tiempo real de cambios (se evaluará si se requiere)
- Visualización de historial de sesiones de usuario (se cubre en pantalla de Detalle de Usuario)

### Referencias
- **Estándar de Auditoría Funcional**: Tipos de eventos `ADMINISTRACION_USUARIOS_*`, severidades INFO/WARNING
- **Glosario**: Usuario, Usuario Interno, Usuario de Cliente, Administrador del Portal, Rol, Permiso, Estado de Usuario

