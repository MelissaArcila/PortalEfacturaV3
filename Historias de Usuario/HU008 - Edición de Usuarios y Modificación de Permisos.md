# HU008 - Edición de Usuarios y Modificación de Permisos

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
| **Arquitectura relacionada** | Portal Unificado - Módulo de Administración, Gestión de Usuarios y Permisos, Control de Acceso |

---

## Contexto

El Portal Unificado de CDN Facturación requiere la capacidad de modificar la información de Usuarios existentes y actualizar sus permisos de acceso. Esta funcionalidad es crítica para mantener la información de usuarios actualizada, corregir errores en datos personales, ajustar permisos cuando cambian las responsabilidades de un Usuario, agregar o revocar accesos a Clientes, y gestionar el estado operativo de las cuentas (activar, inactivar, desbloquear).

Los cambios en permisos de usuario son especialmente sensibles desde el punto de vista de seguridad y auditoría, ya que afectan directamente el control de acceso a información confidencial y operaciones críticas del negocio. Por esta razón, cada modificación debe ser registrada con detalle completo de qué cambió, quién lo cambió, cuándo y por qué.

La edición de usuarios debe mantener la integridad referencial del sistema: no se puede eliminar el último Administrador de Portal, no se puede modificar el Número de Identificación (que es inmutable), y los permisos deben ser consistentes con los Productos contratados por cada Cliente.

---

