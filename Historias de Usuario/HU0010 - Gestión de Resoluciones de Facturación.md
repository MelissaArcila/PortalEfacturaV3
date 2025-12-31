# HU003 - Gestión de Resoluciones de Facturación

## Información General

| Campo | Descripción |
|-------|-------------|
| **Release objetivo** | Release 1 |
| **Epic** | Gestión de Resoluciones de Facturación |
| **Estado** | BORRADOR |
| **Propietario** | Por definir |
| **Arquitecto** | Por definir |
| **Desarrolladores** | Por definir |
| **Tester** | Por definir |
| **Incidentes relacionados** | N/A |
| **Arquitectura relacionada** | Portal Unificado, Módulo de Facturación |

---

## Contexto

Una vez que el Administrador de Cliente puede crear resoluciones de facturación (HU002), es necesario proporcionar una pantalla de gestión donde pueda visualizar todas las resoluciones de su cliente, consultar sus detalles, filtrar por diferentes criterios y modificar el estado de las resoluciones entre Activa e Inactiva.

Esta funcionalidad es fundamental para que el cliente pueda tener visibilidad completa de sus autorizaciones de emisión y realizar ajustes operativos según sus necesidades de negocio.

---

**Notas:**
- Esta HU se enfoca en **listar, consultar y cambiar estado** de resoluciones.
- La creación de resoluciones se maneja en HU002.
- Los estados automáticos (Vencida, Agotada) se gestionarán en futuras HUs.
- Solo se puede cambiar entre estados Activa/Inactiva manualmente. Los estados Vencida y Agotada son definitivos y no modificables.

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Administrador de Cliente** | Quiero visualizar un listado completo de todas las resoluciones de mi cliente | Para tener visibilidad de todas las autorizaciones de emisión configuradas |
| Como **Administrador de Cliente** | Quiero filtrar las resoluciones por número, prefijo y estado | Para encontrar rápidamente resoluciones específicas |
| Como **Administrador de Cliente** | Quiero ver el detalle completo de una resolución | Para consultar toda la información configurada en una resolución específica |
| Como **Administrador de Cliente** | Quiero activar o inactivar resoluciones | Para habilitar o deshabilitar la emisión de documentos según las necesidades operativas |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Acceso a la pantalla de gestión de resoluciones | El Administrador de Cliente ha iniciado sesión en el Portal Unificado | El usuario navega al módulo de Resoluciones | El sistema muestra la pantalla de gestión de resoluciones con el listado de todas las resoluciones del cliente en una tabla paginada | ☐ | ☐ | ☐ |
| 2 | Estructura de la tabla de listado | El Administrador de Cliente visualiza la pantalla de gestión | El sistema carga el listado de resoluciones | El sistema muestra una tabla paginada con las columnas: Número de Resolución, Prefijo, Estado, y Acciones (iconos de Ver Detalle y Editar Estado) | ☐ | ☐ | ☐ |
| 3 | Orden por defecto del listado | El Administrador de Cliente accede al listado sin aplicar filtros | El sistema carga las resoluciones por primera vez | El sistema muestra las resoluciones ordenadas mostrando primero las que tienen estado "Activa", luego "Inactiva", luego "Vencida", y finalmente "Agotada" | ☐ | ☐ | ☐ |
| 4 | Paginación del listado | El cliente tiene más resoluciones de las que caben en una página | El sistema carga el listado | El sistema muestra la paginación con controles para navegar entre páginas y selector de cantidad de elementos por página | ☐ | ☐ | ☐ |
| 5 | Botón crear resolución visible | El Administrador de Cliente visualiza la pantalla de gestión | El sistema muestra la interfaz | El sistema muestra un botón "Crear Resolución" claramente visible que al hacer clic redirige al formulario de creación (HU002) | ☐ | ☐ | ☐ |
| 6 | Filtro por número de resolución | El Administrador de Cliente ingresa un número de resolución en el campo de filtro | El usuario escribe en el campo "Número de Resolución" y presiona buscar o Enter | El sistema filtra la tabla mostrando solo las resoluciones cuyo número contenga el texto ingresado (búsqueda parcial, sin distinguir mayúsculas/minúsculas) | ☐ | ☐ | ☐ |
| 7 | Filtro por prefijo | El Administrador de Cliente ingresa un prefijo en el campo de filtro | El usuario escribe en el campo "Prefijo" y presiona buscar o Enter | El sistema filtra la tabla mostrando solo las resoluciones cuyo prefijo contenga el texto ingresado (búsqueda parcial, sin distinguir mayúsculas/minúsculas) | ☐ | ☐ | ☐ |
| 8 | Filtro por estado | El Administrador de Cliente selecciona un estado en el filtro | El usuario selecciona un estado de la lista desplegable (Activa, Inactiva, Vencida, Agotada, o "Todos") | El sistema filtra la tabla mostrando solo las resoluciones con el estado seleccionado. Si selecciona "Todos", muestra todas las resoluciones sin filtro de estado | ☐ | ☐ | ☐ |
| 9 | Filtros combinados | El Administrador de Cliente aplica múltiples filtros simultáneamente | El usuario ingresa valores en varios campos de filtro y presiona buscar | El sistema filtra la tabla aplicando todos los criterios simultáneamente mediante operador lógico AND (ej: número de resolución Y prefijo Y estado) | ☐ | ☐ | ☐ |
| 10 | Limpiar filtros | El Administrador de Cliente ha aplicado uno o más filtros | El usuario hace clic en el botón "Limpiar filtros" o similar | El sistema elimina todos los filtros aplicados, limpia los campos de filtro y muestra nuevamente todas las resoluciones con el orden por defecto | ☐ | ☐ | ☐ |
| 11 | Ver detalle de resolución | El Administrador de Cliente hace clic en el icono de "Ver Detalle" de una resolución | El usuario hace clic en el icono de visualización | El sistema muestra una pantalla o modal con todos los detalles de la resolución: Número de resolución, Número de formulario DIAN, Tipo de documento, Prefijo, Consecutivo inicial, Consecutivo final, Fecha inicial de vigencia, Fecha final de vigencia, Punto/Sede (si aplica), Estado actual, Fecha de creación, Usuario que creó | ☐ | ☐ | ☐ |
| 12 | Cerrar detalle de resolución | El sistema muestra el detalle de una resolución | El usuario hace clic en "Cerrar" o "Volver" | El sistema cierra la vista de detalle y regresa al listado de resoluciones manteniendo los filtros aplicados previamente | ☐ | ☐ | ☐ |
| 13 | Cambio de estado de Activa a Inactiva - solicitar confirmación | El Administrador de Cliente hace clic en el toggle/switch de estado de una resolución que está en estado "Activa" | El usuario cambia el toggle de Activa a Inactiva | El sistema muestra un mensaje de confirmación "¿Está seguro que desea inactivar esta resolución? No podrá utilizarse para emitir documentos" con opciones "Sí, inactivar" y "Cancelar" | ☐ | ☐ | ☐ |
| 14 | Cambio de estado de Activa a Inactiva - confirmación afirmativa | El sistema muestra la confirmación de inactivación | El usuario hace clic en "Sí, inactivar" | El sistema cambia el estado de la resolución a "Inactiva", actualiza la tabla mostrando el nuevo estado, muestra mensaje "Estado actualizado exitosamente" y registra el evento en auditoría | ☐ | ☐ | ☐ |
| 15 | Cambio de estado de Activa a Inactiva - cancelación | El sistema muestra la confirmación de inactivación | El usuario hace clic en "Cancelar" | El sistema cierra el mensaje de confirmación, mantiene el estado original "Activa" sin realizar cambios y deja el toggle en su posición original | ☐ | ☐ | ☐ |
| 16 | Cambio de estado de Inactiva a Activa - solicitar confirmación | El Administrador de Cliente hace clic en el toggle/switch de estado de una resolución que está en estado "Inactiva" | El usuario cambia el toggle de Inactiva a Activa | El sistema muestra un mensaje de confirmación "¿Está seguro que desea activar esta resolución? Podrá utilizarse para emitir documentos" con opciones "Sí, activar" y "Cancelar" | ☐ | ☐ | ☐ |
| 17 | Cambio de estado de Inactiva a Activa - confirmación afirmativa | El sistema muestra la confirmación de activación | El usuario hace clic en "Sí, activar" | El sistema cambia el estado de la resolución a "Activa", actualiza la tabla mostrando el nuevo estado, muestra mensaje "Estado actualizado exitosamente" y registra el evento en auditoría | ☐ | ☐ | ☐ |
| 18 | Cambio de estado de Inactiva a Activa - cancelación | El sistema muestra la confirmación de activación | El usuario hace clic en "Cancelar" | El sistema cierra el mensaje de confirmación, mantiene el estado original "Inactiva" sin realizar cambios y deja el toggle en su posición original | ☐ | ☐ | ☐ |
| 19 | Restricción de cambio de estado Vencida | El Administrador de Cliente visualiza una resolución con estado "Vencida" | El usuario intenta interactuar con el toggle/switch de estado | El sistema muestra el toggle deshabilitado o sin opción de cambio, con un tooltip o mensaje "No se puede modificar el estado de una resolución vencida" | ☐ | ☐ | ☐ |
| 20 | Restricción de cambio de estado Agotada | El Administrador de Cliente visualiza una resolución con estado "Agotada" | El usuario intenta interactuar con el toggle/switch de estado | El sistema muestra el toggle deshabilitado o sin opción de cambio, con un tooltip o mensaje "No se puede modificar el estado de una resolución agotada" | ☐ | ☐ | ☐ |
| 21 | Visualización de estado en la tabla | El Administrador de Cliente visualiza el listado de resoluciones | El sistema muestra la tabla | El sistema muestra el estado de cada resolución de forma clara y diferenciada visualmente (ej: con colores o badges): Activa (verde), Inactiva (gris), Vencida (naranja), Agotada (rojo) | ☐ | ☐ | ☐ |
| 22 | Listado vacío - sin resoluciones creadas | El Administrador de Cliente accede al módulo y su cliente no tiene resoluciones creadas | El sistema carga el listado | El sistema muestra un mensaje "No hay resoluciones creadas" o similar, junto con el botón "Crear Resolución" destacado para invitar a crear la primera resolución | ☐ | ☐ | ☐ |
| 23 | Listado vacío - por filtros aplicados | El Administrador de Cliente aplica filtros que no coinciden con ninguna resolución | El sistema aplica los filtros | El sistema muestra un mensaje "No se encontraron resoluciones con los criterios de búsqueda aplicados" y mantiene visibles los filtros para que puedan ser modificados | ☐ | ☐ | ☐ |

