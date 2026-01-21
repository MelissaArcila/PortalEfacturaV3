# HU004D - Ubicación de Empresa

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
| **Arquitectura relacionada** | Portal Unificado - Módulo de Administración, Gestión de Clientes, Catálogos Geográficos |

---

## Contexto

Esta HU permite agregar o editar la **ubicación geográfica** de una Empresa existente. La ubicación incluye país, departamento y ciudad (para empresas colombianas), y dirección principal.

Esta información es necesaria para el correcto funcionamiento de facturación electrónica y cumplimiento con requisitos DIAN. Una empresa puede crearse sin ubicación (HU004B) y completarse posteriormente.

---

**Notas:**
- Solo Administradores del Portal pueden agregar/editar ubicación
- La empresa debe existir previamente (creada en HU004B)
- Si País = Colombia, Departamento y Ciudad son obligatorios
- Si País ≠ Colombia, Departamento y Ciudad no aplican
- La dirección principal es siempre obligatoria

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Administrador del Portal** | Quiero agregar la ubicación de una empresa | Para registrar la dirección fiscal necesaria para facturación electrónica |
| Como **Administrador del Portal** | Quiero que el sistema me muestre departamentos y ciudades solo si el país es Colombia | Para evitar confusión y mantener datos geográficos correctos |
| Como **Administrador del Portal** | Quiero editar la ubicación de una empresa | Para actualizar la dirección cuando la empresa cambie su domicilio fiscal |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Acceso a agregar/editar ubicación | Empresa existe, Administrador en vista de detalle | Usuario hace clic en "Agregar Ubicación" o "Editar Ubicación" | Sistema muestra formulario con campos: *País (select con búsqueda, todos los países), *Departamento (select, inicialmente deshabilitado), *Ciudad (select, inicialmente deshabilitado), *Dirección Principal (textarea). Si es edición, campos pre-poblados con datos actuales | ☐ | ☐ | ☐ |
| 2 | Selección de país | Administrador en formulario | Usuario hace clic en campo País | Sistema muestra lista desplegable con todos los países del mundo (tabla maestra Países). Permite búsqueda por texto (typeahead). Campo obligatorio | ☐ | ☐ | ☐ |
| 3 | Selección de Colombia habilita Departamento y Ciudad | Administrador selecciona "Colombia" como país | Usuario selecciona Colombia | Sistema: (1) Habilita campo "Departamento" y lo marca como obligatorio (*Departamento), (2) Carga lista de departamentos de Colombia (tabla maestra Departamentos), (3) Habilita campo "Ciudad" (pero deshabilitado hasta seleccionar departamento) y lo marca como obligatorio (*Ciudad). Si había selección previa de otro país, limpia Departamento y Ciudad | ☐ | ☐ | ☐ |
| 4 | Selección de país diferente a Colombia deshabilita campos | Administrador selecciona país diferente a Colombia (ej: Argentina, México) | Usuario selecciona país no-Colombia | Sistema: (1) Deshabilita campos "Departamento" y "Ciudad", (2) Los marca como NO obligatorios, (3) Limpia cualquier valor previo si existía. Solo País y Dirección son obligatorios | ☐ | ☐ | ☐ |
| 5 | Selección de departamento carga ciudades | Administrador con País=Colombia selecciona un departamento | Usuario selecciona departamento | Sistema carga lista de ciudades del departamento seleccionado (tabla maestra Ciudades filtrada por código de departamento). Habilita campo Ciudad para selección. Si había ciudad seleccionada de otro departamento, la limpia | ☐ | ☐ | ☐ |
| 6 | Cambio de departamento actualiza ciudades | Administrador cambia de un departamento a otro | Usuario cambia departamento | Sistema recarga lista de ciudades con las del nuevo departamento. Limpia selección de ciudad anterior si existía | ☐ | ☐ | ☐ |
| 7 | Ingreso de dirección principal | Administrador ingresa dirección | Usuario escribe en campo Dirección Principal | Sistema acepta caracteres alfanuméricos, espacios, y caracteres especiales comunes en direcciones (#, -, /, números, bis, etc.). Campo obligatorio. Máximo 200 caracteres | ☐ | ☐ | ☐ |
| 8 | Guardado exitoso de ubicación | Administrador completa todos los campos obligatorios correctamente | Usuario hace clic en "Guardar" | Sistema valida campos obligatorios según país seleccionado (si Colombia: País+Depto+Ciudad+Dirección, si otro: País+Dirección), guarda/actualiza en BD, registra en auditoría, muestra mensaje "Ubicación guardada exitosamente", regresa a vista de detalle | ☐ | ☐ | ☐ |
| 9 | Validación de campos obligatorios según país | Administrador intenta guardar sin completar obligatorios | Usuario hace clic en "Guardar" | Sistema valida: Si País=Colombia requiere Depto+Ciudad+Dirección. Si País≠Colombia requiere solo Dirección. Muestra mensaje "Complete todos los campos obligatorios (*)", marca faltantes en rojo, hace scroll al primero | ☐ | ☐ | ☐ |
| 10 | Error técnico durante guardado | Administrador confirma guardado pero ocurre error backend | Sistema intenta guardar pero falla | Sistema muestra mensaje "Ocurrió un error al guardar. Intente nuevamente o contacte a soporte", mantiene datos ingresados, registra error en auditoría (ERROR) | ☐ | ☐ | ☐ |
| 11 | Cancelación de edición | Administrador en formulario | Usuario hace clic en "Cancelar" | Sistema muestra diálogo "¿Cancelar? Los cambios no guardados se perderán". Si confirma: descarta cambios, regresa a vista detalle. Si continúa: cierra diálogo | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 12 | Estructura obligatoria de auditoría | Eventos de agregar/editar ubicación | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (ADMINISTRACION_EMPRESA_UBICACION_*), Fecha/Hora (UTC ISO 8601), Usuario, Cliente (NULL), IP Local, IP Pública, Resultado, Descripción, Severidad, Datos Adicionales | ☐ | ☐ | ☐ |
| 13 | Auditoría de guardado exitoso | Administrador guarda ubicación | Sistema guarda | Registro: Tipo "ADMINISTRACION_EMPRESA_UBICACION_GUARDADA", Resultado "EXITOSO", Severidad "INFO", Descripción "Administrador {usuario} guardó ubicación de empresa {nombre}", Datos Adicionales: {"empresa_id": "UUID", "pais": "Colombia", "departamento": "Antioquia", "ciudad": "Medellín", "direccion": "Calle 10 # 20-30"} | ☐ | ☐ | ☐ |
| 14 | Auditoría de error técnico | Error en backend | Sistema falla | Registro: Tipo "ADMINISTRACION_EMPRESA_UBICACION_ERROR", Resultado "FALLIDO", Severidad "ERROR", Descripción "Error al guardar ubicación", Datos Adicionales: {"empresa_id": "UUID", "error_tipo": "timeout"} | ☐ | ☐ | ☐ |
| 15 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, usuario, evento, resultado. Exportable CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

#### **HU004D-Ubicacion.html** - Formulario de ubicación

**Elementos:**
- Título: "Ubicación - {Nombre Empresa}" (H3)
- Formulario:
  - *País (select con búsqueda, lista completa de países)
  - *Departamento (select, habilitado solo si País=Colombia, lista de departamentos)
  - *Ciudad (select, habilitado solo si País=Colombia y Departamento seleccionado, lista de ciudades del departamento)
  - *Dirección Principal (textarea, max 200 caracteres, permite #, -, /, números, etc.)
- Info box: "Si la empresa está en Colombia, debe seleccionar Departamento y Ciudad. Para otros países, solo se requiere Dirección."
- Botones:
  - "Cancelar" (outlined)
  - "Guardar" (contained gradiente, deshabilitado hasta cumplir obligatorios)

**Estados:**
- País no seleccionado (Depto y Ciudad deshabilitados)
- País = Colombia (Depto y Ciudad habilitados y obligatorios)
- País ≠ Colombia (Depto y Ciudad deshabilitados y no obligatorios)
- Departamento seleccionado (Ciudad habilitada con opciones del departamento)
- Validación exitosa (todos los obligatorios completos)

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Tablas geográficas desactualizadas** | Si departamentos o ciudades no están actualizados, administrador no podrá seleccionar ubicación correcta | Medio | Media | Implementar módulo de mantenimiento de tablas maestras. Actualización trimestral. Permitir soporte agregar valores manualmente si no existen | Mitigado |
| **Caracteres especiales en dirección incompatibles con DIAN** | Caracteres raros en dirección podrían causar rechazo en transmisiones XML a DIAN | Medio | Baja | Permitir solo caracteres compatibles con XML 1.0 estándar DIAN. Validar lista permitida | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance

- Datos de identificación → **HU004B**
- Información complementaria → **HU004C**
- Registros mercantiles → **HU004E**
- Múltiples direcciones (secundarias, sucursales) → HU futura
- Validación de existencia de dirección con API geográfica → HU futura

### Dependencias

**Depende de:**
- **HU004B - Datos Identificación**: Empresa debe existir
- **Tablas Maestras**: Países, Departamentos (Colombia), Ciudades (Colombia)

**Relacionada con:**
- **HU004A - Módulo Administración**: Navegación desde vista de detalle

### Valor Independiente

- ✅ Agrega ubicación necesaria para facturación electrónica
- ✅ Puede agregarse después de crear empresa
- ✅ Soporta empresas nacionales e internacionales
- ✅ Editable en cualquier momento

### Referencias

- **Estándar Auditoría**: Tipos `ADMINISTRACION_EMPRESA_UBICACION_*`
- **Glosario**: País, Departamento, Ciudad, Dirección Principal
- **Guía de Estilos**: Material UI, Primary/Secondary
- **Resoluciones DIAN**: Requisitos de dirección en facturación electrónica

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**HU Original:** HU004 - Creación de Empresas (atomizada en HU004A, B, C, D, E)
