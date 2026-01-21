# HU004B - Datos de Identificación de Empresa

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
| **Arquitectura relacionada** | Portal Unificado - Módulo de Administración, Gestión de Clientes |

---

## Contexto

Esta HU permite crear una **Empresa con datos de identificación mínimos** en el Portal Unificado. Una Empresa representa una entidad comercial (persona natural o jurídica) que utiliza los servicios de facturación electrónica.

Los datos de identificación son el **núcleo mínimo** requerido para registrar una Empresa en el sistema. Una vez creada con estos datos básicos, la Empresa puede completarse posteriormente con información complementaria (HU004C), ubicación (HU004D), y registros mercantiles (HU004E).

Esta HU también permite **editar los datos de identificación** de empresas existentes.

---

**Notas:**
- Solo Administradores del Portal pueden crear/editar empresas
- El Número de Identificación debe ser único en el sistema
- El Dígito de Verificación (DV) se valida con el algoritmo oficial DIAN
- Los campos obligatorios dependen del Tipo de Identificación seleccionado
- Una empresa creada solo con identificación es válida y funcional (puede completarse después)

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Administrador del Portal** | Quiero crear una empresa ingresando sus datos de identificación básicos | Para registrar rápidamente un nuevo cliente en el sistema con la información mínima necesaria |
| Como **Administrador del Portal** | Quiero que el sistema valide el Número de Identificación para evitar duplicados | Para mantener la integridad de los datos y prevenir empresas duplicadas |
| Como **Administrador del Portal** | Quiero que el sistema valide automáticamente el Dígito de Verificación según algoritmo DIAN | Para garantizar que el NIT es válido antes de crear la empresa |
| Como **Administrador del Portal** | Quiero poder editar los datos de identificación de una empresa existente | Para corregir errores o actualizar información de identificación del cliente |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Acceso a creación de empresa | Administrador en HU004A hace clic en "Crear Nueva Empresa" | Usuario navega a creación | Sistema muestra formulario "Crear Empresa - Datos de Identificación" con campos: *Tipo de Identificación (select), *Número de Identificación (input numérico max 10), DV (input 1 dígito), Nombre Abreviado (input texto max 50), Razón Social/Nombres/Apellidos (condicionales). Botones: "Cancelar", "Crear Empresa" (deshabilitado inicialmente) | ☐ | ☐ | ☐ |
| 2 | Selección de Tipo de Identificación | Administrador en formulario | Usuario hace clic en campo Tipo de Identificación | Sistema muestra lista desplegable con opciones: NIT, Cédula de ciudadanía, Pasaporte, Cédula de extranjería, etc. (tabla maestra Tipos Identificación). Permite búsqueda (typeahead). Campo obligatorio | ☐ | ☐ | ☐ |
| 3 | Validación de unicidad de Número de Identificación | Administrador ingresa Número de Identificación | Usuario completa campo y hace blur/enter | Sistema valida en tiempo real: (1) Solo acepta números, max 10 dígitos, (2) Consulta si existe en BD. Si existe: muestra error "Este número de identificación ya existe (Empresa: {nombre_existente})". Si es único: muestra checkmark verde. Botón "Crear" permanece deshabilitado mientras haya error | ☐ | ☐ | ☐ |
| 4 | Validación de Dígito de Verificación | Administrador con Tipo NIT ingresa DV | Usuario completa campo DV y hace blur | Sistema calcula DV correcto usando algoritmo DIAN (multiplicadores 3,7,13,17,19,23,29,37,41,43,47,53,59,67,71 de derecha a izquierda, suma, módulo 11, cálculo residuo). Si DV ingresado coincide: checkmark verde. Si NO coincide: error "DV incorrecto. El DV correcto es: {dv_calculado}" | ☐ | ☐ | ☐ |
| 5 | Campos condicionales según tipo de identificación | Administrador selecciona tipo de identificación | Usuario selecciona NIT o diferente de NIT | Si tipo = NIT: campo "Razón Social" se habilita y es obligatorio (*Razón Social), campos "Nombres" y "Apellidos" se deshabilitan. Si tipo ≠ NIT: campos "Nombres" y "Apellidos" se habilitan y son obligatorios (*Nombres, *Apellidos), campo "Razón Social" se deshabilita. Sistema ajusta validaciones dinámicamente | ☐ | ☐ | ☐ |
| 6 | Creación exitosa de empresa con datos mínimos | Administrador completa campos obligatorios correctamente (Tipo ID, Número ID único, DV válido si NIT, Razón Social o Nombres+Apellidos según tipo) | Usuario hace clic en "Crear Empresa" | Sistema valida todos los campos, crea empresa en BD con estado "Activa", genera ID único, registra fecha/hora y usuario creador, muestra mensaje "¡Empresa creada exitosamente! {Nombre} con {Tipo}-{Número} ha sido registrada", ofrece botones "Ver Empresa", "Agregar Más Información", "Crear Otra Empresa". Registra evento en auditoría | ☐ | ☐ | ☐ |
| 7 | Validación de campos obligatorios incompletos | Administrador intenta crear empresa sin completar obligatorios | Usuario hace clic en "Crear Empresa" con campos faltantes | Sistema valida y muestra mensaje "Complete todos los campos obligatorios (*)", marca campos faltantes en rojo, hace scroll al primer campo incompleto. Botón permanece deshabilitado | ☐ | ☐ | ☐ |
| 8 | Error técnico durante creación | Administrador confirma creación pero ocurre error en backend (timeout, BD error) | Sistema intenta guardar pero falla | Sistema muestra mensaje "Ocurrió un error al crear la empresa. Intente nuevamente o contacte a soporte", mantiene todos los datos ingresados en el formulario, registra error en auditoría con nivel ERROR | ☐ | ☐ | ☐ |
| 9 | Cancelación de creación | Administrador en medio del formulario | Usuario hace clic en "Cancelar" | Sistema muestra diálogo "¿Cancelar creación? Se perderán los datos ingresados" con botones "Continuar Editando" y "Sí, Cancelar". Si confirma: limpia formulario, regresa a HU004A. Si continúa: cierra diálogo y mantiene datos | ☐ | ☐ | ☐ |
| 10 | Edición de datos de identificación de empresa existente | Administrador en vista de detalle de empresa (desde HU004A) hace clic en "Editar Identificación" | Sistema carga formulario de edición | Sistema muestra formulario pre-poblado con datos actuales de la empresa. Número de Identificación NO es editable (solo lectura). Tipo de Identificación, DV, Nombre Abreviado, Razón Social/Nombres/Apellidos son editables. Botón "Guardar Cambios" disponible | ☐ | ☐ | ☐ |
| 11 | Guardado de cambios en edición | Administrador modifica datos y hace clic en "Guardar Cambios" | Sistema valida y guarda | Sistema valida cambios (DV si modificó, obligatorios), actualiza empresa en BD, registra cambio en auditoría, muestra mensaje "Datos actualizados exitosamente", regresa a vista de detalle | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 12 | Estructura obligatoria de auditoría | Eventos de creación/edición de identificación | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (ADMINISTRACION_EMPRESA_IDENTIFICACION_*), Fecha/Hora (UTC ISO 8601), Usuario, Cliente (NULL), IP Local, IP Pública, Resultado (EXITOSO/FALLIDO), Descripción, Severidad (INFO/WARNING/ERROR), Datos Adicionales (JSON) | ☐ | ☐ | ☐ |
| 13 | Auditoría de creación exitosa | Administrador crea empresa | Sistema guarda empresa | Registro: Tipo "ADMINISTRACION_EMPRESA_IDENTIFICACION_CREADA", Resultado "EXITOSO", Severidad "INFO", Descripción "Administrador {usuario} creó empresa {nombre} con {tipo_id}-{numero_id}", Datos Adicionales: {"empresa_id": "UUID", "numero_identificacion": "900123456", "dv": "7", "tipo_identificacion": "NIT", "razon_social": "Empresa XYZ SAS"} | ☐ | ☐ | ☐ |
| 14 | Auditoría de validación NIT duplicado | Administrador ingresa número duplicado | Sistema detecta duplicado | Registro: Tipo "ADMINISTRACION_EMPRESA_IDENTIFICACION_DUPLICADA", Resultado "FALLIDO", Severidad "WARNING", Descripción "Intento de crear empresa con número de identificación duplicado", Datos Adicionales: {"numero_identificacion": "900123456", "empresa_existente_id": "UUID", "empresa_existente_nombre": "Empresa Existente"} | ☐ | ☐ | ☐ |
| 15 | Auditoría de validación DV incorrecto | Administrador ingresa DV incorrecto | Sistema detecta error | Registro: Tipo "ADMINISTRACION_EMPRESA_DV_INCORRECTO", Resultado "FALLIDO", Severidad "WARNING", Descripción "DV incorrecto para NIT {numero}", Datos Adicionales: {"numero_identificacion": "900123456", "dv_ingresado": "5", "dv_correcto": "7"} | ☐ | ☐ | ☐ |
| 16 | Auditoría de edición exitosa | Administrador edita identificación | Sistema guarda cambios | Registro: Tipo "ADMINISTRACION_EMPRESA_IDENTIFICACION_EDITADA", Resultado "EXITOSO", Severidad "INFO", Descripción "Administrador {usuario} editó identificación de empresa {nombre}", Datos Adicionales: {"empresa_id": "UUID", "cambios": {"razon_social_anterior": "ABC", "razon_social_nueva": "ABC SAS"}} | ☐ | ☐ | ☐ |
| 17 | Auditoría de error técnico | Error en backend al crear/editar | Sistema falla | Registro: Tipo "ADMINISTRACION_EMPRESA_IDENTIFICACION_ERROR", Resultado "FALLIDO", Severidad "ERROR", Descripción "Error al crear/editar empresa", Datos Adicionales: {"error_tipo": "timeout", "datos_intentados": "JSON"} | ☐ | ☐ | ☐ |
| 18 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables (no modificables/eliminables). Filtrable por: fechas, usuario, evento, resultado. Exportable CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