### Criterios de Aceptación - Auditoría y Trazabilidad

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 24 | Auditoría de consulta de listado de resoluciones | Ocurre el escenario 1 (acceso al listado) | El Administrador de Cliente accede a la pantalla de gestión | El sistema crea un registro de auditoría con: Tipo de evento="RESOLUCIONES_CONSULTADAS", Resultado="EXITOSO", Descripción="Consulta de listado de resoluciones", Nivel de severidad="INFO", Datos adicionales: nombre de usuario, NIT del cliente, cantidad de resoluciones mostradas, filtros aplicados (si hay), fecha/hora, IP local, IP pública | ☐ | ☐ | ☐ |
| 25 | Auditoría de visualización de detalle | Ocurre el escenario 11 (ver detalle) | El usuario visualiza el detalle completo de una resolución | El sistema crea un registro de auditoría con: Tipo de evento="RESOLUCION_DETALLE_CONSULTADO", Resultado="EXITOSO", Descripción="Consulta de detalle de resolución", Nivel de severidad="INFO", Datos adicionales: nombre de usuario, NIT del cliente, número de resolución consultada, prefijo, tipo de documento, fecha/hora, IP local, IP pública | ☐ | ☐ | ☐ |
| 26 | Auditoría de cambio de estado a Inactiva | Ocurre el escenario 14 (inactivación confirmada) | El usuario inactiva una resolución exitosamente | El sistema crea un registro de auditoría con: Tipo de evento="RESOLUCION_INACTIVADA", Resultado="EXITOSO", Descripción="Resolución cambiada de estado Activa a Inactiva", Nivel de severidad="WARNING", Datos adicionales: nombre de usuario, NIT del cliente, número de resolución, prefijo, tipo de documento, estado anterior (Activa), estado nuevo (Inactiva), fecha/hora, IP local, IP pública | ☐ | ☐ | ☐ |
| 27 | Auditoría de cambio de estado a Activa | Ocurre el escenario 17 (activación confirmada) | El usuario activa una resolución exitosamente | El sistema crea un registro de auditoría con: Tipo de evento="RESOLUCION_ACTIVADA", Resultado="EXITOSO", Descripción="Resolución cambiada de estado Inactiva a Activa", Nivel de severidad="INFO", Datos adicionales: nombre de usuario, NIT del cliente, número de resolución, prefijo, tipo de documento, estado anterior (Inactiva), estado nuevo (Activa), fecha/hora, IP local, IP pública | ☐ | ☐ | ☐ |
| 28 | Auditoría de cancelación de cambio de estado | Ocurre el escenario 15 o 18 (cancelación de cambio) | El usuario cancela el cambio de estado | El sistema crea un registro de auditoría con: Tipo de evento="RESOLUCION_CAMBIO_ESTADO_CANCELADO", Resultado="EXITOSO", Descripción="Cambio de estado de resolución cancelado por el usuario", Nivel de severidad="INFO", Datos adicionales: nombre de usuario, NIT del cliente, número de resolución, estado actual (sin cambios), acción cancelada (activar/inactivar), fecha/hora, IP local, IP pública | ☐ | ☐ | ☐ |
| 29 | Auditoría de intento de cambio de estado no permitido | Ocurre el escenario 19 o 20 (intento de cambiar estado Vencida/Agotada) | El usuario intenta cambiar el estado de una resolución Vencida o Agotada | El sistema crea un registro de auditoría con: Tipo de evento="RESOLUCION_CAMBIO_ESTADO_RECHAZADO", Resultado="FALLIDO", Descripción="Intento de cambiar estado de resolución en estado definitivo", Nivel de severidad="WARNING", Datos adicionales: nombre de usuario, NIT del cliente, número de resolución, estado actual (Vencida/Agotada), motivo rechazo (estado definitivo no modificable), fecha/hora, IP local, IP pública | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes
Pendiente

