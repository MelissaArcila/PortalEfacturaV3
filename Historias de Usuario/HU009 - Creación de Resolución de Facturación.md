# HU002 - Creación de Resolución de Facturación

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

Las resoluciones de facturación son autorizaciones otorgadas por la DIAN que definen los rangos de numeración, prefijos, fechas de vigencia y tipos de documentos que un cliente puede emitir. Actualmente, el Portal Unificado no cuenta con un módulo que permita al Administrador de Cliente crear y gestionar sus propias resoluciones de facturación.

Esta funcionalidad es crítica para que los clientes puedan configurar de forma autónoma los parámetros necesarios para la emisión de documentos electrónicos, garantizando el cumplimiento de la normativa DIAN y evitando conflictos en la numeración de documentos.

---

**Notas:**
- Esta HU se enfoca exclusivamente en la **creación** de resoluciones.
- La gestión automática de estados (Vencida, Agotada) se manejará en futuras HUs.
- La funcionalidad de gestión de Puntos de Venta/Sedes se manejará en una HU separada.
- La pantalla de listado y edición de resoluciones se define en HU003.

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Administrador de Cliente** | Quiero acceder a un módulo de resoluciones de facturación | Para gestionar las autorizaciones de emisión de documentos de mi cliente |
| Como **Administrador de Cliente** | Quiero crear una nueva resolución de facturación con todos los campos y validaciones requeridas | Para habilitar la emisión de documentos electrónicos según las autorizaciones de la DIAN |
| Como **Administrador de Cliente** | Quiero que el sistema valide automáticamente los datos ingresados | Para prevenir errores y garantizar el cumplimiento de las reglas de negocio |
| Como **Administrador de Cliente** | Quiero recibir confirmación de creación exitosa | Para saber que la resolución fue guardada correctamente y poder crear otra o ir al listado |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Acceso al módulo de resoluciones | El Administrador de Cliente ha iniciado sesión en el Portal Unificado | El usuario navega al módulo de Resoluciones | El sistema muestra la pantalla de gestión de resoluciones (definida en HU003) con la opción "Crear Resolución" visible | ☐ | ☐ | ☐ |
| 2 | Acceso al formulario de creación | El Administrador de Cliente se encuentra en la pantalla de gestión de resoluciones | El usuario hace clic en el botón "Crear Resolución" | El sistema muestra el formulario de creación de resolución con todos los campos vacíos y habilitados según el orden y dependencias definidas | ☐ | ☐ | ☐ |
| 3 | Creación exitosa de resolución con estado activa | El Administrador de Cliente ha completado correctamente todos los campos obligatorios, las validaciones son exitosas y seleccionó estado "Activa" | El usuario hace clic en el botón "Guardar" o "Crear" | El sistema guarda la resolución con estado "Activa", muestra mensaje de confirmación "Resolución creada exitosamente" con opciones "Crear otra resolución" y "Ir a gestión de resoluciones", y registra el evento en auditoría | ☐ | ☐ | ☐ |
| 4 | Creación exitosa de resolución con estado inactiva | El Administrador de Cliente ha completado correctamente todos los campos obligatorios, las validaciones son exitosas y seleccionó estado "Inactiva" | El usuario hace clic en el botón "Guardar" o "Crear" | El sistema guarda la resolución con estado "Inactiva", muestra mensaje de confirmación "Resolución creada exitosamente" con opciones "Crear otra resolución" y "Ir a gestión de resoluciones", y registra el evento en auditoría | ☐ | ☐ | ☐ |
| 5 | Opción "Crear otra resolución" desde confirmación | El sistema muestra la confirmación de creación exitosa | El usuario hace clic en "Crear otra resolución" | El sistema redirige al formulario de creación con todos los campos limpios y vacíos, listo para ingresar una nueva resolución | ☐ | ☐ | ☐ |
| 6 | Opción "Ir a gestión de resoluciones" desde confirmación | El sistema muestra la confirmación de creación exitosa | El usuario hace clic en "Ir a gestión de resoluciones" | El sistema redirige a la pantalla de gestión de resoluciones (HU003) mostrando el listado completo incluyendo la resolución recién creada | ☐ | ☐ | ☐ |
| 7 | Validación de número de resolución obligatorio | El Administrador de Cliente está en el formulario de creación | El usuario intenta guardar sin ingresar número de resolución | El sistema muestra el mensaje "El número de resolución es obligatorio. Por favor ingrese un número de resolución válido" y no permite guardar | ☐ | ☐ | ☐ |
| 8 | Validación de número de resolución único | El Administrador de Cliente ingresa un número de resolución que ya existe para su cliente | El usuario intenta guardar la resolución | El sistema muestra el mensaje "El número de resolución ya existe para este NIT. Por favor ingrese un número de resolución nuevo" y no permite guardar | ☐ | ☐ | ☐ |
| 9 | Validación de tipo de documento obligatorio | El Administrador de Cliente está en el formulario de creación | El usuario intenta guardar sin seleccionar tipo de documento | El sistema muestra el mensaje "El tipo de documento es obligatorio. Por favor seleccione un tipo de documento" y no permite guardar | ☐ | ☐ | ☐ |
| 10 | Validación de prefijo obligatorio | El Administrador de Cliente está en el formulario de creación | El usuario intenta guardar sin ingresar prefijo | El sistema muestra el mensaje "El prefijo es obligatorio. Por favor ingrese un prefijo válido de hasta 4 caracteres" y no permite guardar | ☐ | ☐ | ☐ |
| 11 | Validación de formato de prefijo | El Administrador de Cliente ingresa un prefijo con caracteres especiales o minúsculas | El usuario escribe en el campo prefijo | El sistema convierte automáticamente las letras minúsculas a mayúsculas y solo permite ingresar letras (A-Z) y números (0-9), con un máximo de 4 caracteres, sin permitir caracteres especiales ni espacios | ☐ | ☐ | ☐ |
| 12 | Validación de consecutivo inicial obligatorio | El Administrador de Cliente está en el formulario de creación | El usuario intenta guardar sin ingresar consecutivo inicial | El sistema muestra el mensaje "El consecutivo inicial es obligatorio. Por favor ingrese un número entero positivo" y no permite guardar | ☐ | ☐ | ☐ |
| 13 | Validación de consecutivo final obligatorio | El Administrador de Cliente está en el formulario de creación | El usuario intenta guardar sin ingresar consecutivo final | El sistema muestra el mensaje "El consecutivo final es obligatorio. Por favor ingrese un número entero positivo" y no permite guardar | ☐ | ☐ | ☐ |
| 14 | Validación de rango de consecutivos coherente | El Administrador de Cliente ingresa un consecutivo inicial mayor que el consecutivo final | El usuario intenta guardar la resolución | El sistema muestra el mensaje "El consecutivo inicial debe ser menor o igual al consecutivo final. Por favor verifique los valores ingresados" y no permite guardar | ☐ | ☐ | ☐ |
| 15 | Habilitación de consecutivos solo con prefijo válido | El Administrador de Cliente no ha ingresado un prefijo válido | El usuario intenta escribir en los campos de consecutivo inicial o final | El sistema mantiene deshabilitados los campos de consecutivo inicial y final hasta que se ingrese un prefijo válido | ☐ | ☐ | ☐ |
| 16 | Validación de traslape de rangos | El Administrador de Cliente ingresa un rango de consecutivos (inicial-final) que se traslapa con otra resolución existente del mismo prefijo y tipo de documento | El usuario intenta guardar la resolución | El sistema muestra el mensaje "El rango de consecutivos (X - Y) se traslapa con una resolución existente para el prefijo [PREFIJO] y tipo de documento [TIPO]. Por favor ingrese un rango diferente" y no permite guardar | ☐ | ☐ | ☐ |
| 17 | Validación de fecha inicial de vigencia obligatoria | El Administrador de Cliente está en el formulario de creación | El usuario intenta guardar sin seleccionar fecha inicial de vigencia | El sistema muestra el mensaje "La fecha inicial de vigencia es obligatoria. Por favor seleccione una fecha" y no permite guardar | ☐ | ☐ | ☐ |
| 18 | Validación de fecha final de vigencia obligatoria | El Administrador de Cliente está en el formulario de creación | El usuario intenta guardar sin seleccionar fecha final de vigencia | El sistema muestra el mensaje "La fecha final de vigencia es obligatoria. Por favor seleccione una fecha" y no permite guardar | ☐ | ☐ | ☐ |
| 19 | Validación de coherencia de fechas de vigencia | El Administrador de Cliente ingresa una fecha inicial posterior a la fecha final | El usuario intenta guardar la resolución | El sistema muestra el mensaje "La fecha final de vigencia debe ser mayor o igual a la fecha inicial. Por favor verifique las fechas ingresadas" y no permite guardar | ☐ | ☐ | ☐ |
| 20 | Selección de calendario para fechas | El Administrador de Cliente hace clic en los campos de fecha inicial o final de vigencia | El usuario hace clic en el campo de fecha | El sistema muestra un calendario interactivo que permite seleccionar la fecha de forma visual | ☐ | ☐ | ☐ |
| 21 | Campo Punto/Sede opcional - con puntos disponibles | El Administrador de Cliente tiene puntos de venta o sedes creados para su cliente | El usuario visualiza el formulario de creación | El sistema muestra el campo "Punto/Sede" como opcional con una lista desplegable que contiene todos los puntos de venta o sedes del cliente | ☐ | ☐ | ☐ |
| 22 | Campo Punto/Sede opcional - sin puntos disponibles | El Administrador de Cliente no tiene puntos de venta o sedes creados para su cliente | El usuario visualiza el formulario de creación | El sistema muestra el campo "Punto/Sede" con un mensaje "No hay puntos de venta o sedes creados" y permite continuar sin seleccionar ningún punto | ☐ | ☐ | ☐ |
| 23 | Campo Número de formulario DIAN opcional | El Administrador de Cliente está en el formulario de creación | El usuario deja vacío el campo "Número de formulario DIAN" | El sistema permite guardar la resolución sin este campo, ya que es opcional | ☐ | ☐ | ☐ |
| 24 | Validación de formato de número de formulario DIAN | El Administrador de Cliente ingresa un número de formulario DIAN con letras o caracteres especiales | El usuario escribe en el campo "Número de formulario DIAN" | El sistema solo permite ingresar dígitos numéricos (0-9) con una longitud máxima de 15 caracteres | ☐ | ☐ | ☐ |
| 25 | Lista desplegable de tipos de documento | El Administrador de Cliente hace clic en el campo "Tipo de documento" | El usuario abre la lista desplegable | El sistema muestra las opciones: Factura electrónica, Nota crédito, Nota débito, Factura electrónica de contingencia | ☐ | ☐ | ☐ |
| 26 | Cancelar creación de resolución - solicitar confirmación | El Administrador de Cliente está completando el formulario de creación con datos ingresados | El usuario hace clic en el botón "Cancelar" | El sistema muestra un mensaje de confirmación "¿Está seguro que desea cancelar? Se perderán todos los datos ingresados" con opciones "Sí, cancelar" y "No, continuar editando" | ☐ | ☐ | ☐ |
| 27 | Cancelar creación - confirmación afirmativa | El sistema muestra el mensaje de confirmación de cancelación | El usuario hace clic en "Sí, cancelar" | El sistema descarta todos los datos ingresados y redirige a la pantalla de gestión de resoluciones (HU003) sin guardar ningún cambio | ☐ | ☐ | ☐ |
| 28 | Cancelar creación - confirmación negativa | El sistema muestra el mensaje de confirmación de cancelación | El usuario hace clic en "No, continuar editando" | El sistema cierra el mensaje de confirmación y regresa al formulario de creación manteniendo todos los datos ingresados sin pérdida de información | ☐ | ☐ | ☐ |

