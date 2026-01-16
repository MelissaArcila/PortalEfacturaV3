# HU006 - Creación de Usuarios y Asignación de Permisos

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

El Portal Unificado de CDN Facturación requiere la capacidad de registrar nuevos Usuarios en el sistema con sus respectivos permisos de acceso. Los Usuarios pueden ser de dos tipos fundamentales: **Usuarios Internos** (personal de CDN Facturación que opera y soporta la plataforma) y **Usuarios de Cliente** (personal de empresas contratantes que utiliza los servicios de facturación electrónica).

La complejidad de la gestión de usuarios radica en que un mismo Usuario puede tener acceso a múltiples Empresas (Clientes) con diferentes Roles en cada una. Por ejemplo, un Usuario puede ser Gestor Emisión FE en la Empresa A, Administrador de Cliente en la Empresa B, y Gestor RADIAN en la Empresa C, todo simultáneamente. Además, un Usuario creado inicialmente como Interno puede recibir posteriormente permisos de Cliente para dar soporte directo a empresas específicas.

Los Administradores del Portal requieren una interfaz flexible que permita crear Usuarios de forma guiada, validar la unicidad del Número de Identificación, asignar múltiples permisos (combinaciones de Cliente + Rol) de forma dinámica, y mantener trazabilidad completa de quién creó el Usuario y quién asignó cada permiso para cumplir con requisitos de auditoría y seguridad.

---