**Notas:**
- Solo Usuarios con rol de Administrador del Portal pueden editar Usuarios
- El Número de Identificación es inmutable - NO se puede modificar una vez creado el Usuario
- Se pueden modificar: nombres, apellidos, correo electrónico, estado, y permisos
- La modificación de permisos implica agregar nuevos o eliminar existentes (no "editar" un permiso)
- Todo cambio genera un registro de auditoría separado por campo modificado
- Si un Usuario está bloqueado automáticamente, el Administrador puede desbloquearlo manualmente
- No se puede inactivar o eliminar el último Administrador de Portal del sistema
- Los datos se muestran pre-poblados con los valores actuales del Usuario
- Se requiere confirmación antes de aplicar cambios

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Administrador del Portal** | Quiero poder editar la información personal de un Usuario existente | Para corregir errores, actualizar datos de contacto y mantener la información del Usuario actualizada |
| Como **Administrador del Portal** | Quiero poder cambiar el estado de un Usuario (Activo, Inactivo, Desbloqueado) | Para controlar el acceso al sistema, desactivar cuentas temporalmente o desbloquear usuarios bloqueados por intentos fallidos |
| Como **Administrador del Portal** | Quiero poder agregar nuevos permisos a un Usuario existente | Para otorgar acceso a nuevos Clientes o Roles sin tener que crear un nuevo Usuario |
| Como **Administrador del Portal** | Quiero poder eliminar permisos de un Usuario existente | Para revocar accesos cuando un Usuario ya no debe tener permisos sobre un Cliente o Rol específico |
| Como **Administrador del Portal** | Quiero ver un resumen de todos los cambios antes de confirmarlos | Para revisar que las modificaciones son correctas y evitar errores que afecten el acceso del Usuario |
| Como **Administrador del Portal** | Quiero que el sistema preserve el historial de cambios realizados a cada Usuario | Para poder auditar quién modificó qué información y cuándo, cumpliendo con requisitos de seguridad |
| Como **Auditor** | Quiero que cada cambio en usuarios (datos personales, estado, permisos) sea registrado en auditoría | Para garantizar trazabilidad completa de modificaciones y detectar cambios no autorizados |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Acceso a edición desde Gestión de Usuarios | Administrador del Portal en pantalla de Gestión de Usuarios (HU005) | Administrador hace clic en icono "Editar" (lápiz) de un Usuario en la tabla | Sistema navega a pantalla de "Editar Usuario" con URL `/admin/usuarios/{usuario_id}/editar`. Muestra formulario estructurado en secciones: "Datos Personales", "Estado", "Permisos". Todos los campos pre-poblados con valores actuales del Usuario | ☐ | ☐ | ☐ |
| 2 | Visualización de Número de Identificación inmutable | Administrador en formulario de edición | Sistema carga datos del Usuario | Campo "Número de Identificación" se muestra como solo lectura (disabled, fondo gris). Muestra valor actual del Usuario. Texto informativo debajo: "El Número de Identificación no puede ser modificado después de la creación del usuario" con ícono de información | ☐ | ☐ | ☐ |
| 3 | Visualización de Tipo de Usuario inmutable | Administrador en formulario de edición | Sistema carga datos del Usuario | Sección "Tipo de Usuario" muestra badge "Usuario Interno" o "Usuario de Cliente" en solo lectura. Texto informativo: "El tipo de usuario se determina automáticamente según los permisos asignados y no puede modificarse directamente" | ☐ | ☐ | ☐ |
| 4 | Edición de Primer Apellido | Administrador modifica campo "Primer Apellido" | Administrador ingresa nuevo valor | Sistema acepta letras (A-Z, a-z), espacios, guiones, apóstrofes. NO acepta números ni símbolos especiales. Campo obligatorio (*Primer Apellido). Máximo 50 caracteres. Muestra indicador de cambio pendiente (color diferente o ícono) | ☐ | ☐ | ☐ |
| 5 | Edición de Segundo Apellido | Administrador modifica o deja vacío campo "Segundo Apellido" | Administrador ingresa nuevo valor o borra | Sistema acepta letras, espacios, guiones, apóstrofes. NO acepta números ni símbolos especiales. Campo OPCIONAL. Máximo 50 caracteres. Si se deja vacío, se acepta sin error | ☐ | ☐ | ☐ |
| 6 | Edición de Primer Nombre | Administrador modifica campo "Primer Nombre" | Administrador ingresa nuevo valor | Sistema acepta letras, espacios, guiones, apóstrofes. NO acepta números ni símbolos especiales. Campo obligatorio (*Primer Nombre). Máximo 50 caracteres. Muestra indicador de cambio pendiente | ☐ | ☐ | ☐ |
| 7 | Edición de Segundo Nombre | Administrador modifica o deja vacío campo "Segundo Nombre" | Administrador ingresa nuevo valor o borra | Sistema acepta letras, espacios, guiones, apóstrofes. Campo OPCIONAL. Máximo 50 caracteres | ☐ | ☐ | ☐ |
| 8 | Edición de Correo Electrónico | Administrador modifica campo "Correo Electrónico" | Administrador ingresa nuevo correo y hace blur | Sistema valida formato de correo electrónico usando regex estándar (debe contener @, dominio válido, sin espacios). Campo OPCIONAL. Si formato es válido, muestra checkmark verde. Si formato es inválido, muestra error: "Ingrese un correo electrónico válido (ejemplo: usuario@dominio.com)". Permite dejarlo vacío | ☐ | ☐ | ☐ |
| 9 | Cambio de Estado a Inactivo | Administrador en sección "Estado" hace clic en dropdown de Estado | Administrador selecciona "Inactivo" | Sistema muestra opciones: "Activo", "Inactivo". Al seleccionar "Inactivo", muestra diálogo de confirmación: "¿Está seguro que desea inactivar este usuario? El usuario no podrá iniciar sesión hasta que se reactive." Botones: "Cancelar", "Confirmar". Si confirma, marca el estado como pendiente de cambio | ☐ | ☐ | ☐ |
| 10 | Cambio de Estado a Activo | Administrador selecciona Estado "Activo" para usuario Inactivo o Bloqueado | Administrador confirma cambio | Sistema cambia estado a "Activo". Si el Usuario estaba Bloqueado automáticamente, limpia el bloqueo y resetea contadores de intentos fallidos. Marca el estado como pendiente de cambio con indicador visual | ☐ | ☐ | ☐ |
| 11 | Prevención de inactivar último Administrador del Portal | Administrador intenta inactivar un Usuario que es el único con rol de Administrador del Portal | Administrador selecciona Estado "Inactivo" | Sistema valida y muestra error bloqueante: "No se puede inactivar este usuario porque es el único Administrador del Portal activo en el sistema. Asigne el rol de Administrador del Portal a otro usuario antes de continuar." No permite cambiar el estado | ☐ | ☐ | ☐ |
| 12 | Visualización de permisos actuales en tabla | Administrador en sección "Permisos" | Sistema carga permisos del Usuario | Sistema muestra tabla "Permisos Actuales" con columnas: Empresa (nombre o "Interno"), Rol (nombre), Fecha Asignación, Asignado Por (admin que creó el permiso), Acción (botón Eliminar). Muestra todos los permisos del usuario. Paginación si >10 permisos | ☐ | ☐ | ☐ |
| 13 | Agregar nuevo permiso a usuario existente | Administrador en sección "Permisos" hace clic en "Agregar Nuevo Permiso" | Sistema muestra formulario de nuevo permiso | Sistema muestra formulario igual a HU006: Selección de Empresa (dropdown con búsqueda, incluyendo "Sin Cliente (Rol Interno)" si aplica), Selección de Rol (dropdown dinámico según empresa y productos contratados), Botón "Agregar Permiso". Al agregar, valida que no sea duplicado y agrega fila a tabla marcada como "Nuevo" con badge verde | ☐ | ☐ | ☐ |
| 14 | Prevención de permisos duplicados al agregar | Administrador intenta agregar permiso que ya existe (misma Empresa + mismo Rol) | Administrador hace clic en "Agregar Permiso" | Sistema valida y detecta duplicado. Muestra mensaje de error: "Este permiso ya existe para este usuario. El usuario ya tiene el rol [Nombre Rol] en [Nombre Empresa]." No agrega el permiso duplicado a la tabla | ☐ | ☐ | ☐ |
| 15 | Eliminar permiso existente con confirmación | Administrador hace clic en botón "Eliminar" junto a un permiso en la tabla de permisos actuales | Sistema detecta clic | Sistema muestra diálogo de confirmación: "¿Está seguro que desea eliminar el permiso [Rol] en [Empresa] para este usuario? Esta acción afectará su acceso." Botones: "Cancelar", "Confirmar". Si confirma, marca la fila con tachado y badge "A eliminar" en rojo. No la elimina de la tabla hasta confirmar todo | ☐ | ☐ | ☐ |
| 16 | Restaurar permiso marcado para eliminar | Administrador marcó un permiso para eliminar pero se arrepiente | Administrador hace clic en "Deshacer" junto al permiso marcado para eliminar | Sistema remueve el marcado de eliminación, quita el tachado y el badge "A eliminar". Permiso vuelve a su estado original. Útil para deshacer errores antes de confirmar | ☐ | ☐ | ☐ |
| 17 | Validación de al menos un permiso después de eliminar | Administrador marca TODOS los permisos actuales para eliminar sin agregar nuevos | Administrador intenta guardar cambios | Sistema valida y muestra error bloqueante: "El usuario debe tener al menos un permiso asignado. No puede eliminar todos los permisos. Si desea inactivar el usuario, cambie su estado a Inactivo." No permite guardar hasta agregar al menos 1 permiso | ☐ | ☐ | ☐ |
| 18 | Prevención de eliminar último permiso de Administrador de Portal | Usuario tiene solo rol de Administrador del Portal y es el único admin activo | Administrador intenta eliminar ese permiso | Sistema valida y muestra error: "No se puede eliminar este permiso porque el usuario es el único Administrador del Portal activo en el sistema. Asigne el rol a otro usuario antes de continuar." No permite marcar para eliminar | ☐ | ☐ | ☐ |
| 19 | Indicador visual de cambios pendientes | Administrador ha modificado varios campos y permisos | Administrador visualiza formulario | Sistema muestra indicadores visuales claros de cambios: campos modificados con borde azul o ícono de lápiz, permisos nuevos con badge "Nuevo" verde, permisos a eliminar tachados con badge "A eliminar" rojo. Contador en parte superior: "X cambios pendientes" | ☐ | ☐ | ☐ |
| 20 | Botón Guardar Cambios habilitado con datos válidos | Administrador ha realizado cambios y todos los campos son válidos | Sistema valida en tiempo real | Botón "Guardar Cambios" se habilita (color primario, gradiente, clickeable). Si hay campos inválidos o validaciones que fallan, botón permanece deshabilitado (gris, cursor not-allowed) con tooltip explicando qué falta | ☐ | ☐ | ☐ |
| 21 | Pantalla de resumen antes de guardar cambios | Administrador completa edición y hace clic en "Guardar Cambios" | Sistema valida y muestra resumen | Sistema muestra pantalla de resumen con secciones: **(1) Cambios en Datos Personales**: tabla comparativa "Antes → Después" solo de campos modificados, **(2) Cambios en Estado**: si cambió, muestra "Antes: [estado] → Después: [estado]", **(3) Cambios en Permisos**: subsecciones "Permisos Agregados" (lista de nuevos con Empresa-Rol), "Permisos Eliminados" (lista de eliminados con Empresa-Rol), "Permisos Sin Cambios" (total). Botones: "Volver y Editar", "Confirmar Cambios" | ☐ | ☐ | ☐ |
| 22 | Resumen sin cambios realizados | Administrador abre edición pero NO modifica nada y hace clic en "Guardar Cambios" | Sistema detecta ausencia de cambios | Sistema muestra mensaje informativo: "No se han realizado cambios en este usuario. No hay nada que guardar." Botones: "Volver a Editar", "Cancelar y Salir" | ☐ | ☐ | ☐ |
| 23 | Volver a editar desde resumen | Administrador en pantalla de resumen hace clic en "Volver y Editar" | Sistema detecta clic | Sistema regresa al formulario de edición manteniendo TODOS los cambios pendientes: campos modificados, permisos agregados, permisos marcados para eliminar. Ningún cambio se pierde. Administrador puede modificar y volver a intentar guardar | ☐ | ☐ | ☐ |
| 24 | Guardado exitoso de cambios | Administrador en pantalla de resumen hace clic en "Confirmar Cambios" | Sistema procesa los cambios | Sistema aplica todos los cambios: actualiza datos personales, cambia estado si aplica, agrega nuevos permisos, elimina permisos marcados. Registra fecha/hora de modificación, usuario modificador. Muestra mensaje de éxito: "¡Usuario actualizado exitosamente! Los cambios en [Nombre Completo] han sido guardados. Resumen: [X] campos modificados, [Y] permisos agregados, [Z] permisos eliminados." Botones: "Ver Usuario" (detalle), "Volver a Gestión de Usuarios" | ☐ | ☐ | ☐ |
| 25 | Error durante guardado | Administrador confirma cambios pero ocurre error técnico (timeout, error BD, validación backend fallida) | Sistema intenta guardar pero falla | Sistema muestra mensaje de error genérico: "Ocurrió un error al guardar los cambios. Por favor, intente nuevamente. Si el problema persiste, contacte a soporte técnico." Botón "Aceptar". Al hacer clic, sistema regresa al formulario manteniendo TODOS los cambios pendientes. Administrador puede revisar y volver a intentar | ☐ | ☐ | ☐ |
| 26 | Cancelar edición con cambios pendientes | Administrador en medio de edición con cambios pendientes hace clic en "Cancelar" | Sistema detecta cambios no guardados | Sistema muestra diálogo de confirmación: "¿Está seguro que desea cancelar? Se perderán todos los cambios realizados." Lista cambios pendientes (ej: "3 campos modificados, 2 permisos agregados"). Botones: "Continuar Editando", "Sí, Cancelar". Si confirma, descarta cambios y regresa a Gestión de Usuarios. Si continúa editando, cierra diálogo y mantiene cambios | ☐ | ☐ | ☐ |
| 27 | Cancelar edición sin cambios | Administrador no ha realizado cambios y hace clic en "Cancelar" | Sistema detecta ausencia de cambios | Sistema regresa directamente a Gestión de Usuarios sin mostrar confirmación (no hay nada que perder) | ☐ | ☐ | ☐ |
| 28 | Desbloqueo manual de usuario bloqueado | Usuario en estado "Bloqueado" por intentos fallidos, Administrador cambia estado a "Activo" | Administrador confirma cambios | Sistema cambia estado a "Activo", resetea contador de intentos fallidos a cero, limpia timestamp de bloqueo automático. Usuario puede volver a iniciar sesión inmediatamente. Genera auditoría de desbloqueo manual | ☐ | ☐ | ☐ |
| 29 | Información contextual de usuario bloqueado | Administrador edita usuario en estado "Bloqueado" | Sistema muestra datos de estado | Junto al estado "Bloqueado" muestra información contextual: "Bloqueado por [razón]: [5] intentos fallidos el [fecha/hora]. Desbloqueo automático programado para: [fecha/hora]" O "Bloqueado manualmente por [admin] el [fecha/hora]". Permite al admin decidir si desbloquear manualmente o dejar que se desbloquee automáticamente | ☐ | ☐ | ☐ |
| 30 | Filtros en tabla de permisos actuales | Administrador visualiza usuario con 20+ permisos | Administrador usa filtros | Sistema muestra filtros encima de tabla de permisos: "Filtrar por Empresa" (dropdown con empresas del usuario), "Filtrar por Rol" (dropdown con roles del usuario). Al seleccionar filtro, muestra solo filas que coinciden. Botón "Limpiar Filtros" visible. Útil para encontrar permiso específico a eliminar | ☐ | ☐ | ☐ |
| 31 | Ordenamiento de tabla de permisos | Administrador hace clic en encabezado de columna "Empresa" en tabla de permisos | Sistema detecta clic | Sistema ordena permisos alfabéticamente por nombre de Empresa ascendente. Muestra indicador de ordenamiento (flecha arriba). Al hacer clic nuevamente, invierte orden. Permite ordenar por: Empresa, Rol, Fecha Asignación | ☐ | ☐ | ☐ |
| 32 | Validación de permiso de acceso a edición | Usuario sin rol de Administrador del Portal intenta acceder a edición mediante URL directa | Usuario no autorizado navega a URL | Sistema valida permisos. Muestra mensaje: "No tiene permisos para acceder a esta sección. Solo Administradores del Portal pueden editar usuarios." y redirige a pantalla de inicio | ☐ | ☐ | ☐ |
| 33 | Validación de existencia del usuario a editar | Administrador navega a URL de edición con ID de usuario que no existe | Sistema intenta cargar usuario | Sistema muestra mensaje de error: "El usuario solicitado no existe o ha sido eliminado." Botón "Volver a Gestión de Usuarios". Código HTTP 404 | ☐ | ☐ | ☐ |
| 34 | Empresa inactiva no aparece al agregar permisos | Existe Empresa en el sistema pero está Inactiva | Administrador abre dropdown "Selección de Empresa" para agregar permiso | Sistema NO muestra empresas inactivas. Solo muestra Empresas con estado Activo. Previene asignar permisos a empresas que no están operativas. SIN EMBARGO, permisos existentes de empresas inactivas SÍ se muestran en tabla de permisos actuales con indicador visual (badge "Empresa Inactiva" naranja) | ☐ | ☐ | ☐ |
| 35 | Empresa sin productos contratados al agregar permiso | Empresa activa pero sin productos contratados | Administrador selecciona esa Empresa en dropdown al agregar permiso | Sistema muestra dropdown "Selección de Rol" con solo: "Administrador de Cliente". NO muestra roles de gestor. Mensaje informativo: "Esta empresa no tiene productos contratados. Solo puede asignar rol Administrador de Cliente" | ☐ | ☐ | ☐ |
| 36 | Producto cancelado - permiso huérfano visible | Usuario tiene permiso "Gestor Emisión FE" pero Empresa ABC canceló Producto "Emisión FE" | Administrador visualiza permisos | Sistema muestra el permiso en tabla con badge de advertencia "Producto No Contratado" en naranja. Tooltip: "Este rol está asociado a un producto que la empresa ya no tiene contratado. Considere eliminar este permiso." Permite al admin identificar y limpiar permisos obsoletos | ☐ | ☐ | ☐ |
| 37 | Búsqueda rápida de empresa al agregar permiso | Administrador agregando permiso, hay 200 empresas en el sistema | Administrador escribe en campo "Selección de Empresa" | Dropdown permite typeahead search. Al escribir "ABC", filtra y muestra solo empresas cuyo nombre contiene "ABC". Búsqueda case-insensitive. Mejora UX para grandes volúmenes | ☐ | ☐ | ☐ |
| 38 | Historial de cambios visible en formulario | Administrador edita usuario que ya fue modificado anteriormente | Sistema carga formulario | Sistema muestra sección colapsable "Historial de Cambios" al final del formulario. Al expandir, muestra últimas 10 modificaciones: fecha/hora, campo modificado, valor anterior → valor nuevo, usuario que modificó. Link "Ver historial completo" navega a auditoría. Útil para contexto antes de hacer cambios | ☐ | ☐ | ☐ |
| 39 | Indicador de último acceso del usuario | Administrador edita usuario | Sistema muestra información | En sección de información general (parte superior), muestra: "Último acceso: [fecha/hora] desde IP [dirección]" O "Este usuario nunca ha iniciado sesión" si aplica. Ayuda al admin saber si el usuario está activo | ☐ | ☐ | ☐ |
| 40 | Autoguardado temporal opcional | Administrador ha realizado cambios en el formulario | Cada 30 segundos de inactividad | Sistema opcionalmente guarda cambios pendientes en localStorage del navegador como respaldo. Si navegador se cierra accidentalmente y se vuelve a abrir la misma edición, muestra mensaje: "Detectamos cambios sin guardar. ¿Desea recuperarlos?" con botones "Recuperar" y "Descartar". Previene pérdida de trabajo | ☐ | ☐ | ☐ |
| 41 | Cambio de correo electrónico - validación de unicidad opcional | Administrador cambia correo de usuario a uno que ya existe en otro usuario | Administrador hace blur del campo | Sistema OPCIONALMENTE valida unicidad de correo electrónico. Si otro usuario ya tiene ese correo, muestra advertencia: "Este correo electrónico ya está registrado en el usuario [Nombre]. ¿Desea continuar de todas formas?" Permite duplicados pero advierte. Depende de política de negocio | ☐ | ☐ | ☐ |
| 42 | Rol Interno a Usuario de Cliente - conversión automática de tipo | Usuario tiene solo permisos internos (tipo "Usuario Interno"), Administrador elimina todos los internos y agrega solo permisos de Cliente | Administrador confirma cambios | Sistema actualiza automáticamente el Tipo de Usuario de "Usuario Interno" a "Usuario de Cliente" basándose en sus nuevos permisos. Tipo se recalcula dinámicamente: solo roles internos = Interno, solo roles cliente = Cliente, ambos = Interno con permisos de Cliente | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 43 | Estructura obligatoria de registro de auditoría | Cualquier evento relacionado con edición de usuarios (modificación de datos, cambio de estado, adición/eliminación de permisos) | Sistema registra evento | Registro contiene: ID Único del Evento (UUID), Tipo de Evento (MODULO_ENTIDAD_ACCION), Fecha y Hora (UTC milisegundos ISO 8601), Usuario (Administrador editor), Cliente (NULL), Cliente (Nombre - NULL), IP Local, IP Pública, Resultado (EXITOSO/FALLIDO), Descripción, Nivel de Severidad (INFO/WARNING/ERROR), Datos Adicionales (JSON con detalles: ID usuario editado, campo modificado, valor anterior, valor nuevo, etc.) | ☐ | ☐ | ☐ |
| 44 | Auditoría de acceso a edición de usuario | Administrador del Portal accede a formulario de edición de un usuario | Administrador hace clic en "Editar" | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_USUARIO_EDICION_ACCESO", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Administrador {admin} accedió a edición de usuario {usuario_nombre}", Datos Adicionales: {"usuario_editado_id": "UUID", "usuario_editado_numero_id": "123456789", "usuario_editado_nombre": "Juan Pérez Gómez"} | ☐ | ☐ | ☐ |
| 45 | Auditoría de modificación exitosa de datos personales | Administrador confirma cambios en datos personales (nombres, apellidos, correo) | Administrador confirma guardado | Sistema genera UN registro por CADA campo modificado: Tipo de Evento: "ADMINISTRACION_USUARIO_DATOS_MODIFICADOS", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Administrador {admin} modificó {campo} de usuario {usuario}", Datos Adicionales: {"usuario_id": "UUID", "usuario_numero_id": "123456789", "campo_modificado": "primer_nombre", "valor_anterior": "Juan", "valor_nuevo": "José", "motivo": "opcional"} | ☐ | ☐ | ☐ |
| 46 | Auditoría de cambio de estado | Administrador cambia estado de usuario (Activo ↔ Inactivo, Desbloqueado) | Administrador confirma cambio | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_USUARIO_ESTADO_MODIFICADO", Resultado: "EXITOSO", Severidad: "INFO" o "WARNING" (si es inactivación), Descripción: "Administrador {admin} cambió estado de usuario {usuario} de {estado_anterior} a {estado_nuevo}", Datos Adicionales: {"usuario_id": "UUID", "usuario_nombre": "Juan Pérez", "estado_anterior": "Activo", "estado_nuevo": "Inactivo", "razon_cambio": "opcional"} | ☐ | ☐ | ☐ |
| 47 | Auditoría de desbloqueo manual | Administrador desbloquea manualmente usuario bloqueado por intentos fallidos | Administrador confirma desbloqueo | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_USUARIO_DESBLOQUEADO_MANUAL", Resultado: "EXITOSO", Severidad: "WARNING", Descripción: "Administrador {admin} desbloqueó manualmente a usuario {usuario} que estaba bloqueado", Datos Adicionales: {"usuario_id": "UUID", "usuario_nombre": "Juan Pérez", "razon_bloqueo_original": "5 intentos fallidos", "fecha_bloqueo_original": "timestamp", "desbloqueo_programado_para": "timestamp"} | ☐ | ☐ | ☐ |
| 48 | Auditoría de agregación de permiso | Administrador agrega nuevo permiso a usuario | Administrador confirma cambios | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_USUARIO_PERMISO_AGREGADO", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Administrador {admin} agregó permiso {rol} en {empresa} a usuario {usuario}", Datos Adicionales: {"usuario_id": "UUID", "usuario_nombre": "Juan Pérez", "empresa_id": "UUID", "empresa_nombre": "Empresa ABC", "rol_id": "3", "rol_nombre": "Gestor Emisión FE", "fecha_asignacion": "timestamp"} | ☐ | ☐ | ☐ |
| 49 | Auditoría de eliminación de permiso | Administrador elimina permiso existente de usuario | Administrador confirma cambios | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_USUARIO_PERMISO_ELIMINADO", Resultado: "EXITOSO", Severidad: "WARNING", Descripción: "Administrador {admin} eliminó permiso {rol} en {empresa} de usuario {usuario}", Datos Adicionales: {"usuario_id": "UUID", "usuario_nombre": "Juan Pérez", "empresa_id": "UUID", "empresa_nombre": "Empresa ABC", "rol_id": "3", "rol_nombre": "Gestor Emisión FE", "fecha_eliminacion": "timestamp", "fecha_asignacion_original": "timestamp", "asignado_originalmente_por": "admin_nombre"} | ☐ | ☐ | ☐ |
| 50 | Auditoría de intento fallido de edición | Administrador confirma cambios pero validación backend falla (ej: intenta inactivar último admin) | Sistema rechaza cambios | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_USUARIO_EDICION_FALLIDA", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Administrador {admin} intentó editar usuario {usuario} pero la operación falló", Datos Adicionales: {"usuario_id": "UUID", "razon_fallo": "No se puede inactivar último Administrador del Portal", "cambios_intentados": {"estado": "Activo → Inactivo"}} | ☐ | ☐ | ☐ |
| 51 | Auditoría de error técnico durante guardado | Administrador confirma cambios pero error técnico (timeout, BD error) | Sistema intenta guardar pero falla | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_USUARIO_EDICION_ERROR", Resultado: "FALLIDO", Severidad: "ERROR", Descripción: "Error técnico al editar usuario {usuario} por Administrador {admin}", Datos Adicionales: {"usuario_id": "UUID", "usuario_nombre": "Juan Pérez", "error_tipo": "timeout", "error_mensaje": "Mensaje técnico", "cambios_pendientes": "JSON para debugging"} | ☐ | ☐ | ☐ |
| 52 | Auditoría de cancelación de edición | Administrador cancela edición después de hacer cambios pendientes | Administrador confirma cancelación | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_USUARIO_EDICION_CANCELADA", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Administrador {admin} canceló edición de usuario {usuario}", Datos Adicionales: {"usuario_id": "UUID", "usuario_nombre": "Juan Pérez", "cambios_pendientes_descartados": {"campos_modificados": 3, "permisos_agregados": 2, "permisos_eliminados": 1}} | ☐ | ☐ | ☐ |
| 53 | Auditoría de acceso no autorizado | Usuario sin rol Admin intenta acceder a edición de usuario | Usuario no autorizado navega a URL | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_USUARIO_EDICION_ACCESO_DENEGADO", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} sin permisos intentó acceder a edición de usuario", Datos Adicionales: {"rol_usuario": "Gestor FE", "usuario_intentado_editar_id": "UUID", "url_intentada": "/admin/usuarios/{id}/editar"} | ☐ | ☐ | ☐ |
| 54 | Inmutabilidad de registros de auditoría | Cualquier escenario de auditoría de edición | Sistema genera registros | Registros de auditoría NO pueden ser modificados ni eliminados. Solo se permiten consultas de lectura | ☐ | ☐ | ☐ |
| 55 | Consulta de historial de cambios de usuario | Auditor o Administrador requiere ver historial completo de modificaciones de un usuario | Auditor accede a módulo de auditoría filtrando por usuario específico | Sistema muestra todos los registros de auditoría relacionados con ese usuario ordenados cronológicamente: creación, modificaciones de datos, cambios de estado, permisos agregados/eliminados, accesos a edición, cancelaciones. Permite exportar CSV/Excel. Muestra: fecha/hora, admin que realizó cambio, tipo de evento, descripción, antes/después | ☐ | ☐ | ☐ |
| 56 | Retención de registros de auditoría | Registros de auditoría de edición de usuarios | Paso del tiempo | Registros se conservan por mínimo 7 años según política estándar. Después pueden archivarse o eliminarse según política de retención | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