**Descripción funcional de la pantalla de gestión:**

**Pantalla - Gestión de Resoluciones:**

**Sección: Filtros**
- Campo: Número de Resolución (texto, búsqueda parcial)
- Campo: Prefijo (texto, búsqueda parcial)
- Campo: Estado (lista desplegable: Todos, Activa, Inactiva, Vencida, Agotada)
- Botón: "Buscar" o "Aplicar filtros"
- Botón: "Limpiar filtros"

**Sección: Acciones**
- Botón: "Crear Resolución" (prominente)

**Sección: Tabla de Resultados (Paginada)**

Columnas:
- Número de Resolución
- Prefijo
- Estado (con indicador visual: color/badge)
- Acciones:
  - Icono: Ver Detalle (ojo o similar)
  - Toggle/Switch: Cambiar Estado (habilitado solo para Activa/Inactiva)

**Pantalla/Modal - Detalle de Resolución:**

Información mostrada:
- Número de resolución
- Número de formulario DIAN
- Tipo de documento
- Prefijo
- Rango de consecutivos (inicial - final)
- Fechas de vigencia (inicial - final)
- Punto/Sede (si aplica)
- Estado actual
- Fecha de creación
- Usuario que creó

Botones:
- Botón: "Cerrar" o "Volver"

**Mensajes de confirmación:**
- Confirmación de activación
- Confirmación de inactivación
- Mensajes informativos para estados no modificables

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Cambio accidental de estado** | El usuario podría cambiar el estado de una resolución sin querer si el toggle es muy sensible | Alto | Media | Implementar confirmación obligatoria en todos los cambios de estado | Mitigado |
| **Confusión con estados no modificables** | El usuario podría no entender por qué no puede modificar resoluciones Vencidas o Agotadas | Medio | Media | Mostrar tooltips o mensajes informativos claros explicando que estos estados son definitivos | Mitigado |
| **Pérdida de contexto al ver detalle** | El usuario podría perder los filtros aplicados al consultar un detalle | Bajo | Baja | Mantener los filtros activos al regresar del detalle al listado | Mitigado |
| **Listado excesivamente largo** | Clientes con muchas resoluciones podrían experimentar lentitud | Medio | Baja | Implementar paginación y permitir ajustar cantidad de elementos por página | Mitigado |