**Notas:**
- Solo Usuarios con rol de Administrador del Portal pueden crear Usuarios
- El Número de Identificación es único en todo el sistema - no se permiten duplicados
- El Correo Electrónico es único en todo el sistema - no se permiten duplicados (se usa para autenticación)
- Un Usuario puede ser Interno, de Cliente, o ambos (Interno con permisos adicionales de Cliente para soporte)
- Los permisos se asignan como combinaciones de Cliente + Rol
- Un Usuario debe tener al menos un permiso (rol) asignado antes de poder crearse
- Los Roles disponibles para un Cliente dependen de los Productos que tenga contratados
- Todo el proceso debe ser auditable: creación del Usuario y cada asignación de permiso
- Si ocurre error durante creación, el Usuario no debe perder los datos ingresados

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Administrador del Portal** | Quiero poder crear nuevos Usuarios indicando si son Internos o de Cliente | Para registrar tanto personal de CDN como personal de empresas contratantes en el sistema |
| Como **Administrador del Portal** | Quiero completar un formulario con datos personales del Usuario (nombres, apellidos, identificación, correo) | Para capturar la información necesaria para identificar y contactar al Usuario |
| Como **Administrador del Portal** | Quiero que el sistema valide que el Número de Identificación y el Correo Electrónico son únicos | Para prevenir duplicación de Usuarios y mantener la integridad de los datos y credenciales de autenticación |
| Como **Administrador del Portal** | Quiero asignar múltiples permisos a un Usuario seleccionando combinaciones de Cliente y Rol | Para configurar el acceso del Usuario a diferentes empresas con roles específicos según sus responsabilidades |
| Como **Administrador del Portal** | Quiero ver en una tabla los permisos que voy asignando y poder eliminarlos si me equivoco | Para tener control completo sobre los permisos antes de confirmar la creación del Usuario |
| Como **Administrador del Portal** | Quiero que los Roles disponibles se filtren según los Productos que tiene contratados cada Cliente | Para asignar solo Roles válidos y evitar errores de configuración |
| Como **Administrador del Portal** | Quiero poder crear un Usuario Interno y luego asignarle también permisos de Cliente | Para que personal de soporte pueda acceder a empresas específicas para dar asistencia |
| Como **Administrador del Portal** | Quiero revisar un resumen completo antes de confirmar la creación | Para verificar que toda la información y permisos son correctos antes de registrar al Usuario |
| Como **Auditor** | Quiero que se registre quién creó cada Usuario y quién asignó cada permiso con fecha y hora | Para garantizar trazabilidad completa de la gestión de accesos y cumplir con políticas de auditoría |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Acceso a creación de Usuario | Administrador del Portal en pantalla de Gestión de Usuarios (HU005) | Administrador hace clic en "Crear Nuevo Usuario" | Sistema navega a pantalla de "Crear Usuario". Muestra formulario estructurado en secciones: "Tipo de Usuario", "Datos Personales", "Permisos". Todos los campos obligatorios marcados con asterisco (*) | ☐ | ☐ | ☐ |
| 2 | Selección de Tipo de Usuario - Usuario de Cliente | Administrador en formulario de creación | Administrador selecciona opción "Usuario de Cliente" en sección "Tipo de Usuario" | Sistema muestra radio button seleccionado. Habilita sección de "Permisos" con instrucción: "Asigne al menos un permiso seleccionando Cliente y Rol". Campo "Selección de Empresa" se vuelve visible y obligatorio para asignación de permisos | ☐ | ☐ | ☐ |
| 3 | Selección de Tipo de Usuario - Usuario Interno | Administrador en formulario de creación | Administrador selecciona opción "Usuario Interno" en sección "Tipo de Usuario" | Sistema muestra radio button seleccionado. Habilita sección de "Permisos" con instrucción: "Asigne roles internos o roles de cliente para soporte". Campo "Selección de Empresa" en permisos muestra opción "Sin Cliente (Rol Interno)" además de la lista de empresas | ☐ | ☐ | ☐ |
| 4 | Validación de Número de Identificación - solo numérico | Administrador ingresa valor en campo "Número de Identificación" | Administrador escribe caracteres no numéricos | Sistema NO permite ingresar caracteres no numéricos. Solo acepta dígitos del 0 al 9. Máximo 15 caracteres. Muestra hint text: "Solo números, máximo 15 dígitos" | ☐ | ☐ | ☐ |
| 5 | Validación de unicidad de Número de Identificación | Administrador completa campo "Número de Identificación" con un valor que ya existe en el sistema | Administrador hace clic fuera del campo o presiona Enter | Sistema valida en tiempo real contra la base de datos. Muestra mensaje de error: "Este número de identificación ya está registrado en el sistema. Usuario existente: [Nombre Completo del usuario existente]". Campo se marca con error visual (borde rojo). No permite continuar hasta corregir | ☐ | ☐ | ☐ |
| 6 | Número de Identificación único y válido | Administrador ingresa Número de Identificación que NO existe en el sistema | Administrador hace clic fuera del campo o presiona Enter | Sistema valida y confirma que el número es único. Muestra indicador de validación exitosa (checkmark verde). Permite continuar con el formulario | ☐ | ☐ | ☐ |
| 7 | Ingreso de Primer Apellido obligatorio | Administrador ingresa texto en campo "Primer Apellido" | Administrador completa el campo | Sistema acepta letras (A-Z, a-z), espacios, guiones, apóstrofes. NO acepta números ni símbolos especiales. Campo es obligatorio (*Primer Apellido). Máximo 50 caracteres | ☐ | ☐ | ☐ |
| 8 | Ingreso de Segundo Apellido opcional | Administrador ingresa o deja vacío campo "Segundo Apellido" | Administrador interactúa con el campo | Sistema acepta letras, espacios, guiones, apóstrofes. NO acepta números ni símbolos especiales. Campo es OPCIONAL (Segundo Apellido). Máximo 50 caracteres. Si se deja vacío, se acepta sin error | ☐ | ☐ | ☐ |
| 9 | Ingreso de Primer Nombre obligatorio | Administrador ingresa texto en campo "Primer Nombre" | Administrador completa el campo | Sistema acepta letras, espacios, guiones, apóstrofes. NO acepta números ni símbolos especiales. Campo es obligatorio (*Primer Nombre). Máximo 50 caracteres | ☐ | ☐ | ☐ |
| 10 | Ingreso de Segundo Nombre opcional | Administrador ingresa o deja vacío campo "Segundo Nombre" | Administrador interactúa con el campo | Sistema acepta letras, espacios, guiones, apóstrofes. Campo es OPCIONAL (Segundo Nombre). Máximo 50 caracteres | ☐ | ☐ | ☐ |
| 11 | Validación de formato de Correo Electrónico | Administrador ingresa texto en campo "Correo Electrónico" | Administrador completa el campo y hace clic fuera | Sistema valida formato de correo electrónico usando regex estándar (debe contener @, dominio válido, sin espacios). Campo es OBLIGATORIO (*Correo Electrónico). Si formato es válido, muestra checkmark verde. Si formato es inválido, muestra error: "Ingrese un correo electrónico válido (ejemplo: usuario@dominio.com)" | ☐ | ☐ | ☐ |
| 11a | Validación de unicidad de Correo Electrónico | Administrador completa campo "Correo Electrónico" con un valor que ya existe en el sistema | Administrador hace clic fuera del campo o presiona Enter | Sistema valida en tiempo real contra la base de datos. Muestra mensaje de error: "Este correo electrónico ya está registrado en el sistema. Usuario existente: [Nombre Completo del usuario existente]". Campo se marca con error visual (borde rojo). No permite continuar hasta corregir | ☐ | ☐ | ☐ |
| 11b | Correo Electrónico único y válido | Administrador ingresa Correo Electrónico que NO existe en el sistema y tiene formato válido | Administrador hace clic fuera del campo o presiona Enter | Sistema valida formato y confirma que el correo es único. Muestra indicador de validación exitosa (checkmark verde). Permite continuar con el formulario | ☐ | ☐ | ☐ |
| 12 | Estado inicial del Usuario | Formulario de creación se carga | Sistema inicializa campo Estado | Campo "Estado" se muestra como información (no editable en creación). Valor predeterminado: "Activo" (badge verde). Texto informativo: "El usuario se creará en estado Activo por defecto. Puede modificarse después de la creación." | ☐ | ☐ | ☐ |
| 13 | Selección de Empresa para asignar permiso | Administrador en sección "Permisos", selecciona Tipo de Usuario "Usuario de Cliente" | Administrador hace clic en campo "Selección de Empresa" | Sistema muestra dropdown con lista de todas las Empresas/Clientes registrados y activos (ordenados alfabéticamente por nombre). Dropdown permite búsqueda por nombre de empresa (typeahead). NO muestra opción "Sin Cliente (Rol Interno)" si Tipo es "Usuario de Cliente" | ☐ | ☐ | ☐ |
| 14 | Selección de Empresa para Usuario Interno | Administrador selecciona Tipo "Usuario Interno", hace clic en campo "Selección de Empresa" | Administrador abre dropdown | Sistema muestra dropdown con: primera opción "Sin Cliente (Rol Interno)" + lista de Empresas/Clientes activos. Permite búsqueda. Usuario Interno puede seleccionar "Sin Cliente" para asignar roles internos O seleccionar una Empresa para asignar roles de cliente (soporte) | ☐ | ☐ | ☐ |
| 15 | Selección de Rol según Empresa seleccionada | Administrador selecciona una Empresa específica en "Selección de Empresa" | Sistema carga Roles disponibles | Sistema muestra dropdown "Selección de Rol" con Roles disponibles para esa Empresa. Si la Empresa tiene contratados Productos 1, 2, 7 (Emisión FE, Emisión POS, RADIAN), muestra: "Administrador de Cliente", "Gestor Emisión FE", "Gestor Emisión POS", "Gestor RADIAN". Solo muestra Roles de Gestores para Productos contratados. Dropdown permite búsqueda | ☐ | ☐ | ☐ |
| 16 | Selección de Rol Interno sin Cliente | Administrador selecciona "Sin Cliente (Rol Interno)" en Empresa | Sistema carga Roles Internos | Sistema muestra dropdown "Selección de Rol" con Roles Internos: "Administrador de Portal", "Analista Interno", "Soporte Técnico", "Auditor Interno", "Desarrollador", "Consultor Funcional". Solo muestra Roles con Aplica_A = "INTERNO" de tabla maestra Roles | ☐ | ☐ | ☐ |
| 17 | Agregar permiso a la tabla | Administrador selecciona Empresa + Rol válidos, hace clic en "Agregar Permiso" | Sistema valida selección | Sistema valida que: (1) Empresa está seleccionada, (2) Rol está seleccionado, (3) La combinación Empresa+Rol NO existe ya en la tabla de permisos agregados. Si válido, agrega fila a tabla visible con columnas: Empresa (nombre o "Interno"), Rol (nombre), Acción (botón Eliminar). Limpia selectores de Empresa y Rol para permitir agregar otro. Si inválido, muestra error correspondiente | ☐ | ☐ | ☐ |
| 18 | Prevención de permisos duplicados | Administrador intenta agregar permiso que ya existe (misma Empresa + mismo Rol) | Administrador hace clic en "Agregar Permiso" | Sistema valida y detecta duplicado. Muestra mensaje de error: "Este permiso ya fue agregado. El usuario ya tiene el rol [Nombre Rol] en [Nombre Empresa]". No agrega el permiso duplicado a la tabla | ☐ | ☐ | ☐ |
| 19 | Visualización de permisos agregados en tabla | Administrador ha agregado 3 permisos | Sistema actualiza tabla | Sistema muestra tabla de "Permisos Asignados" con 3 filas. Cada fila muestra: **(1) Empresa**: "Empresa ABC" o "Interno" si rol interno, **(2) Rol**: "Gestor Emisión FE", **(3) Acción**: botón "Eliminar" (icono basurero o texto rojo). Tabla es paginada si hay más de 10 permisos (10 por página) | ☐ | ☐ | ☐ |
| 20 | Eliminar permiso de la tabla con confirmación | Administrador hace clic en botón "Eliminar" junto a un permiso en la tabla | Sistema detecta clic | Sistema muestra diálogo de confirmación: "¿Está seguro que desea eliminar el permiso [Rol] en [Empresa]?". Botones: "Cancelar" y "Confirmar". Si Administrador confirma, elimina la fila de la tabla. Si cancela, cierra el diálogo sin eliminar | ☐ | ☐ | ☐ |
| 21 | Filtros en tabla de permisos agregados | Administrador ha agregado 15 permisos de diferentes empresas | Administrador usa filtros | Sistema muestra filtros encima de la tabla: "Filtrar por Empresa" (dropdown con empresas agregadas), "Filtrar por Rol" (dropdown con roles agregados). Al seleccionar filtro, muestra solo filas que coinciden. Botón "Limpiar Filtros" visible | ☐ | ☐ | ☐ |
| 22 | Validación de al menos un permiso antes de crear | Administrador completa todos los campos personales pero NO agrega ningún permiso | Administrador intenta hacer clic en "Crear Usuario" | Sistema valida y muestra error: "Debe asignar al menos un permiso antes de crear el usuario. Agregue combinaciones de Cliente + Rol en la sección Permisos." Botón "Crear Usuario" permanece deshabilitado hasta que haya al menos 1 permiso en la tabla | ☐ | ☐ | ☐ |
| 23 | Validación de campos obligatorios completos | Administrador ha completado algunos campos pero NO todos los obligatorios | Administrador intenta avanzar | Sistema valida TODOS los campos obligatorios: *Número de Identificación (único), *Primer Apellido, *Primer Nombre, *Correo Electrónico (único y formato válido), Al menos 1 permiso asignado. Si falta alguno, muestra mensaje: "Complete todos los campos obligatorios (*) antes de continuar" y hace scroll al primer campo faltante | ☐ | ☐ | ☐ |
| 24 | Botón Crear Usuario habilitado con datos completos | Administrador completa todos los campos obligatorios y agrega al menos 1 permiso | Sistema valida en tiempo real | Botón "Crear Usuario" se habilita (color primario, gradiente, clickeable). Antes de completar requisitos, botón permanece deshabilitado (gris, cursor not-allowed) | ☐ | ☐ | ☐ |
| 25 | Pantalla de resumen antes de crear | Administrador completa formulario y hace clic en "Crear Usuario" | Sistema valida y muestra resumen | Sistema muestra pantalla de resumen con secciones: **(1) Tipo de Usuario**: Interno / Cliente / Interno con permisos de Cliente, **(2) Datos Personales**: Número ID, Nombres completos, Apellidos completos, Correo, Estado, **(3) Permisos Asignados**: Tabla completa con todas las combinaciones Empresa + Rol agregadas. Botones: "Volver y Editar", "Confirmar Creación" | ☐ | ☐ | ☐ |
| 26 | Volver a editar desde resumen | Administrador en pantalla de resumen hace clic en "Volver y Editar" | Sistema detecta clic | Sistema regresa al formulario de creación manteniendo TODOS los datos ingresados: tipo de usuario, datos personales, permisos agregados en la tabla. Ningún dato se pierde. Administrador puede modificar y volver a intentar crear | ☐ | ☐ | ☐ |
| 27 | Creación exitosa de Usuario | Administrador en pantalla de resumen hace clic en "Confirmar Creación" | Sistema procesa la creación | Sistema guarda Usuario en base de datos con todos los datos, asigna todos los permisos de la tabla, genera ID único del Usuario, registra fecha/hora de creación, usuario creador. Muestra mensaje de éxito: "¡Usuario creado exitosamente! El usuario [Nombre Completo] con identificación [Número ID] ha sido registrado con [X] permisos asignados." Botones: "Ver Usuario" (navega a detalle), "Crear Otro Usuario" (limpia formulario), "Volver a Gestión de Usuarios" | ☐ | ☐ | ☐ |
| 28 | Error durante creación - mensaje genérico | Administrador confirma creación pero ocurre error técnico (timeout, error BD, validación backend fallida) | Sistema intenta guardar pero falla | Sistema muestra mensaje de error genérico: "Ocurrió un error al crear el usuario. Por favor, intente nuevamente. Si el problema persiste, contacte a soporte técnico." Botón "Aceptar" disponible. Al hacer clic, sistema regresa al formulario manteniendo TODOS los datos. Administrador puede revisar y volver a intentar | ☐ | ☐ | ☐ |
| 29 | Usuario Interno con permisos de Cliente | Administrador selecciona Tipo "Usuario Interno", agrega rol interno "Soporte Técnico", luego agrega rol "Gestor Emisión FE" en "Empresa ABC" | Administrador crea usuario | Sistema permite la combinación. Tabla de permisos muestra: Fila 1: "Interno" - "Soporte Técnico", Fila 2: "Empresa ABC" - "Gestor Emisión FE". En resumen muestra Tipo: "Usuario Interno con permisos de Cliente (Soporte)". Permite crear sin error | ☐ | ☐ | ☐ |
| 30 | Usuario de Cliente sin permisos internos | Administrador selecciona Tipo "Usuario de Cliente", intenta agregar rol interno sin seleccionar empresa | Administrador intenta agregar | Sistema NO permite. Si Tipo es "Usuario de Cliente", el dropdown de Empresa NO muestra opción "Sin Cliente (Rol Interno)". Si intenta manipular, backend rechaza. Usuario de Cliente SOLO puede tener roles de cliente en empresas | ☐ | ☐ | ☐ |
| 31 | Múltiples permisos en diferentes empresas | Administrador agrega permisos: "Empresa A - Gestor FE", "Empresa B - Admin Cliente", "Empresa C - Gestor RADIAN", "Empresa A - Gestor POS" | Administrador crea usuario | Sistema permite agregar múltiples permisos en diferentes empresas Y múltiples roles en la misma empresa (Empresa A tiene 2 roles). Tabla muestra 4 filas. Validación previene duplicar exactamente la misma combinación Empresa+Rol | ☐ | ☐ | ☐ |
| 32 | Cancelar creación de Usuario | Administrador en medio del formulario decide cancelar | Administrador hace clic en botón "Cancelar" | Sistema muestra diálogo de confirmación: "¿Está seguro que desea cancelar? Se perderán todos los datos ingresados." Botones: "Continuar Editando", "Sí, Cancelar". Si confirma, limpia formulario y regresa a Gestión de Usuarios. Si continúa editando, cierra diálogo y mantiene datos | ☐ | ☐ | ☐ |
| 33 | Validación de permiso de acceso | Usuario sin rol de Administrador del Portal intenta acceder a creación de usuarios mediante URL directa | Usuario no autorizado navega a URL | Sistema valida permisos. Muestra mensaje: "No tiene permisos para acceder a esta sección. Solo Administradores del Portal pueden crear usuarios." y redirige a pantalla de inicio | ☐ | ☐ | ☐ |
| 34 | Empresa inactiva no aparece en selector | Existe una Empresa en el sistema pero está en estado Inactivo | Administrador abre dropdown "Selección de Empresa" | Sistema NO muestra empresas inactivas en el selector. Solo muestra Empresas con estado Activo. Previene asignar permisos a empresas que no están operativas | ☐ | ☐ | ☐ |
| 35 | Empresa sin productos contratados | Empresa activa pero sin ningún producto contratado | Administrador selecciona esa Empresa en dropdown | Sistema muestra dropdown "Selección de Rol" con solo: "Administrador de Cliente". NO muestra roles de gestor porque no hay productos contratados. Muestra mensaje informativo: "Esta empresa no tiene productos contratados. Solo puede asignar rol Administrador de Cliente" | ☐ | ☐ | ☐ |
| 36 | Autoguardado temporal opcional | Administrador ha llenado 50% del formulario | Cada 30 segundos de inactividad | Sistema opcionalmente guarda datos del formulario en localStorage del navegador como respaldo. Si navegador se cierra accidentalmente y se vuelve a abrir, muestra mensaje: "Detectamos que estaba creando un usuario. ¿Desea recuperar los datos?" con botones "Recuperar" y "Empezar Nuevo". Útil para prevenir pérdida de datos por errores | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 37 | Estructura obligatoria de registro de auditoría | Cualquier evento relacionado con creación de usuarios (creación exitosa, intento fallido, asignación de permiso, validación de ID duplicado) | Sistema registra evento | Registro contiene: ID Único del Evento (UUID), Tipo de Evento (MODULO_ENTIDAD_ACCION), Fecha y Hora (UTC milisegundos ISO 8601), Usuario (Administrador creador), Cliente (NULL), Cliente (Nombre - NULL), IP Local, IP Pública, Resultado (EXITOSO/FALLIDO), Descripción, Nivel de Severidad (INFO/WARNING/ERROR), Datos Adicionales (JSON: ID usuario creado, permisos asignados, etc.) | ☐ | ☐ | ☐ |
| 38 | Auditoría de creación exitosa de Usuario | Administrador confirma creación y proceso es exitoso | Sistema guarda Usuario y permisos | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_USUARIO_CREACION_EXITOSA", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Administrador {usuario} creó usuario {nombre_completo} con correo {correo_electronico}", Datos Adicionales: {"usuario_creado_id": "UUID", "numero_identificacion": "123456789", "nombre_completo": "Juan Pérez Gómez", "correo_electronico": "juan@empresa.com", "tipo_usuario": "Usuario de Cliente", "estado": "Activo", "permisos_asignados_count": 3} | ☐ | ☐ | ☐ |
| 38a | Auditoría de intento fallido por correo duplicado | Administrador ingresa Correo Electrónico existente | Administrador hace blur/enter en campo | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_USUARIO_VALIDACION_CORREO_DUPLICADO", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Administrador {usuario} intentó crear usuario con correo electrónico duplicado {correo}", Datos Adicionales: {"correo_electronico": "usuario@ejemplo.com", "usuario_existente_id": "UUID", "usuario_existente_nombre": "Usuario Existente Nombre"} | ☐ | ☐ | ☐ |
| 39 | Auditoría de cada permiso asignado | Administrador crea Usuario con 3 permisos (A-Rol1, B-Rol2, C-Rol3) | Sistema guarda permisos | Sistema genera 3 registros de auditoría separados (uno por cada permiso): Tipo de Evento: "ADMINISTRACION_USUARIO_PERMISO_ASIGNADO", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Administrador {admin} asignó permiso {rol} en {empresa} a usuario {usuario_id}", Datos Adicionales: {"usuario_id": "UUID", "empresa_id": "UUID", "empresa_nombre": "Empresa ABC", "rol_id": "3", "rol_nombre": "Gestor Emisión FE", "fecha_asignacion": "timestamp"} | ☐ | ☐ | ☐ |
| 40 | Auditoría de intento fallido por ID duplicado | Administrador ingresa Número de Identificación existente | Administrador hace blur/enter en campo | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_USUARIO_VALIDACION_ID_DUPLICADO", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Administrador {usuario} intentó crear usuario con número de identificación duplicado {numero_id}", Datos Adicionales: {"numero_identificacion": "123456789", "usuario_existente_id": "UUID", "usuario_existente_nombre": "Usuario Existente Nombre"} | ☐ | ☐ | ☐ |
| 41 | Auditoría de error técnico durante creación | Administrador confirma creación pero error técnico (timeout, BD error) | Sistema intenta guardar pero falla | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_USUARIO_CREACION_ERROR", Resultado: "FALLIDO", Severidad: "ERROR", Descripción: "Error al crear usuario {nombre_completo} por Administrador {usuario}", Datos Adicionales: {"numero_identificacion": "123456789", "nombre_completo": "Juan Pérez", "error_tipo": "timeout", "error_mensaje": "Mensaje técnico", "datos_formulario": "JSON para debugging"} | ☐ | ☐ | ☐ |
| 42 | Auditoría de cancelación de creación | Administrador cancela proceso después de ingresar datos | Administrador confirma cancelación | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_USUARIO_CREACION_CANCELADA", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Administrador {usuario} canceló creación de usuario", Datos Adicionales: {"numero_identificacion_parcial": "123456789", "nombre_parcial": "Juan Pérez", "permisos_agregados_count": 2, "porcentaje_completado": "estimado"} | ☐ | ☐ | ☐ |
| 43 | Auditoría de acceso no autorizado | Usuario sin rol Admin intenta acceder a creación | Usuario no autorizado navega a URL | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_USUARIO_ACCESO_DENEGADO", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} sin permisos intentó acceder a creación de usuarios", Datos Adicionales: {"rol_usuario": "Gestor FE", "url_intentada": "/admin/usuarios/crear"} | ☐ | ☐ | ☐ |
| 44 | Inmutabilidad de registros de auditoría | Cualquier escenario de auditoría | Sistema genera registro | Registros NO pueden ser modificados ni eliminados. Solo consultas de lectura | ☐ | ☐ | ☐ |
| 45 | Consulta de registros de creación de usuarios | Auditor requiere revisar historial de creaciones | Auditor accede a módulo de auditoría | Sistema permite filtrar por: fechas, usuario creador, tipo de evento (ADMINISTRACION_USUARIO_*), resultado, número de identificación del usuario creado. Permite exportar CSV/Excel. Muestra: fecha/hora, admin creador, usuario creado, permisos asignados, resultado | ☐ | ☐ | ☐ |
| 46 | Retención de registros de auditoría | Registros de creación de usuarios | Paso del tiempo | Registros se conservan por mínimo 7 años. Después pueden archivarse o eliminarse según política | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