Se requieren los siguientes mockups interactivos en HTML siguiendo la Guía de Estilos:

#### 1. **HU008-EditarUsuario.html** - Formulario de edición de usuario
**Elementos:**

**Sección 1: Información General (Encabezado)**
- Badge de Tipo de Usuario: "Usuario Interno" / "Usuario de Cliente" / "Usuario Interno con permisos de Cliente" (solo lectura)
- Último acceso: "[fecha/hora] desde IP [dirección]" o "Nunca ha iniciado sesión"
- Link "Ver Historial Completo de Cambios"

**Sección 2: Datos Personales**
- *Número de Identificación (input disabled, fondo gris, con tooltip explicativo de inmutabilidad)
- *Primer Apellido (input texto, max 50, pre-poblado, indicador de cambio si modifica)
- Segundo Apellido (input texto, max 50, opcional, pre-poblado)
- *Primer Nombre (input texto, max 50, pre-poblado, indicador de cambio)
- Segundo Nombre (input texto, max 50, opcional, pre-poblado)
- Correo Electrónico (input email, validación formato, opcional, pre-poblado)

**Sección 3: Estado**
- Dropdown de Estado: opciones "Activo", "Inactivo"
- Si estado actual es "Bloqueado": muestra info contextual "Bloqueado por [razón] el [fecha]. Desbloqueo automático: [fecha]"
- Indicador de cambio si modifica