#### **HU004B-CrearEmpresaIdentificacion.html** - Formulario de identificación

**Elementos:**
- Título: "Crear Empresa - Datos de Identificación" (H3, Primary Dark)
- Subtítulo: "Información mínima requerida para registrar la empresa" (Body2, grey-600)
- Formulario:
  - *Tipo de Identificación (select con búsqueda, obligatorio)
  - *Número de Identificación (TextField numérico, max 10, validación unicidad con checkmark/error)
  - DV (TextField numérico, 1 dígito, validación algoritmo DIAN, visible solo si tipo seleccionado requiere DV)
  - Nombre Abreviado (TextField, max 50, contador caracteres, opcional)
  - *Razón Social (TextField, visible y obligatorio solo si tipo = NIT)
  - *Nombres (TextField, visible y obligatorio solo si tipo ≠ NIT)
  - *Apellidos (TextField, visible y obligatorio solo si tipo ≠ NIT)
- Info box: "Después de crear la empresa, puede agregar información complementaria, ubicación y registros mercantiles"
- Botones:
  - "Cancelar" (outlined)
  - "Crear Empresa" (contained gradiente, deshabilitado hasta cumplir obligatorios)

**Estados:**
- Formulario con tipo NIT (muestra Razón Social, oculta Nombres/Apellidos)
- Formulario con tipo Cédula (muestra Nombres/Apellidos, oculta Razón Social)
- Error de número duplicado
- Error de DV incorrecto
- Validación exitosa (todos checkmarks verdes)