Se requieren los siguientes mockups interactivos en HTML siguiendo la Guía de Estilos 

#### 1. **HU006-CrearUsuario.html** - Formulario de creación de usuario
**Elementos:**

**Sección 1: Tipo de Usuario**
- Radio buttons: "Usuario de Cliente" / "Usuario Interno"
- Texto explicativo según selección

**Sección 2: Datos Personales**
- *Número de Identificación (input numérico, max 15, validación unicidad en blur/enter con checkmark verde o error rojo mostrando usuario existente)
- *Primer Apellido (input texto, max 50, solo letras/espacios/guiones)
- Segundo Apellido (input texto, max 50, opcional)
- *Primer Nombre (input texto, max 50, solo letras/espacios/guiones)
- Segundo Nombre (input texto, max 50, opcional)
- *Correo Electrónico (input email, validación formato y unicidad en blur/enter con checkmark verde o error rojo mostrando usuario existente)
- Estado (badge "Activo" verde, no editable, texto informativo)

**Sección 3: Permisos**
- Título: "Asignación de Permisos"
- Texto: "Asigne al menos un permiso seleccionando Cliente y Rol"
- Formulario de permiso:
  - Selección de Empresa (dropdown con búsqueda, opciones: "Sin Cliente (Rol Interno)" si tipo=Interno + lista de empresas activas)
  - Selección de Rol (dropdown con búsqueda, opciones dinámicas según empresa seleccionada y productos contratados)
  - Botón "Agregar Permiso" (outlined, ícono +)