**Sección 4: Permisos**
- Título: "Permisos Asignados"
- Tabla "Permisos Actuales":
  - Columnas: Empresa, Rol, Fecha Asignación, Asignado Por, Acción (Eliminar)
  - Filas con permisos existentes
  - Permisos marcados para eliminar: tachado + badge "A eliminar" rojo + botón "Deshacer"
  - Permisos con advertencias: badge "Empresa Inactiva" o "Producto No Contratado" naranja
  - Filtros: por Empresa, por Rol
  - Paginación si >10
- Botón "Agregar Nuevo Permiso" (outlined, ícono +)
- Formulario de nuevo permiso (igual a HU006):
  - Selección de Empresa (dropdown con búsqueda)
  - Selección de Rol (dropdown dinámico)
  - Botón "Agregar Permiso"
- Permisos agregados en tabla con badge "Nuevo" verde

**Sección 5: Historial de Cambios (Colapsable)**
- Título: "Historial de Cambios" (inicialmente colapsado)
- Al expandir: lista de últimos 10 cambios con fecha/hora, campo, antes→después, admin
- Link "Ver historial completo"

**Indicadores Globales:**
- Contador en parte superior: "X cambios pendientes" (solo si hay cambios)
- Indicadores visuales: campos modificados con borde azul o ícono