### Criterios de Aceptación - Auditoría y Trazabilidad

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 29 | Auditoría de creación exitosa de resolución | Ocurre el escenario 3 o 4 (creación exitosa) | El sistema guarda la resolución exitosamente | El sistema crea un registro de auditoría con: Tipo de evento="RESOLUCION_CREADA", Resultado="EXITOSO", Descripción="Resolución de facturación creada exitosamente", Nivel de severidad="INFO", Datos adicionales: nombre de usuario, NIT del cliente, número de resolución, prefijo, tipo de documento, rango de consecutivos (inicial-final), fechas de vigencia (inicial-final), estado inicial (Activa/Inactiva), punto de venta (si aplica), fecha/hora, IP local, IP pública | ☐ | ☐ | ☐ |
| 30 | Auditoría de intento fallido por validación | Alguna validación de negocio falla (duplicidad, traslape, formato incorrecto, etc.) | El usuario intenta guardar pero el sistema rechaza por validación | El sistema crea un registro de auditoría con: Tipo de evento="RESOLUCION_CREACION_FALLIDA", Resultado="FALLIDO", Descripción="Intento fallido de creación de resolución por validación", Nivel de severidad="WARNING", Datos adicionales: nombre de usuario, NIT del cliente, tipo de validación fallida, datos ingresados (número resolución, prefijo, tipo documento), motivo del rechazo, fecha/hora, IP local, IP pública | ☐ | ☐ | ☐ |
| 31 | Auditoría de cancelación de creación | Ocurre el escenario 27 (cancelación confirmada) | El usuario confirma la cancelación de la creación | El sistema crea un registro de auditoría con: Tipo de evento="RESOLUCION_CREACION_CANCELADA", Resultado="EXITOSO", Descripción="Creación de resolución cancelada por el usuario", Nivel de severidad="INFO", Datos adicionales: nombre de usuario, NIT del cliente, fecha/hora, IP local, IP pública | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes
Pendiente

