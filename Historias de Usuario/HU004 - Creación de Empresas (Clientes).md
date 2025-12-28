# HU004 - Creación de Empresas (Clientes)

## Información General

| Campo | Descripción |
|-------|-------------|
| **Release objetivo** | Release 1 |
| **Epic** | Gestión de Clientes y Administración |
| **Estado** | BORRADOR |
| **Propietario** | Por definir |
| **Arquitecto** | Por definir |
| **Desarrolladores** | Por definir |
| **Tester** | Por definir |
| **Incidentes relacionados** | N/A |
| **Arquitectura relacionada** | Portal Unificado - Módulo de Administración, Gestión de Clientes, Catálogos Maestros DIAN |

---

## Contexto

El Portal Unificado de CDN Facturación requiere la capacidad de registrar nuevas Empresas (Clientes) en el sistema para que puedan acceder a los servicios del ecosistema de facturación electrónica DIAN. Una Empresa representa una entidad comercial (persona natural o jurídica) que contrata los productos y servicios de facturación electrónica, Documento Soporte, RADIAN, y otros módulos del ecosistema.

Actualmente no existe un mecanismo centralizado y estructurado para registrar nuevas Empresas con toda la información requerida por la normativa DIAN, incluyendo datos fiscales, ubicación geográfica, registros mercantiles, y configuración tributaria. Los Administradores del Portal necesitan poder crear Empresas de forma guiada, con validaciones en tiempo real que aseguren la integridad y unicidad de la información, especialmente del Número de Identificación Tributaria (NIT) y Dígito de Verificación (DV).

La creación de Empresas debe seguir la estructura de datos exigida por la DIAN para facturación electrónica, validar correctamente el dígito de verificación según el algoritmo oficial, permitir registro de múltiples domicilios fiscales (registros mercantiles), y capturar información completa de ubicación geográfica con soporte para clientes internacionales fuera de Colombia.

---