**Botones del formulario:**
- "Cancelar" (outlined, confirmación si hay cambios)
- "Guardar Cambios" (contained gradiente, deshabilitado hasta que haya cambios válidos)

#### 2. **HU008-ResumenCambios.html** - Resumen antes de confirmar cambios
**Elementos:**
- Título: "Resumen de Cambios - [Nombre Usuario]"
- Información del usuario: Número ID, Nombre Completo
- Secciones:
  - **Cambios en Datos Personales**:
    - Tabla comparativa con columnas: Campo, Valor Anterior, Valor Nuevo
    - Solo muestra campos modificados
    - Si no hay cambios, muestra "Sin cambios en datos personales"
  - **Cambios en Estado**:
    - "Estado anterior: [badge estado] → Estado nuevo: [badge estado]"
    - Si no hay cambio, muestra "Sin cambios en estado"
  - **Cambios en Permisos**:
    - **Permisos Agregados**: lista con Empresa - Rol (badge "Nuevo" verde)
    - **Permisos Eliminados**: lista con Empresa - Rol (badge "Eliminado" rojo)
    - **Permisos Sin Cambios**: "X permisos se mantienen sin modificar"
- Resumen total: "Total de cambios: [X] campos modificados, [Y] permisos agregados, [Z] permisos eliminados"
- Botones:
  - "Volver y Editar" (outlined)
  - "Confirmar Cambios" (contained gradiente Primary → Secondary)