**Descripción funcional del formulario de creación:**

**Pantalla - Formulario de Creación de Resolución:**

**Sección: Datos Generales**
- Campo: Número de resolución (texto alfanumérico, hasta 30 caracteres, obligatorio)
- Campo: Número de formulario DIAN (numérico, hasta 15 dígitos, opcional)
- Campo: Tipo de documento (lista desplegable, obligatorio)
- Campo: Estado inicial (opciones: Activa / Inactiva, obligatorio)

**Sección: Prefijo y Rango de Consecutivos**
- Campo: Prefijo (texto alfanumérico, hasta 4 caracteres, solo mayúsculas, obligatorio)
- Campo: Consecutivo inicial (número entero positivo, obligatorio, deshabilitado hasta ingresar prefijo válido)
- Campo: Consecutivo final (número entero positivo, obligatorio, deshabilitado hasta ingresar prefijo válido)

**Sección: Vigencia**
- Campo: Fecha inicial de vigencia (calendario, obligatorio)
- Campo: Fecha final de vigencia (calendario, obligatorio)

**Sección: Punto de Venta (Opcional)**
- Campo: Punto/Sede (lista desplegable, opcional)

**Botones:**
- Botón "Guardar" o "Crear"
- Botón "Cancelar"

**Pantalla - Confirmación de Creación Exitosa:**
- Mensaje: "Resolución creada exitosamente"
- Botón: "Crear otra resolución"
- Botón: "Ir a gestión de resoluciones"

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Traslape de rangos no detectado** | Una falla en la validación podría permitir la creación de resoluciones con rangos traslapados, causando conflictos en la emisión de documentos | Alto | Baja | Implementar validación robusta que compare el rango ingresado contra todas las resoluciones existentes del mismo prefijo y tipo de documento, independientemente de su estado | Mitigado |
| **Pérdida de datos sin confirmación** | El usuario podría perder datos ingresados si cierra accidentalmente el formulario | Medio | Media | Implementar confirmación obligatoria al intentar cancelar con datos ingresados | Mitigado |
| **Resoluciones inactivas creadas por error** | El usuario podría crear una resolución con estado "Inactiva" por error cuando la intención era "Activa" | Medio | Media | Mostrar claramente el estado seleccionado y requerir selección explícita del estado | Mitigado |
| **Dependencia de puntos de venta no creados** | Si el cliente no tiene puntos de venta creados, podría generar confusión o bloqueo funcional | Bajo | Media | Mostrar mensaje claro cuando no hay puntos disponibles y permitir creación sin este campo | Mitigado |