#### **HU004B-EmpresaCreada.html** - Confirmación de creación

**Elementos:**
- Icono éxito (checkmark verde grande)
- Título: "¡Empresa Creada Exitosamente!"
- Mensaje: "La empresa {Nombre} con {Tipo}-{Número} ha sido registrada"
- Info: "La empresa está activa. Puede agregar más información ahora o después"
- Botones:
  - "Ver Empresa" (outlined)
  - "Agregar Ubicación" (outlined, navega a HU004D)
  - "Agregar Información Complementaria" (outlined, navega a HU004C)
  - "Crear Otra Empresa" (text)

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Error en algoritmo DV** | Implementación incorrecta del algoritmo DIAN causaría validación errónea de NITs | Alto | Baja | Implementar algoritmo exacto según documentación DIAN. Casos de prueba con NITs reales verificados. Validación backend adicional | Mitigado |
| **Duplicación por validación fallida** | Condición de carrera podría crear empresas duplicadas | Alto | Baja | Validación frontend + backend + constraint UNIQUE en BD. Transacciones atómicas | Mitigado |
| **Empresas "incompletas" causan problemas** | Empresas creadas solo con identificación sin ubicación podrían causar errores en otros módulos | Medio | Media | Validar en módulos que requieren ubicación si existe. Mostrar advertencias. Permitir completar datos después. Diseño flexible | Aceptado |

---

## Notas Adicionales

### Fuera de Alcance

- Información complementaria (centros de costos, tipo sociedad, etc.) → **HU004C**
- Ubicación de la empresa → **HU004D**
- Registros mercantiles → **HU004E**
- Inactivación/eliminación de empresas → HU futura
- Importación masiva de empresas → HU futura

### Dependencias

**Depende de:**
- **HU004A - Módulo Administración**: Navegación desde botón "Crear Nueva Empresa"
- **HU001 - Autenticación**: Usuario Administrador autenticado
- **Tablas Maestras**: Tipos de Identificación debe existir y estar poblada

**Es prerequisito para:**
- **HU004C - Información Complementaria**: Requiere empresa existente
- **HU004D - Ubicación**: Requiere empresa existente
- **HU004E - Registros Mercantiles**: Requiere empresa existente

### Valor Independiente

- ✅ Crea empresas funcionales con datos mínimos
- ✅ Valida unicidad de identificación
- ✅ Valida DV según DIAN
- ✅ Permite entregas tempranas sin esperar HUs C, D, E
- ✅ Empresas pueden completarse posteriormente de forma incremental

### Algoritmo DV DIAN (Resumen)

**Multiplicadores (derecha a izquierda):** 3, 7, 13, 17, 19, 23, 29, 37, 41, 43, 47, 53, 59, 67, 71

**Fórmula:**
1. Multiplicar cada dígito por su multiplicador
2. Sumar resultados
3. Dividir entre 11, obtener residuo
4. Si residuo = 0 o 1 → DV = residuo
5. Si residuo > 1 → DV = 11 - residuo

**Ejemplo:** NIT 900123456 → DV = 7

### Referencias

- **Estándar Auditoría**: Tipos `ADMINISTRACION_EMPRESA_IDENTIFICACION_*`
- **Estándar Seguridad APL03**: Validación permisos, integridad datos
- **Glosario**: Empresa, Cliente, NIT, Dígito de Verificación, Administrador del Portal
- **Guía de Estilos**: Material UI, Primary/Secondary, validaciones con feedback visual

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**HU Original:** HU004 - Creación de Empresas (atomizada en HU004A, B, C, D, E)