#### 3. **HU008-EdicionExitosa.html** - Confirmación de guardado exitoso
**Elementos:**
- Icono de éxito (checkmark grande verde)
- Título: "¡Usuario Actualizado Exitosamente!"
- Mensaje: "Los cambios en [Nombre Completo] han sido guardados correctamente."
- Resumen breve:
  - "[X] campos modificados"
  - "[Y] permisos agregados"
  - "[Z] permisos eliminados"
- Botones:
  - "Ver Detalle de Usuario" (navega a detalle)
  - "Volver a Gestión de Usuarios"

#### 4. **HU008-ErrorEdicion.html** - Error durante guardado
**Elementos:**
- Icono de error (warning rojo/naranja)
- Título: "Error al Guardar Cambios"
- Mensaje: "Ocurrió un error al guardar los cambios. Por favor, intente nuevamente. Si el problema persiste, contacte a soporte técnico."
- Detalle técnico (colapsable opcional): "Error: [mensaje técnico]"
- Botón "Aceptar" (regresa al formulario manteniendo cambios)

#### 5. **HU008-SinCambios.html** - Intento de guardar sin cambios
**Elementos:**
- Icono informativo (ícono "i" azul)
- Título: "Sin Cambios Realizados"
- Mensaje: "No se han realizado cambios en este usuario. No hay nada que guardar."
- Botones:
  - "Volver a Editar"
  - "Cancelar y Salir"