**Notas:**
- Solo Usuarios con rol de Administrador del Portal pueden crear Empresas
- El Número de Identificación es único en el sistema - no se permite duplicación
- El Dígito de Verificación (DV) se valida usando el algoritmo oficial de la DIAN
- Una Empresa puede tener uno o más Registros Mercantiles (domicilios fiscales)
- Los campos obligatorios están marcados con asterisco (*) en el formulario
- Se debe validar integridad de datos antes de permitir creación
- Todo el proceso debe ser auditable siguiendo el Estándar de Auditoría Funcional
- Si ocurre un error durante la creación, el Usuario no debe perder los datos ingresados

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Administrador del Portal** | Quiero acceder a un menú de Administración de Empresas con la opción de crear nueva Empresa | Para registrar nuevos clientes en el sistema de forma centralizada |
| Como **Administrador del Portal** | Quiero completar un formulario guiado con validaciones en tiempo real para crear una Empresa | Para asegurar que la información fiscal y tributaria cumple con los requisitos de la DIAN y evitar errores de digitación |
| Como **Administrador del Portal** | Quiero que el sistema valide automáticamente el Dígito de Verificación (DV) según el algoritmo de la DIAN | Para garantizar que el NIT ingresado es válido y evitar rechazos posteriores en transmisiones a la DIAN |
| Como **Administrador del Portal** | Quiero que el sistema me alerte si el Número de Identificación ya existe en el sistema | Para prevenir duplicación de Empresas y mantener la integridad de los datos |
| Como **Administrador del Portal** | Quiero poder agregar múltiples Centros de Costos y Registros Mercantiles a la Empresa | Para capturar la estructura organizacional completa del cliente y sus domicilios fiscales |
| Como **Administrador del Portal** | Quiero revisar un resumen completo de los datos antes de confirmar la creación | Para verificar que toda la información es correcta antes de registrar la Empresa en el sistema |
| Como **Auditor** | Quiero que todas las creaciones de Empresas sean registradas con detalle completo | Para garantizar trazabilidad, identificar quién creó cada cliente, y cumplir con políticas de auditoría |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Acceso al menú de Administración de Empresas | Usuario Administrador del Portal autenticado en el sistema | Usuario navega al menú principal | Sistema muestra opción "Administración de Empresas" en el menú de navegación. Solo visible para Usuarios con rol de Administrador del Portal | ☐ | ☐ | ☐ |
| 2 | Opción de crear nueva Empresa | Administrador accede a módulo de Administración de Empresas | Administrador visualiza las opciones disponibles | Sistema muestra botón o enlace "Crear Nueva Empresa" prominente en la pantalla de Administración de Empresas | ☐ | ☐ | ☐ |
| 3 | Visualización del formulario de creación | Administrador hace clic en "Crear Nueva Empresa" | Sistema carga formulario | Sistema muestra formulario estructurado en 3 secciones: "Datos de la Empresa", "Ubicación", y "Registro Mercantil". Todos los campos marcados con asterisco (*) están identificados como obligatorios | ☐ | ☐ | ☐ |
| 4 | Selección de Tipo de Identificación | Administrador en formulario de creación, sección "Datos de la Empresa" | Administrador hace clic en campo "Tipo de Identificación" | Sistema muestra lista desplegable (select) con opciones de tipos de identificación: Registro civil, Tarjeta de identidad, Cédula de ciudadanía, Tarjeta de extranjería, Cédula de extranjería, NIT, Pasaporte, Doc. de identificación extranjero, NUIP, Permiso especial de permanencia (valores desde tabla maestra "Tipos Identificación.csv") | ☐ | ☐ | ☐ |
| 5 | Validación de Número de Identificación - solo numérico | Administrador ingresa valor en campo "Número de Identificación" | Administrador escribe caracteres no numéricos (letras, símbolos) | Sistema NO permite ingresar caracteres no numéricos. Solo acepta dígitos del 0 al 9. Máximo 10 caracteres | ☐ | ☐ | ☐ |
| 6 | Validación de unicidad de Número de Identificación | Administrador completa campo "Número de Identificación" con un valor que ya existe en el sistema | Administrador hace clic fuera del campo o presiona Enter | Sistema valida en tiempo real contra la base de datos. Muestra mensaje de error: "Este número de identificación ya existe en el sistema. No se puede crear una empresa duplicada." Campo se marca con error visual (borde rojo). No permite continuar hasta corregir | ☐ | ☐ | ☐ |
| 7 | Número de Identificación único y válido | Administrador ingresa Número de Identificación que NO existe en el sistema | Administrador hace clic fuera del campo o presiona Enter | Sistema valida y confirma que el número es único. Muestra indicador de validación exitosa (checkmark verde). Permite continuar con el formulario | ☐ | ☐ | ☐ |
| 8 | Cálculo y validación de Dígito de Verificación (DV) | Administrador ha ingresado Número de Identificación válido, ahora ingresa el DV en el campo "DV" | Administrador completa campo DV y hace clic fuera del campo | Sistema calcula el DV correcto usando el algoritmo de la DIAN: Lee dígitos del número de derecha a izquierda, multiplica cada uno por la secuencia fija (3, 7, 13, 17, 19, 23, 29, 37, 41, 43, 47, 53, 59, 67, 71), suma los resultados, divide entre 11 para obtener residuo. Si residuo es 0 o 1, DV = residuo. Si residuo > 1, DV = 11 - residuo. Compara DV ingresado con DV calculado. Si coinciden, muestra checkmark verde. Si NO coinciden, muestra error: "Dígito de verificación incorrecto. El DV correcto es: [X]" | ☐ | ☐ | ☐ |
| 9 | Validación de longitud de Nombre Abreviado | Administrador ingresa texto en campo "Nombre Abreviado" | Administrador escribe más de 50 caracteres | Sistema permite máximo 50 caracteres. Muestra contador "X/50 caracteres" debajo del campo. Al alcanzar 50 caracteres, NO permite escribir más | ☐ | ☐ | ☐ |
| 10 | Campo Razón Social activo solo si tipo ID es NIT | Administrador selecciona "NIT" como Tipo de Identificación | Sistema detecta selección de NIT | Campo "Razón Social" se activa (habilita para edición). Placeholder muestra "Ingrese la razón social de la empresa". Campo se marca como obligatorio (*Razón Social) | ☐ | ☐ | ☐ |
| 11 | Campo Razón Social inactivo si tipo ID NO es NIT | Administrador selecciona cualquier tipo de identificación diferente a "NIT" (ej: Cédula de ciudadanía) | Sistema detecta selección diferente a NIT | Campo "Razón Social" se desactiva (deshabilita, fondo gris). NO permite ingresar texto. Campo NO es obligatorio en este caso | ☐ | ☐ | ☐ |
| 12 | Campos Nombres y Apellidos activos si tipo ID NO es NIT | Administrador selecciona tipo de identificación diferente a "NIT" (ej: Cédula de ciudadanía, Pasaporte) | Sistema detecta selección diferente a NIT | Campos "Nombres" y "Apellidos" se activan (habilitan para edición). Ambos se marcan como obligatorios (*Nombres, *Apellidos). Placeholder muestra "Ingrese nombres" y "Ingrese apellidos" respectivamente | ☐ | ☐ | ☐ |
| 13 | Campos Nombres y Apellidos inactivos si tipo ID es NIT | Administrador selecciona "NIT" como Tipo de Identificación | Sistema detecta selección de NIT | Campos "Nombres" y "Apellidos" se desactivan (deshabilitan, fondo gris). NO permiten ingresar texto. Campos NO son obligatorios en este caso | ☐ | ☐ | ☐ |
| 14 | Selección de Tipo de Sociedad con búsqueda | Administrador hace clic en campo "Tipo de Sociedad" | Administrador abre lista desplegable | Sistema muestra lista desplegable (select) con opciones de Tipos de Sociedad: Asociación o corporación sin ánimo de lucro, Cooperativas, Otra, Sociedad anónima, Sociedad colectiva, Sociedad de economía mixta, Sociedad en comandita por acciones, Sociedad en comandita simple, Sociedad limitada, Sociedad por acciones simplificada, Sociedad unipersonal, Privada, Pública (valores desde tabla maestra "TipoSociedad.csv"). Lista permite búsqueda por texto (typeahead) | ☐ | ☐ | ☐ |
| 15 | Agregar Centro de Costos a la lista | Administrador en sección "Centros de Costos", campo de entrada vacío | Administrador ingresa valor numérico de hasta 5 dígitos y hace clic en botón "Agregar" | Sistema valida que: (1) El valor es numérico, (2) Máximo 5 dígitos, (3) No está vacío. Si válido, agrega el centro de costos a la tabla visible debajo del campo. Limpia el campo de entrada. Si inválido, muestra mensaje: "Ingrese un valor numérico de hasta 5 dígitos" | ☐ | ☐ | ☐ |
| 16 | Visualización de Centros de Costos agregados | Administrador ha agregado uno o más Centros de Costos | Sistema actualiza lista | Sistema muestra tabla con columnas: "Centro de Costo" y "Acciones". Cada fila muestra el número del centro de costo y botón "Eliminar" (icono de basurero o texto "Eliminar") | ☐ | ☐ | ☐ |
| 17 | Eliminar Centro de Costos con confirmación | Administrador hace clic en botón "Eliminar" junto a un Centro de Costos en la tabla | Sistema detecta clic en eliminar | Sistema muestra diálogo de confirmación: "¿Está seguro que desea eliminar el Centro de Costos [XXXXX]?". Botones: "Cancelar" y "Confirmar". Si Administrador hace clic en "Confirmar", elimina la fila de la tabla. Si hace clic en "Cancelar", cierra el diálogo sin eliminar | ☐ | ☐ | ☐ |
| 18 | Selección de Tipo de Contribuyente | Administrador hace clic en campo "Tipo de Contribuyente" | Administrador abre lista desplegable | Sistema muestra lista desplegable (select) con dos opciones: "Natural" y "Jurídico". Campo es obligatorio (*Tipo de Contribuyente) | ☐ | ☐ | ☐ |
| 19 | Selección de Tipo de Régimen | Administrador hace clic en campo "Tipo de Régimen" | Administrador abre lista desplegable | Sistema muestra lista desplegable (select) con opciones: "Ninguno" (valor: NA), "Simplificado (No responsable de IVA)" (valor: 49), "Común (Impuesto sobre las ventas)" (valor: 48). Campo es obligatorio (*Tipo de Régimen) | ☐ | ☐ | ☐ |
| 20 | Validación de formato de Correo Electrónico Emisor | Administrador ingresa texto en campo "Correo Electrónico Emisor" | Administrador completa el campo y hace clic fuera | Sistema valida formato de correo electrónico usando regex estándar (debe contener @, dominio válido, sin espacios). Si formato es válido, muestra checkmark verde. Si formato es inválido, muestra error: "Ingrese un correo electrónico válido (ejemplo: empresa@dominio.com)" | ☐ | ☐ | ☐ |
| 21 | Selección de País en sección Ubicación | Administrador en sección "Ubicación", hace clic en campo "País" | Administrador abre lista desplegable | Sistema muestra lista desplegable (dropdown) con todos los países del mundo (valores desde tabla maestra "Paises.csv"). Lista permite búsqueda por texto. Campo es obligatorio (*País) | ☐ | ☐ | ☐ |
| 22 | Campo Departamento activo solo si País es Colombia | Administrador selecciona "Colombia" en campo "País" | Sistema detecta selección de Colombia | Campo "Departamento" se activa (habilita para edición). Sistema carga lista desplegable con departamentos de Colombia (valores desde tabla maestra "Departamentos.csv": Amazonas, Antioquia, Arauca, Atlántico, Bolívar, Boyacá, etc.). Campo se marca como obligatorio (*Departamento). Lista permite búsqueda | ☐ | ☐ | ☐ |
| 23 | Campo Departamento inactivo si País NO es Colombia | Administrador selecciona cualquier país diferente a "Colombia" (ej: Argentina, México) | Sistema detecta selección diferente a Colombia | Campo "Departamento" se desactiva (deshabilita, fondo gris). NO muestra opciones. Campo NO es obligatorio. Si había un valor seleccionado previamente, se limpia | ☐ | ☐ | ☐ |
| 24 | Campo Ciudad activo solo si País es Colombia | Administrador selecciona "Colombia" como País y luego selecciona un Departamento | Sistema detecta selección de Colombia y departamento específico | Campo "Ciudad" se activa (habilita para edición). Sistema carga lista desplegable con ciudades del departamento seleccionado (valores desde tabla maestra "Ciudades.csv" filtrados por Departamento_Code). Campo se marca como obligatorio (*Ciudad). Lista permite búsqueda | ☐ | ☐ | ☐ |
| 25 | Campo Ciudad inactivo si País NO es Colombia | Administrador selecciona país diferente a "Colombia" | Sistema detecta selección diferente a Colombia | Campo "Ciudad" se desactiva (deshabilita, fondo gris). NO muestra opciones. Campo NO es obligatorio. Si había un valor seleccionado, se limpia | ☐ | ☐ | ☐ |
| 26 | Cambio de Departamento actualiza lista de Ciudades | Administrador ha seleccionado Colombia como País y un Departamento (ej: Antioquia), luego cambia a otro Departamento (ej: Cundinamarca) | Sistema detecta cambio en campo Departamento | Sistema recarga lista desplegable de "Ciudad" con las ciudades del nuevo departamento seleccionado. Si había una ciudad seleccionada del departamento anterior, se limpia la selección | ☐ | ☐ | ☐ |
| 27 | Validación de caracteres especiales en Dirección Principal | Administrador ingresa dirección en campo "Dirección Principal" | Administrador escribe caracteres especiales (#, -, números, letras, espacios, /, etc.) | Sistema permite caracteres alfanuméricos, espacios, y caracteres especiales comunes usados en direcciones (#, -, /, números de vía, bis, etc.). NO rechaza caracteres especiales. Campo es obligatorio (*Dirección Principal) | ☐ | ☐ | ☐ |
| 28 | Agregar Registro Mercantil - validación de Matrícula Mercantil | Administrador en sección "Registro Mercantil", ingresa valor en campo "Matrícula Mercantil" | Administrador completa el campo | Sistema valida: (1) Longitud entre 5 y 15 caracteres, (2) Solo permite letras (A-Z, a-z), números (0-9), guion (-), barra (/), (3) NO permite símbolos especiales (@, #, $, %, &, *), (4) NO permite espacios al inicio o al final. Si hay espacios extras, sistema los elimina automáticamente. Convierte a mayúsculas antes de guardar. Si formato es inválido, muestra error: "La matrícula debe tener entre 5 y 15 caracteres. Solo se permiten letras, números, guion y barra" | ☐ | ☐ | ☐ |
| 29 | Agregar Registro Mercantil - País, Departamento, Ciudad con misma lógica de Ubicación | Administrador completa campos de Registro Mercantil | Administrador selecciona País, Departamento, Ciudad | Sistema aplica la misma lógica condicional que en sección "Ubicación": Campo "País" con todos los países del mundo (obligatorio). Si País = Colombia, campos "Departamento" y "Ciudad" se activan (obligatorios) con listas de Departamentos y Ciudades de Colombia. Si País ≠ Colombia, campos "Departamento" y "Ciudad" se desactivan (no obligatorios) | ☐ | ☐ | ☐ |
| 30 | Validación de campos obligatorios de Registro Mercantil | Administrador intenta agregar un Registro Mercantil sin completar todos los campos obligatorios | Administrador hace clic en botón "Agregar Registro Mercantil" | Sistema valida que todos los campos obligatorios estén completos: *Matrícula Mercantil, *Nombre en Registro Mercantil, *País, *Departamento (si País=Colombia), *Ciudad (si País=Colombia), *Dirección. Si falta alguno, muestra mensaje: "Complete todos los campos obligatorios del Registro Mercantil antes de agregar" y marca los campos faltantes en rojo | ☐ | ☐ | ☐ |
| 31 | Agregar Registro Mercantil exitoso | Administrador ha completado todos los campos obligatorios de un Registro Mercantil | Administrador hace clic en "Agregar Registro Mercantil" | Sistema valida todos los campos. Si válidos, agrega el Registro Mercantil a la tabla visible debajo. Limpia todos los campos del formulario de Registro Mercantil para permitir agregar otro. Tabla muestra: Matrícula Mercantil, Nombre, País, Departamento (si aplica), Ciudad (si aplica), Dirección, botón "Eliminar" | ☐ | ☐ | ☐ |
| 32 | Visualización de Registros Mercantiles con paginación | Administrador ha agregado 6 o más Registros Mercantiles | Sistema actualiza tabla | Sistema muestra tabla de Registros Mercantiles con paginación. Máximo 5 registros visibles por página. Botones de navegación "Anterior" y "Siguiente" disponibles. Contador muestra "Mostrando 1-5 de X registros" | ☐ | ☐ | ☐ |
| 33 | Eliminar Registro Mercantil con confirmación | Administrador hace clic en botón "Eliminar" junto a un Registro Mercantil en la tabla | Sistema detecta clic en eliminar | Sistema muestra diálogo de confirmación: "¿Está seguro que desea eliminar el Registro Mercantil [Matrícula]?". Botones: "Cancelar" y "Confirmar". Si Administrador confirma, elimina la fila de la tabla. Si cancela, cierra el diálogo sin eliminar | ☐ | ☐ | ☐ |
| 34 | Validación de campos obligatorios del formulario completo | Administrador ha completado algunos campos pero NO todos los obligatorios | Administrador intenta avanzar al resumen o guardar | Sistema valida TODOS los campos obligatorios del formulario: *Tipo de Identificación, *Número de Identificación, DV (validado), Nombre Abreviado, *Razón Social (si NIT) o *Nombres y *Apellidos (si no NIT), *Tipo de Sociedad, *Tipo de Contribuyente, *Tipo de Régimen, *Correo Electrónico Emisor, *País, *Departamento (si Colombia), *Ciudad (si Colombia), *Dirección Principal. Si falta alguno, muestra mensaje: "Complete todos los campos obligatorios (*) antes de continuar" y hace scroll al primer campo faltante | ☐ | ☐ | ☐ |
| 35 | Botón Crear Empresa habilitado solo con campos obligatorios completos | Administrador completa todos los campos obligatorios correctamente | Sistema valida en tiempo real | Botón "Crear Empresa" se habilita (color primario, clickeable). Antes de completar todos los obligatorios, botón permanece deshabilitado (gris, cursor not-allowed) | ☐ | ☐ | ☐ |
| 36 | Pantalla de resumen antes de crear | Administrador ha completado todos los campos obligatorios y hace clic en "Crear Empresa" | Sistema valida y muestra resumen | Sistema muestra pantalla de resumen con todas las secciones expandibles: "Datos de la Empresa" (muestra todos los campos ingresados), "Ubicación" (País, Departamento, Ciudad, Dirección), "Centros de Costos" (lista completa), "Registros Mercantiles" (tabla completa con todos los registros). Botones disponibles: "Volver y Editar" y "Confirmar Creación" | ☐ | ☐ | ☐ |
| 37 | Volver a editar desde pantalla de resumen | Administrador en pantalla de resumen hace clic en "Volver y Editar" | Sistema detecta clic | Sistema regresa al formulario de creación manteniendo TODOS los datos ingresados previamente. Ningún campo se pierde. Administrador puede modificar cualquier campo y volver a intentar crear | ☐ | ☐ | ☐ |
| 38 | Creación exitosa de Empresa | Administrador en pantalla de resumen hace clic en "Confirmar Creación" | Sistema procesa la creación | Sistema guarda la Empresa en la base de datos con todos los datos ingresados, genera ID único de la Empresa, registra fecha y hora de creación, usuario creador. Muestra mensaje de éxito: "¡Empresa creada exitosamente! La empresa [Nombre Abreviado] con NIT [Número-DV] ha sido registrada en el sistema." Botones: "Ver Empresa" (navega a pantalla de detalle), "Crear Otra Empresa" (limpia formulario y permite crear nueva), "Volver a Administración de Empresas" | ☐ | ☐ | ☐ |
| 39 | Error durante creación - mensaje genérico | Administrador confirma creación pero ocurre error técnico en el servidor (timeout, error de BD, validación backend fallida) | Sistema intenta guardar pero falla | Sistema muestra mensaje de error genérico (NO revela detalles técnicos): "Ocurrió un error al crear la empresa. Por favor, intente nuevamente. Si el problema persiste, contacte a soporte técnico." Botón "Aceptar" disponible. Al hacer clic en "Aceptar", sistema regresa al formulario manteniendo TODOS los datos ingresados. Administrador puede revisar y volver a intentar sin perder información | ☐ | ☐ | ☐ |
| 40 | Validación de permiso de Usuario | Usuario sin rol de Administrador del Portal intenta acceder a Administración de Empresas mediante URL directa | Usuario no autorizado intenta acceso | Sistema valida permisos. Muestra mensaje: "No tiene permisos para acceder a esta sección. Solo usuarios Administradores pueden crear empresas." y redirige a pantalla de inicio del Portal | ☐ | ☐ | ☐ |
| 41 | Cancelar creación de Empresa | Administrador en medio del formulario de creación decide cancelar | Administrador hace clic en botón "Cancelar" | Sistema muestra diálogo de confirmación: "¿Está seguro que desea cancelar? Se perderán todos los datos ingresados." Botones: "Continuar Editando" y "Sí, Cancelar". Si Administrador confirma cancelación, limpia el formulario y regresa a pantalla de Administración de Empresas. Si continúa editando, cierra el diálogo y mantiene datos | ☐ | ☐ | ☐ |
| 42 | Campo Actividad Económica | Administrador ingresa información en campo "Actividad Económica" | Administrador completa el campo | Sistema permite ingresar texto libre describiendo la actividad económica principal de la empresa. Campo es opcional (no obligatorio). Máximo 200 caracteres | ☐ | ☐ | ☐ |
| 43 | Validación de caracteres especiales en Dirección de Registro Mercantil | Administrador ingresa dirección en Registro Mercantil | Administrador escribe caracteres especiales | Sistema aplica misma lógica que "Dirección Principal": permite caracteres alfanuméricos, espacios, y caracteres especiales comunes (#, -, /, etc.). NO rechaza caracteres especiales | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 44 | Estructura obligatoria de registro de auditoría | Cualquier evento relacionado con creación de Empresa (intento exitoso, intento fallido, validación de unicidad fallida, cancelación) | Sistema registra evento | Registro contiene: ID Único del Evento (UUID), Tipo de Evento (formato MODULO_ENTIDAD_ACCION), Fecha y Hora (UTC con milisegundos, ISO 8601), Usuario (nombre de usuario Administrador), Cliente (NULL en este caso, no aplica), Cliente (Nombre - NULL), IP Local, IP Pública, Resultado (EXITOSO/FALLIDO), Descripción (texto descriptivo del evento), Nivel de Severidad (INFO/WARNING/ERROR), Datos Adicionales (JSON con datos relevantes: NIT, nombre empresa, etc.) | ☐ | ☐ | ☐ |
| 45 | Auditoría de creación exitosa de Empresa | Administrador confirma creación de Empresa y el proceso es exitoso | Sistema guarda Empresa en base de datos | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_EMPRESA_CREACION_EXITOSA", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Administrador {usuario} creó empresa {nombre_abreviado} con NIT {numero_identificacion}-{dv}", Datos Adicionales: {"empresa_id": "UUID", "numero_identificacion": "NIT", "dv": "X", "nombre_abreviado": "Nombre", "razon_social": "Razón Social o NULL", "tipo_identificacion": "31", "tipo_contribuyente": "Jurídico", "pais": "Colombia", "email_emisor": "correo@empresa.com", "registros_mercantiles_count": 2} | ☐ | ☐ | ☐ |
| 46 | Auditoría de intento fallido por NIT duplicado | Administrador ingresa Número de Identificación que ya existe y sistema detecta duplicación | Administrador hace clic fuera del campo o presiona Enter | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_EMPRESA_VALIDACION_NIT_DUPLICADO", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Administrador {usuario} intentó crear empresa con NIT duplicado {numero_identificacion}", Datos Adicionales: {"numero_identificacion": "NIT", "empresa_existente_id": "UUID de empresa existente", "nombre_empresa_existente": "Nombre"} | ☐ | ☐ | ☐ |
| 47 | Auditoría de error técnico durante creación | Administrador confirma creación pero ocurre error técnico (timeout, error BD) | Sistema intenta guardar pero falla | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_EMPRESA_CREACION_ERROR", Resultado: "FALLIDO", Severidad: "ERROR", Descripción: "Error al crear empresa {nombre_abreviado} por Administrador {usuario}", Datos Adicionales: {"numero_identificacion": "NIT", "nombre_abreviado": "Nombre", "error_tipo": "timeout o database_error", "error_mensaje": "Mensaje técnico del error (no mostrado al usuario)", "datos_formulario": "JSON con todos los datos ingresados para debugging"} | ☐ | ☐ | ☐ |
| 48 | Auditoría de cancelación de creación | Administrador cancela el proceso de creación después de haber ingresado datos | Administrador confirma cancelación en diálogo | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_EMPRESA_CREACION_CANCELADA", Resultado: "EXITOSO", Severidad: "INFO", Descripción: "Administrador {usuario} canceló creación de empresa", Datos Adicionales: {"numero_identificacion": "NIT parcial si fue ingresado", "nombre_abreviado": "Nombre parcial si fue ingresado", "porcentaje_completado": "estimado basado en campos completados"} | ☐ | ☐ | ☐ |
| 49 | Auditoría de validación de DV incorrecto | Administrador ingresa DV incorrecto para un NIT | Sistema valida y detecta DV incorrecto | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_EMPRESA_VALIDACION_DV_INCORRECTO", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Administrador {usuario} ingresó DV incorrecto para NIT {numero_identificacion}", Datos Adicionales: {"numero_identificacion": "NIT", "dv_ingresado": "X", "dv_correcto": "Y", "algoritmo_usado": "DIAN_oficial"} | ☐ | ☐ | ☐ |
| 50 | Auditoría de acceso no autorizado | Usuario sin rol Administrador intenta acceder a creación de Empresas | Usuario no autorizado navega a URL | Registro de auditoría: Tipo de Evento: "ADMINISTRACION_EMPRESA_ACCESO_DENEGADO", Resultado: "FALLIDO", Severidad: "WARNING", Descripción: "Usuario {usuario} sin permisos intentó acceder a creación de empresas", Datos Adicionales: {"rol_usuario": "Usuario estándar", "ip_origen": "IP", "url_intentada": "/admin/empresas/crear"} | ☐ | ☐ | ☐ |
| 51 | Inmutabilidad de registros de auditoría | Cualquier escenario de auditoría de creación de Empresas | Sistema genera registro | Registros de auditoría NO pueden ser modificados ni eliminados. Solo se permiten consultas de lectura. Implementar controles que prevengan alteración de registros históricos | ☐ | ☐ | ☐ |
| 52 | Consulta de registros de auditoría de creación de Empresas | Administrador o Auditor requiere revisar historial de creaciones de Empresas | Administrador accede a módulo de auditoría | Sistema permite filtrar por: rango de fechas, usuario creador, tipo de evento (ADMINISTRACION_EMPRESA_*), resultado (EXITOSO/FALLIDO), severidad (INFO/WARNING/ERROR), número de identificación. Permite exportar resultados en CSV/Excel. Muestra: fecha/hora, usuario, evento, empresa creada, resultado | ☐ | ☐ | ☐ |
| 53 | Retención de registros de auditoría | Registros de auditoría de creación de Empresas | Paso del tiempo | Registros se conservan por tiempo mínimo definido por regulación (7 años para documentos fiscales, aplicar misma política). Después de este período, pueden ser archivados en almacenamiento de largo plazo o eliminados según política de retención | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

Se requieren los siguientes mockups interactivos en HTML siguiendo la Guía de Estilos (Material UI, paleta de colores Primary #4A5A9E, Secondary #D4145A):

#### 1. **HU004-MenuAdministracion.html** - Menú de Administración de Empresas
**Elementos:**
- Barra de navegación con menú "Administración" visible solo para Administradores
- Submenu: "Administración de Empresas"
- Botón prominente: "Crear Nueva Empresa" (gradiente Primary → Secondary)
- Tabla con listado de Empresas existentes (Número ID, Nombre, Estado, Fecha Creación, Acciones)
- Buscador de Empresas
- Paginación

#### 2. **HU004-CrearEmpresa.html** - Formulario de creación de Empresa
**Elementos:**

**Sección 1: Datos de la Empresa**
- *Tipo de Identificación (select con opciones de tabla maestra)
- *Número de Identificación (input numérico, max 10, validación de unicidad en blur/enter con checkmark verde o error rojo)
- DV (input numérico 1 dígito, validación de algoritmo DIAN con checkmark o error mostrando DV correcto)
- Nombre Abreviado (input texto, max 50 caracteres, contador "X/50")
- Razón Social (input texto, activo solo si Tipo ID = NIT, obligatorio en ese caso)
- Nombres (input texto, activo solo si Tipo ID ≠ NIT, obligatorio en ese caso)
- Apellidos (input texto, activo solo si Tipo ID ≠ NIT, obligatorio en ese caso)
- *Tipo de Sociedad (select con búsqueda/typeahead, tabla maestra)
- Centros de Costos:
  - Input numérico (max 5 dígitos) + botón "Agregar"
  - Tabla con centros agregados (Centro, Acción Eliminar)
  - Diálogo de confirmación al eliminar
- *Tipo de Contribuyente (select: Natural/Jurídico)
- *Tipo de Régimen (select: Ninguno NA / Simplificado 49 / Común 48)
- *Correo Electrónico Emisor (input email, validación de formato)
- Actividad Económica (textarea, max 200 caracteres, opcional)

**Sección 2: Ubicación**
- *País (dropdown con búsqueda, tabla maestra Paises)
- *Departamento (dropdown, solo activo si País=Colombia, tabla maestra Departamentos)
- *Ciudad (dropdown, solo activo si País=Colombia, tabla maestra Ciudades filtradas por Departamento)
- *Dirección Principal (input texto, permite caracteres especiales)

**Sección 3: Registro Mercantil** (formulario repetible)
- *Matrícula Mercantil (input texto, 5-15 caracteres, validación letras/números/guion/barra, normalización a mayúsculas)
- *Nombre en Registro Mercantil (input texto)
- *País (dropdown con búsqueda, misma tabla maestra)
- *Departamento (condicional si País=Colombia)
- *Ciudad (condicional si País=Colombia y según Departamento)
- *Dirección (input texto, permite caracteres especiales)
- Botón "Agregar Registro Mercantil"
- Tabla con registros agregados (Matrícula, Nombre, País, Depto, Ciudad, Dirección, Acción Eliminar)
- Paginación (máximo 5 registros visibles por página)
- Diálogo de confirmación al eliminar

**Botones del formulario:**
- "Cancelar" (outlined, confirmación de pérdida de datos)
- "Crear Empresa" (contained, gradiente, deshabilitado hasta completar obligatorios)

#### 3. **HU004-ResumenEmpresa.html** - Pantalla de resumen antes de confirmar
**Elementos:**
- Título: "Resumen de la Empresa a Crear"
- Secciones expandibles/colapsables:
  - Datos de la Empresa (todos los campos ingresados)
  - Ubicación (País, Departamento, Ciudad, Dirección)
  - Centros de Costos (lista completa)
  - Registros Mercantiles (tabla completa con todos los registros)
- Botones:
  - "Volver y Editar" (outlined)
  - "Confirmar Creación" (contained, gradiente Primary → Secondary)

#### 4. **HU004-CreacionExitosa.html** - Confirmación de creación exitosa
**Elementos:**
- Icono de éxito (checkmark grande en verde)
- Título: "¡Empresa Creada Exitosamente!"
- Mensaje: "La empresa [Nombre Abreviado] con NIT [Número-DV] ha sido registrada en el sistema."
- Resumen breve de datos creados
- Botones:
  - "Ver Empresa" (navega a detalle de empresa)
  - "Crear Otra Empresa" (limpia formulario)
  - "Volver a Administración de Empresas"

#### 5. **HU004-ErrorCreacion.html** - Pantalla de error durante creación
**Elementos:**
- Icono de error (icono warning en naranja/rojo)
- Título: "Error al Crear Empresa"
- Mensaje: "Ocurrió un error al crear la empresa. Por favor, intente nuevamente. Si el problema persiste, contacte a soporte técnico."
- Botón "Aceptar" (regresa al formulario manteniendo datos)

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Error en validación de DV** | Si el algoritmo de validación del Dígito de Verificación se implementa incorrectamente, se podrían crear Empresas con NIT inválido que luego serán rechazadas por la DIAN en transmisiones de facturas | Alto | Baja | Implementar algoritmo oficial de la DIAN exactamente como está documentado: multiplicadores (3, 7, 13, 17, 19, 23, 29, 37, 41, 43, 47, 53, 59, 67, 71), suma, módulo 11, residuo. Incluir casos de prueba con NITs reales verificados. Validar en backend además del frontend | Mitigado |
| **Duplicación de Empresas por validación de unicidad fallida** | Si la validación de unicidad del Número de Identificación falla (ej: condición de carrera, error en query), se podrían crear Empresas duplicadas, generando inconsistencia en facturación y reportes | Alto | Baja | Implementar validación en 3 niveles: (1) Frontend en tiempo real (blur/enter), (2) Backend al recibir datos (antes de guardar), (3) Constraint UNIQUE en base de datos a nivel de columna numero_identificacion. Usar transacciones para garantizar atomicidad | Mitigado |
| **Pérdida de datos ingresados por error o timeout** | Si ocurre error durante creación y el sistema no mantiene los datos ingresados, el Administrador debe volver a escribir todo el formulario (20+ campos, centros de costos, registros mercantiles), generando frustración y posibles errores de re-digitación | Medio | Media | Mantener todos los datos del formulario en estado (memoria) después de error. Al regresar al formulario, pre-poblar todos los campos. Opcionalmente, implementar autoguardado temporal en localStorage del navegador cada 30 segundos como respaldo adicional | Mitigado |
| **Tablas maestras desactualizadas** | Si las tablas maestras (Tipos de Identificación, Tipos de Sociedad, Países, Departamentos, Ciudades) no se mantienen actualizadas con cambios normativos DIAN o geográficos, los Administradores no podrán seleccionar valores válidos | Medio | Media | Diseñar módulo de mantenimiento de tablas maestras accesible para Administradores. Implementar proceso de actualización periódica (trimestral) revisando resoluciones DIAN y cambios geográficos. Permitir agregar valores nuevos manualmente si no están en la lista | Mitigado |
| **Formulario muy extenso causa abandono** | Un formulario con 20+ campos distribuidos en 3 secciones puede ser percibido como muy largo y complejo, causando que Administradores abandonen el proceso o cometan errores por fatiga | Medio | Media | Dividir formulario en pasos (wizard) con indicador de progreso: Paso 1 (Identificación), Paso 2 (Datos Empresa), Paso 3 (Ubicación), Paso 4 (Registros Mercantiles), Paso 5 (Resumen). Permitir navegación entre pasos. Guardar progreso en cada paso. Mostrar solo campos relevantes según tipo de identificación (ocultar Nombres/Apellidos si NIT) | Mitigado |
| **Caracteres especiales en direcciones causan problemas en integraciones DIAN** | Si se permiten caracteres especiales poco comunes en campos de dirección (emojis, símbolos raros) y luego esos datos se envían a la DIAN en XML de factura, la DIAN podría rechazar el documento por caracteres inválidos | Medio | Baja | Definir lista permitida de caracteres especiales para direcciones basada en estándar de facturación electrónica DIAN (XML 1.0): letras, números, espacios, #, -, /, bis, números de vía. Rechazar caracteres fuera de esta lista. Validar en frontend y backend antes de guardar | Mitigado |
| **Falta de validación de formato de Correo Electrónico Emisor** | Si el correo electrónico no se valida correctamente y se guarda con formato inválido, el sistema no podrá enviar notificaciones de facturación a la Empresa, causando pérdida de comunicación | Medio | Baja | Validar formato de email con regex estándar en frontend (retroalimentación inmediata). Validar en backend antes de guardar. Opcionalmente, enviar correo de verificación a la dirección ingresada para confirmar que es válida y está activa | Mitigado |
| **Registros Mercantiles sin límite de cantidad** | Si no se establece un límite razonable de Registros Mercantiles que se pueden agregar, un Administrador podría agregar cientos de registros por error o malicia, afectando rendimiento de carga de datos | Bajo | Baja | Establecer límite razonable de Registros Mercantiles por Empresa (ej: máximo 20 registros). Mostrar mensaje al alcanzar el límite: "Ha alcanzado el número máximo de Registros Mercantiles permitidos (20)". Si se requiere más, contactar soporte para caso especial | Aceptado con límite |
| **Usuarios no Administradores acceden mediante manipulación de URL** | Si la validación de permisos solo se hace en frontend (ocultando menú), un Usuario estándar podría acceder a la URL directa de creación de Empresas y crear/modificar datos sin autorización | Alto | Baja | Implementar validación de permisos en TODAS las capas: (1) Frontend oculta menú, (2) Frontend valida permisos al cargar componente, (3) Backend valida rol de Usuario en CADA endpoint de creación/modificación. Retornar HTTP 403 Forbidden si no autorizado. Registrar intento en auditoría | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance (Out of Scope)
- Modificación/Edición de Empresas existentes (se cubrirá en HU005 - Edición de Empresas)
- Eliminación/Inactivación de Empresas (se cubrirá en HU006 - Gestión del Ciclo de Vida de Empresas)
- Importación masiva de Empresas desde archivo CSV/Excel (se evaluará en Release 2)
- Asignación de Usuarios a Empresas (se cubre en HU007 - Gestión de Usuarios y Permisos)
- Configuración de productos contratados por Empresa (Facturación, RADIAN, etc.) - se cubre en módulo de Productos
- Consulta y visualización de listado de Empresas (cubierto parcialmente en HU004-MenuAdministracion, se detallará en HU separada)

### Consideraciones Técnicas para Implementación (Informativas)
- **Algoritmo DV DIAN:** Implementar exactamente como se describe: multiplicadores fijos (3, 7, 13, 17, 19, 23, 29, 37, 41, 43, 47, 53, 59, 67, 71), lectura de derecha a izquierda, suma, módulo 11, si residuo 0 o 1 → DV = residuo, si residuo > 1 → DV = 11 - residuo
- **Validación de unicidad:** Implementar constraint UNIQUE en columna numero_identificacion en base de datos. Query de validación en backend: `SELECT COUNT(*) FROM empresas WHERE numero_identificacion = ?`
- **Tablas maestras:** Implementar como tablas de referencia en base de datos con capacidad de mantenimiento (CRUD) por Administradores
- **Cascada de selects:** Al cambiar País a Colombia → cargar Departamentos. Al seleccionar Departamento → cargar Ciudades de ese departamento. Usar llamadas asíncronas para no bloquear UI
- **Normalización de datos:** Matrícula Mercantil → convertir a mayúsculas, eliminar espacios extras. Direcciones → trim() para eliminar espacios al inicio/final
- **Validación frontend y backend:** SIEMPRE validar en ambos lados. Frontend para UX inmediata, backend para seguridad y integridad
- **Manejo de errores:** Catch de excepciones en backend, logging detallado para debugging, mensaje genérico al usuario, mantener datos del formulario
- **Transacciones:** Usar transacciones de base de datos para garantizar que Empresa + Centros de Costos + Registros Mercantiles se guarden atómicamente (todo o nada)

### Dependencias
- **Tablas Maestras:** Tipos Identificación, Tipo Sociedad, Países, Departamentos, Ciudades deben estar creadas y pobladas antes de habilitar creación de Empresas
- **Módulo de Autenticación (HU001):** Usuario Administrador debe estar autenticado para acceder a creación de Empresas
- **Sistema de Permisos/Roles:** Debe existir validación de rol "Administrador del Portal" en frontend y backend
- **Módulo de Auditoría:** Debe estar implementado para registrar eventos de creación/errores/validaciones

### Referencias
- **Estándar de Auditoría Funcional**: Seguir estructura obligatoria de 13 campos, formato de tipos de eventos (ADMINISTRACION_EMPRESA_*), niveles de severidad
- **Estándar de Seguridad APL03**:
  - Validación de permisos en todas las capas
  - Registro de eventos de administración
  - Protección contra inyección SQL en campos de texto
- **Glosario**: Empresa, Cliente, NIT, Dígito de Verificación, Registro Mercantil, Administrador del Portal
- **Resoluciones DIAN**: Validar que estructura de datos cumple con requisitos de facturación electrónica (Resolución 000042 de 2020 y posteriores)
- **Guía de Estilos**: Material UI, paleta Primary #4A5A9E, Secondary #D4145A, validaciones con feedback visual (checkmarks verdes, errores rojos), botones con gradientes

### Algoritmo Oficial de Cálculo del Dígito de Verificación (DV) DIAN

**Ejemplo práctico:**

NIT: `8001234567` (sin DV)

**Paso 1:** Leer dígitos de derecha a izquierda y asignar multiplicadores:
```
Posición:  10  9  8  7  6  5  4  3  2  1
Dígito:     8  0  0  1  2  3  4  5  6  7
Multiplicador: 71 67 59 53 47 43 41 37 29 23 (continúa patrón: 19, 17, 13, 7, 3...)

Nota: Solo se usan los multiplicadores necesarios según la cantidad de dígitos
```

**Paso 2:** Multiplicar cada dígito por su multiplicador:
```
8 × 71 = 568
0 × 67 = 0
0 × 59 = 0
1 × 53 = 53
2 × 47 = 94
3 × 43 = 129
4 × 41 = 164
5 × 37 = 185
6 × 29 = 174
7 × 23 = 161
```

**Paso 3:** Sumar todos los resultados:
```
568 + 0 + 0 + 53 + 94 + 129 + 164 + 185 + 174 + 161 = 1528
```

**Paso 4:** Dividir entre 11 y obtener residuo:
```
1528 ÷ 11 = 138 con residuo 10
Residuo = 1528 - (138 × 11) = 1528 - 1518 = 10
```

**Paso 5:** Calcular DV según residuo:
- Si residuo = 0 → DV = 0
- Si residuo = 1 → DV = 1
- Si residuo > 1 → DV = 11 - residuo

```
Residuo = 10 (mayor que 1)
DV = 11 - 10 = 1
```

**Resultado:** NIT completo: `8001234567-1`

**Secuencia completa de multiplicadores DIAN (de derecha a izquierda):**
`3, 7, 13, 17, 19, 23, 29, 37, 41, 43, 47, 53, 59, 67, 71`

(Para NITs de más de 15 dígitos, el patrón se repetiría, pero en Colombia los NITs no superan 10-12 dígitos)
