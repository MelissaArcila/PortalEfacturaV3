# HU004C - Información Complementaria de Empresa

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

Esta HU permite agregar o editar **información complementaria** de una Empresa existente en el sistema. Esta información incluye datos tributarios, tipo de sociedad, centros de costos, y datos de contacto necesarios para operación completa.

Una empresa puede ser creada inicialmente solo con datos de identificación (HU004B) y posteriormente completarse con esta información complementaria de forma independiente.

---

**Notas:**
- Solo Administradores del Portal pueden agregar/editar información complementaria
- La empresa debe existir previamente (creada en HU004B)
- Los campos obligatorios son necesarios para habilitar funcionalidades de facturación
- La información puede agregarse inmediatamente después de crear la empresa o en cualquier momento posterior

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Administrador del Portal** | Quiero agregar información complementaria a una empresa | Para completar los datos tributarios y operativos necesarios para facturación |
| Como **Administrador del Portal** | Quiero agregar centros de costos a la empresa | Para permitir la organización contable interna del cliente |
| Como **Administrador del Portal** | Quiero editar la información complementaria de una empresa | Para actualizar datos tributarios o de contacto cuando cambien |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Acceso a agregar información complementaria | Empresa existe (creada en HU004B), Administrador en vista de detalle | Usuario hace clic en "Agregar Información Complementaria" o "Editar Información Complementaria" | Sistema muestra formulario con campos: *Tipo de Sociedad (select con búsqueda), Centros de Costos (formulario repetible), *Tipo de Contribuyente (select: Natural/Jurídico), *Tipo de Régimen (select: Ninguno/Simplificado/Común), *Correo Electrónico Emisor (input email), Actividad Económica (textarea max 200). Si es edición, campos pre-poblados con datos actuales | ☐ | ☐ | ☐ |
| 2 | Selección de Tipo de Sociedad | Administrador en formulario | Usuario hace clic en campo Tipo de Sociedad | Sistema muestra lista desplegable con opciones de tabla maestra: Sociedad anónima, SAS, Limitada, Unipersonal, Cooperativa, etc. Permite búsqueda por texto. Campo obligatorio | ☐ | ☐ | ☐ |
| 3 | Agregar Centro de Costos | Administrador en sección Centros de Costos | Usuario ingresa valor numérico (max 5 dígitos) y hace clic en "Agregar" | Sistema valida: (1) Solo números, (2) Max 5 dígitos, (3) No vacío. Si válido: agrega a tabla visible (Centro de Costo, Acción Eliminar), limpia campo de entrada. Si inválido: muestra error "Ingrese un valor numérico de hasta 5 dígitos" | ☐ | ☐ | ☐ |
| 4 | Eliminar Centro de Costos | Administrador hace clic en "Eliminar" junto a un centro de costos | Usuario confirma eliminación | Sistema muestra diálogo "¿Eliminar Centro de Costos {número}?". Si confirma: elimina de la tabla. Si cancela: cierra diálogo | ☐ | ☐ | ☐ |
| 5 | Selección de Tipo de Contribuyente y Régimen | Administrador selecciona valores | Usuario selecciona en dropdowns | Tipo de Contribuyente: opciones "Natural" o "Jurídico". Tipo de Régimen: opciones "Ninguno (NA)", "Simplificado (No responsable IVA - 49)", "Común (Impuesto ventas - 48)". Ambos obligatorios | ☐ | ☐ | ☐ |
| 6 | Validación de Correo Electrónico Emisor | Administrador ingresa correo | Usuario completa campo y hace blur | Sistema valida formato de email (debe contener @, dominio válido). Si válido: checkmark verde. Si inválido: error "Ingrese un correo electrónico válido (ejemplo: empresa@dominio.com)". Campo obligatorio | ☐ | ☐ | ☐ |
| 7 | Guardado exitoso de información complementaria | Administrador completa todos los campos obligatorios correctamente | Usuario hace clic en "Guardar" | Sistema valida campos obligatorios, guarda/actualiza información en BD, registra cambio en auditoría, muestra mensaje "Información complementaria guardada exitosamente", regresa a vista de detalle de empresa. Información ahora visible en detalle | ☐ | ☐ | ☐ |
| 8 | Validación de campos obligatorios | Administrador intenta guardar sin completar obligatorios | Usuario hace clic en "Guardar" | Sistema valida y muestra mensaje "Complete todos los campos obligatorios (*)", marca campos faltantes en rojo, hace scroll al primero. No permite guardar | ☐ | ☐ | ☐ |
| 9 | Error técnico durante guardado | Administrador confirma guardado pero ocurre error backend | Sistema intenta guardar pero falla | Sistema muestra mensaje "Ocurrió un error al guardar. Intente nuevamente o contacte a soporte", mantiene datos ingresados, registra error en auditoría (ERROR) | ☐ | ☐ | ☐ |
| 10 | Cancelación de edición | Administrador en formulario | Usuario hace clic en "Cancelar" | Sistema muestra diálogo "¿Cancelar? Los cambios no guardados se perderán" con "Continuar Editando" y "Sí, Cancelar". Si confirma: descarta cambios, regresa a vista detalle. Si continúa: cierra diálogo | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 11 | Estructura obligatoria de auditoría | Eventos de agregar/editar información complementaria | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (ADMINISTRACION_EMPRESA_INFO_COMPLEMENTARIA_*), Fecha/Hora (UTC ISO 8601), Usuario, Cliente (NULL), IP Local, IP Pública, Resultado, Descripción, Severidad, Datos Adicionales | ☐ | ☐ | ☐ |
| 12 | Auditoría de guardado exitoso | Administrador guarda información complementaria | Sistema guarda | Registro: Tipo "ADMINISTRACION_EMPRESA_INFO_COMPLEMENTARIA_GUARDADA", Resultado "EXITOSO", Severidad "INFO", Descripción "Administrador {usuario} guardó información complementaria de empresa {nombre}", Datos Adicionales: {"empresa_id": "UUID", "tipo_sociedad": "SAS", "centros_costos_count": 3, "tipo_contribuyente": "Jurídico", "tipo_regimen": "Común", "email_emisor": "correo@empresa.com"} | ☐ | ☐ | ☐ |
| 13 | Auditoría de error técnico | Error en backend | Sistema falla | Registro: Tipo "ADMINISTRACION_EMPRESA_INFO_COMPLEMENTARIA_ERROR", Resultado "FALLIDO", Severidad "ERROR", Descripción "Error al guardar información complementaria", Datos Adicionales: {"empresa_id": "UUID", "error_tipo": "timeout"} | ☐ | ☐ | ☐ |
| 14 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, usuario, evento, resultado. Exportable CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