#### 6. **HU008-DialogoConfirmacionEliminarPermiso.html** - Confirmación eliminar permiso
**Elementos:**
- Título: "¿Eliminar Permiso?"
- Mensaje: "¿Está seguro que desea eliminar el permiso [Rol] en [Empresa] para este usuario? Esta acción afectará su acceso."
- Advertencia si aplica: "ADVERTENCIA: Este es el único permiso del usuario. Debe agregar al menos otro permiso antes de confirmar los cambios."
- Botones: "Cancelar" (text), "Confirmar" (contained error)

#### 7. **HU008-DialogoConfirmacionInactivar.html** - Confirmación cambio a Inactivo
**Elementos:**
- Título: "¿Inactivar Usuario?"
- Mensaje: "¿Está seguro que desea inactivar este usuario? El usuario no podrá iniciar sesión hasta que se reactive."
- Información: "Usuario: [Nombre Completo] (ID: [Número ID])"
- Botones: "Cancelar" (text), "Confirmar Inactivación" (contained warning)

#### 8. **HU008-DialogoConfirmacionCancelar.html** - Confirmación cancelar con cambios
**Elementos:**
- Título: "¿Cancelar Edición?"
- Mensaje: "¿Está seguro que desea cancelar? Se perderán todos los cambios realizados."
- Lista de cambios pendientes:
  - "[X] campos modificados"
  - "[Y] permisos agregados"
  - "[Z] permisos eliminados"