---

## Notas Adicionales

**Fuera de alcance de esta HU:**
- Gestión automática de estados Vencida y Agotada (futuras HUs)
- Edición de resoluciones existentes (HU003)
- Listado y consulta de resoluciones (HU003)
- Creación y gestión de Puntos de Venta o Sedes (futura HU)
- Validación contra la DIAN al momento de creación
- Límites de resoluciones activas por cliente

**Tipos de documento soportados:**
1. Factura electrónica
2. Nota crédito
3. Nota débito
4. Factura electrónica de contingencia

**Reglas de negocio clave:**
- Número de resolución + Prefijo + Tipo de documento = Único por cliente
- No permitir traslape de rangos de consecutivos para el mismo prefijo y tipo de documento (independientemente del estado)
- Prefijo: máximo 4 caracteres, solo A-Z y 0-9, conversión automática a mayúsculas
- Consecutivo inicial ≤ Consecutivo final
- Fecha inicial ≤ Fecha final
- Los campos de consecutivos se habilitan solo después de ingresar un prefijo válido

**Consideraciones de seguridad aplicables:**
- AP-0060: Restringir acceso a funciones críticas solo a usuarios autorizados (solo Administrador de Cliente)
- AP-0022: Registro de eventos de seguridad (creación, intentos fallidos, cancelación)
- AP-0024: Registro detallado con fecha/hora, usuario, acción, IPs