- Tabla "Permisos Asignados":
  - Columnas: Empresa (nombre o "Interno"), Rol (nombre), Acción (botón Eliminar)
  - Paginación si >10 registros
  - Filtros: por Empresa, por Rol
  - Mensaje si vacía: "No hay permisos asignados. Agregue al menos uno para continuar"

**Botones del formulario:**
- "Cancelar" (outlined, confirmación)
- "Crear Usuario" (contained gradiente, deshabilitado hasta completar obligatorios + >=1 permiso)

#### 2. **HU006-ResumenUsuario.html** - Resumen antes de confirmar
**Elementos:**
- Título: "Resumen del Nuevo Usuario"
- Secciones:
  - **Tipo de Usuario**: Badge con "Usuario de Cliente" / "Usuario Interno" / "Usuario Interno con permisos de Cliente"
  - **Datos Personales**: Lista de datos (ID, Nombres, Apellidos, Correo, Estado)
  - **Permisos Asignados**: Tabla completa con todas combinaciones Empresa + Rol
- Botones:
  - "Volver y Editar" (outlined)
  - "Confirmar Creación" (contained gradiente Primary → Secondary)

#### 3. **HU006-CreacionExitosa.html** - Confirmación exitosa
**Elementos:**
- Icono de éxito (checkmark grande verde)
- Título: "¡Usuario Creado Exitosamente!"
- Mensaje: "El usuario [Nombre Completo] con identificación [Número ID] ha sido registrado con [X] permisos asignados."
- Resumen breve de permisos
- Botones:
  - "Ver Usuario" (navega a detalle)
  - "Crear Otro Usuario" (limpia formulario)
  - "Volver a Gestión de Usuarios"