- Botones: "Continuar Editando" (outlined), "Sí, Cancelar" (contained error)

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Modificación accidental de usuario incorrecto** | Administrador puede abrir edición de usuario equivocado y realizar cambios sin darse cuenta, afectando permisos de persona incorrecta | Alto | Media | Mostrar claramente Número ID y Nombre Completo en parte superior durante toda la edición. Resumen antes de confirmar muestra quién es el usuario. Confirmar identidad antes de cambios críticos | Mitigado |
| **Pérdida de cambios por cierre accidental de navegador** | Administrador realiza 30 minutos de edición compleja (agregando/eliminando múltiples permisos) y cierra navegador accidentalmente, perdiendo todo el trabajo | Alto | Media | Implementar autoguardado temporal en localStorage cada 30 segundos. Al volver a abrir edición del mismo usuario, mostrar diálogo "¿Recuperar cambios no guardados?". Advertir con `beforeunload` antes de cerrar pestaña | Mitigado |
| **Eliminar último Administrador del Portal** | Si se permite eliminar o inactivar al último Admin de Portal activo, el sistema quedaría sin administrador y nadie podría gestionar usuarios | Crítico | Baja | Validación en frontend y backend que previene inactivar o eliminar permisos de Admin Portal si es el único activo. Mensaje claro explicando la restricción. Sugerir crear otro admin primero | Mitigado |
| **Permisos huérfanos por productos cancelados** | Usuarios acumulan permisos para productos que las empresas cancelaron, causando confusión sobre qué accesos son realmente válidos | Medio | Alta | Indicadores visuales en tabla de permisos mostrando "Producto No Contratado". Proceso de limpieza periódica (manual o automático) que inactiva permisos huérfanos. Reportes de permisos obsoletos para revisión | Aceptado con indicadores |
| **Cambios conflictivos por edición simultánea** | Dos administradores editan el mismo usuario simultáneamente. Admin A agrega permiso, Admin B elimina permisos, ambos guardan: ¿qué prevalece? | Alto | Baja | Implementar versionado optimista (optimistic locking): cada usuario tiene campo `version`. Al guardar, validar que versión actual == versión leída. Si difiere, mostrar error: "Este usuario fue modificado por otro administrador. Actualice y vuelva a intentar". Mostrar quién modificó y cuándo | Mitigado |
| **Desbloqueo manual de usuario malicioso** | Administrador desbloquea usuario que fue bloqueado legítimamente por intentos de ataque, permitiendo que continúe intentando acceder | Medio | Baja | Mostrar información contextual completa de por qué fue bloqueado (intentos, fechas, IPs). Registrar en auditoría con severidad WARNING todo desbloqueo manual. Alertas de seguridad si un usuario es desbloqueado múltiples veces | Mitigado |
| **Error al guardar deja usuario en estado inconsistente** | Error durante guardado puede dejar algunos cambios aplicados y otros no (transacción parcial), causando estado inconsistente del usuario | Alto | Muy Baja | Usar transacciones de base de datos que garantizan atomicidad: o se aplican TODOS los cambios o NINGUNO. Si falla, rollback completo. Validar integridad post-guardado. Logs detallados para debugging | Mitigado |
| **Modificación masiva de permisos sin revisión** | Administrador agrega 50 permisos nuevos sin revisión cuidadosa y otorga accesos indebidos, comprometiendo seguridad | Alto | Media | Pantalla de resumen OBLIGATORIA antes de confirmar. Resumen muestra TODOS los cambios claramente organizados. Límite razonable de permisos agregados en una sola edición (ej: máx 20). Advertencia si se excede umbral | Mitigado con resumen |
| **No se puede identificar quién eliminó permiso crítico** | Usuario pierde acceso crítico, no se sabe quién eliminó el permiso ni por qué, causando problemas operativos y de auditoría | Alto | Baja | Auditoría detallada de CADA eliminación de permiso registrando: admin que eliminó, fecha/hora, permiso eliminado, quién lo asignó originalmente, cuándo. Historial completo visible en detalle de usuario | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance (Out of Scope)
- Inactivación o eliminación permanente de usuarios (se cubrirá en HU009 - Inactivación y Eliminación de Usuarios)
- Reseteo de contraseña del usuario (se cubre en HU007 - Contraseña Temporal y HU002 - Recuperación de Contraseña)
- Edición masiva de múltiples usuarios simultáneamente (se evaluará en Release 2)
- Asignación masiva de permisos desde archivo CSV (se evaluará en Release 2)
- Configuración de permisos granulares a nivel de funcionalidad específica (se cubre en módulo de Permisos Avanzados)
- Delegación de capacidad de edición a Administrador de Cliente para sus propios usuarios (se evaluará futuro)
- Notificación automática al usuario cuando se modifican sus permisos (se evaluará futuro)
- Proceso de aprobación para cambios críticos (workflow de aprobación - futuro)

### Dependencias
- **HU005** - Visualización y Búsqueda de Usuarios: proporciona punto de acceso a edición
- **HU006** - Creación de Usuarios: comparte validaciones, estructura de permisos y lógica de asignación
- **HU007** - Contraseña Temporal: si admin resetea contraseña desde aquí (fuera de scope pero relacionado)
- **Tabla maestra Productos.csv**: para validar roles según productos contratados
- **Tabla maestra Roles.csv**: para mostrar roles disponibles
- **Glosario**: Usuario Interno, Usuario de Cliente, Permiso, Rol, Estado de Usuario

### Referencias
- **Estándar de Auditoría Funcional**: Tipos `ADMINISTRACION_USUARIO_*`, un registro por campo modificado + un registro por permiso agregado/eliminado
- **Glosario**: Usuario, Permiso, Rol, Estado de Usuario, Asignación de Permisos
- **APL03 Seguridad**: Auditoría de cambios, control de acceso, trazabilidad

---