#### **HU004C-InformacionComplementaria.html** - Formulario

**Elementos:**
- Título: "Información Complementaria - {Nombre Empresa}" (H3)
- Formulario:
  - *Tipo de Sociedad (select con búsqueda)
  - **Centros de Costos:**
    - Input numérico (max 5 dígitos) + botón "Agregar" (outlined, ícono +)
    - Tabla: Centro de Costo, Acción Eliminar
    - Diálogo de confirmación para eliminar
  - *Tipo de Contribuyente (select: Natural/Jurídico)
  - *Tipo de Régimen (select: Ninguno NA / Simplificado 49 / Común 48)
  - *Correo Electrónico Emisor (TextField email, validación formato)
  - Actividad Económica (textarea, max 200 caracteres, contador, opcional)
- Botones:
  - "Cancelar" (outlined)
  - "Guardar" (contained gradiente, deshabilitado hasta cumplir obligatorios)

**Estados:**
- Formulario vacío (primera vez)
- Formulario con datos (edición)
- Centros de costos agregados (3-4 items)
- Validación exitosa
- Errores de validación

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Email inválido causa pérdida de notificaciones** | Si el email no se valida correctamente, notificaciones de facturación no llegarán | Medio | Baja | Validación formato frontend + backend. Considerar verificación por correo de prueba | Mitigado |
| **Centros de costos sin límite** | Agregar cientos de centros de costos podría afectar rendimiento | Bajo | Baja | Establecer límite razonable (ej: max 50 centros por empresa) | Aceptado |

---

## Notas Adicionales

### Fuera de Alcance

- Datos de identificación → **HU004B**
- Ubicación → **HU004D**
- Registros mercantiles → **HU004E**
- Validación de email por envío de correo → HU futura

### Dependencias

**Depende de:**
- **HU004B - Datos Identificación**: Empresa debe existir
- **Tablas Maestras**: Tipos de Sociedad

**Relacionada con:**
- **HU004A - Módulo Administración**: Navegación desde vista de detalle

### Valor Independiente

- ✅ Agrega información tributaria necesaria para facturación
- ✅ Puede agregarse después de crear empresa
- ✅ Puede omitirse si no se requiere inmediatamente
- ✅ Editable en cualquier momento

### Referencias

- **Estándar Auditoría**: Tipos `ADMINISTRACION_EMPRESA_INFO_COMPLEMENTARIA_*`
- **Glosario**: Tipo de Sociedad, Tipo de Contribuyente, Tipo de Régimen, Centro de Costos
- **Guía de Estilos**: Material UI, Primary/Secondary

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**HU Original:** HU004 - Creación de Empresas (atomizada en HU004A, B, C, D, E)