#### 4. **HU006-ErrorCreacion.html** - Error durante creación
**Elementos:**
- Icono de error (warning naranja/rojo)
- Título: "Error al Crear Usuario"
- Mensaje: "Ocurrió un error al crear el usuario. Por favor, intente nuevamente. Si el problema persiste, contacte a soporte técnico."
- Botón "Aceptar" (regresa al formulario manteniendo datos)

#### 5. **HU006-DialogoConfirmacionEliminarPermiso.html** - Confirmación eliminar permiso
**Elementos:**
- Título: "¿Eliminar Permiso?"
- Mensaje: "¿Está seguro que desea eliminar el permiso [Rol] en [Empresa]?"
- Botones: "Cancelar" (text), "Confirmar" (contained error)

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Pérdida de datos si navegador se cierra accidentalmente** | Si Administrador cierra navegador o pestaña después de llenar formulario complejo con 10+ permisos, pierde todo el trabajo, causando frustración y re-trabajo | Medio | Media | Implementar autoguardado temporal en localStorage cada 30 segundos. Al volver a abrir, mostrar diálogo "¿Recuperar datos?". Mantener datos después de errores de creación. Advertir antes de cerrar pestaña con `beforeunload` | Mitigado |
| **Confusión entre Usuario Interno y Usuario de Cliente** | Administrador puede no entender claramente la diferencia y crear usuarios con tipo incorrecto, causando problemas de permisos y acceso | Medio | Media | Texto explicativo claro bajo cada opción de radio button. Ejemplo: "Usuario Interno: Personal de CDN Facturación. Usuario de Cliente: Personal de empresa contratante". Validar en resumen y mostrar tipo claramente antes de confirmar | Mitigado |
| **Asignación masiva de permisos erróneos** | Administrador asigna 50 permisos a un usuario equivocado y no lo nota hasta después de crear, requiriendo edición manual posterior | Bajo | Baja | Pantalla de resumen clara mostrando TODOS los permisos antes de confirmar. Permitir volver y editar fácilmente. Implementar límite razonable de permisos por usuario (ej: máx 50) con advertencia si se aproxima | Aceptado con resumen |
| **Usuario con productos no contratados** | Si empresa cambia sus productos contratados DESPUÉS de que se creó el usuario, el usuario puede tener roles para productos que ya no existen, causando confusión | Bajo | Baja | Validar permisos contra productos activos en cada login. Mostrar advertencia en perfil de usuario si tiene permisos huérfanos. Proceso de sincronización periódica que inactiva permisos de productos cancelados | Aceptado |
| **Duplicación de usuarios por validación fallida** | Si validación de unicidad tiene condición de carrera, dos admins podrían crear usuarios con mismo ID simultáneamente | Alto | Muy Baja | Validación en 3 niveles: (1) Frontend en blur, (2) Backend al recibir, (3) Constraint UNIQUE en BD. Usar transacciones para garantizar atomicidad. Manejo de error de constraint mostrando mensaje claro | Mitigado |
| **Explosión de registros de auditoría** | Si un usuario tiene 100 permisos, se generan 101 registros de auditoría (1 creación + 100 asignaciones), pudiendo llenar tabla de auditoría rápidamente | Bajo | Baja | Implementar límite razonable de permisos por usuario (50 máximo). Optimizar almacenamiento de auditoría con particionamiento por fecha. Política de archivo de registros antiguos >1 año. Índices en tabla de auditoría | Aceptado con límite |
| **Rol no válido para producto** | Si se modifica tabla maestra de Roles mientras Administrador está llenando formulario, podría mostrar roles obsoletos | Bajo | Muy Baja | Cargar lista de roles dinámicamente cada vez que se selecciona empresa. Validar en backend que rol es válido para productos de la empresa antes de guardar. Mostrar error claro si rol ya no existe | Mitigado |
| **Permisos complejos dificultan auditoría** | Usuario con 30+ permisos en 15 empresas diferentes dificulta rastrear quién, cuándo y por qué se asignó cada permiso | Medio | Baja | Registrar en auditoría cada asignación individual con timestamp y admin. Pantalla de detalle de usuario muestra historial completo de cambios. Permitir filtrar auditoría por usuario específico. Reporte de permisos agregables | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance (Out of Scope)
- Edición de usuarios existentes (se cubrirá en HU007 - Edición de Usuarios)
- Eliminación de usuarios (se cubrirá en HU008 - Inactivación y Eliminación de Usuarios)
- Asignación masiva de permisos desde archivo CSV (se evaluará en Release 2)
- Importación de usuarios desde Active Directory o LDAP (se evaluará en Release 2)
- Generación automática de contraseña temporal y envío por correo (se cubre en HU009 - Gestión de Contraseñas)
- Configuración de permisos granulares a nivel de funcionalidad (se cubre en módulo de Permisos Avanzados)
- Roles personalizados creados por Administrador de Cliente (se evaluará futuro)



### Referencias
- **Estándar de Auditoría Funcional**: Tipos `ADMINISTRACION_USUARIO_*`, un registro por creación + un registro por cada permiso asignado
- **Glosario**: Usuario Interno, Usuario de Cliente, Permiso, Rol, Gestor, Administrador de Portal, Administrador de Cliente, Producto, Asignación de Permisos