---

## Notas Adicionales

**Fuera de alcance de esta HU:**
- Gestión automática de cambio a estados Vencida y Agotada (futuras HUs)
- Edición de campos de la resolución (número, prefijo, rangos, fechas, etc.)
- Eliminación física de resoluciones
- Historial de cambios de estado de una resolución
- Exportación del listado de resoluciones

**Estados de resolución:**
1. **Activa**: Habilitada para emisión (puede cambiarse a Inactiva)
2. **Inactiva**: Deshabilitada manualmente (puede cambiarse a Activa)
3. **Vencida**: Fecha final de vigencia alcanzada (definitivo, no modificable)
4. **Agotada**: Todos los consecutivos utilizados (definitivo, no modificable)

**Reglas de negocio clave:**
- Solo los estados Activa e Inactiva pueden ser modificados manualmente
- Los estados Vencida y Agotada son definitivos y no se pueden revertir
- El cambio de estado requiere confirmación explícita del usuario
- Los filtros son opcionales y se pueden combinar
- El orden por defecto prioriza las resoluciones Activas

**Consideraciones de seguridad aplicables:**
- AP-0060: Restringir acceso a funciones críticas solo a usuarios autorizados (solo Administrador de Cliente)
- AP-0022: Registro de eventos de seguridad (consultas, cambios de estado, intentos no permitidos)
- AP-0024: Registro detallado con fecha/hora, usuario, acción, IPs
- AP-0028: Permitir a usuarios consultar su historial de acciones (a través de auditoría)
