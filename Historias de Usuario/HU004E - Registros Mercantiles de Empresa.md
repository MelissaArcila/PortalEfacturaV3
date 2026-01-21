# HU004E - Registros Mercantiles de Empresa

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

Esta HU permite gestionar los **Registros Mercantiles** (domicilios fiscales) de una Empresa. Una empresa puede tener múltiples registros mercantiles en diferentes cámaras de comercio y ubicaciones.

Los registros mercantiles son información complementaria que puede agregarse a una empresa existente de forma independiente y en cualquier momento.

---

**Notas:**
- Solo Administradores del Portal pueden gestionar registros mercantiles
- La empresa debe existir previamente (creada en HU004B)
- Una empresa puede tener 0, 1 o múltiples registros mercantiles
- Cada registro tiene su propia ubicación (puede ser diferente a la ubicación principal de la empresa)
- Los campos obligatorios aplican al agregar cada registro individual

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Administrador del Portal** | Quiero agregar registros mercantiles a una empresa | Para registrar todos los domicilios fiscales y matrículas de cámara de comercio del cliente |
| Como **Administrador del Portal** | Quiero ver la lista de registros mercantiles de una empresa | Para tener visibilidad completa de todos los domicilios fiscales registrados |
| Como **Administrador del Portal** | Quiero eliminar un registro mercantil | Para remover domicilios que ya no están activos o fueron ingresados por error |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Acceso a gestión de registros mercantiles | Empresa existe, Administrador en vista de detalle | Usuario hace clic en "Gestionar Registros Mercantiles" | Sistema muestra pantalla con: título "Registros Mercantiles - {Nombre Empresa}", formulario para agregar registro (campos colapsados/expandibles), tabla de registros existentes (si existen), botón "Volver" | ☐ | ☐ | ☐ |
| 2 | Formulario de agregar registro mercantil | Administrador en pantalla de registros mercantiles | Usuario hace clic en "Agregar Registro Mercantil" o formulario se expande | Sistema muestra formulario con campos: *Matrícula Mercantil (input texto 5-15 caracteres), *Nombre en Registro Mercantil (input texto), *País (select con búsqueda), *Departamento (select, condicional si Colombia), *Ciudad (select, condicional si Colombia), *Dirección (textarea). Botón "Agregar" disponible | ☐ | ☐ | ☐ |
| 3 | Validación de Matrícula Mercantil | Administrador ingresa matrícula | Usuario completa campo Matrícula y hace blur | Sistema valida: (1) Longitud 5-15 caracteres, (2) Solo acepta letras, números, guion (-), barra (/), (3) Elimina espacios extras, (4) Convierte a mayúsculas automáticamente. Si válido: checkmark verde. Si inválido: error "La matrícula debe tener 5-15 caracteres. Solo letras, números, guion y barra" | ☐ | ☐ | ☐ |
| 4 | Campos de ubicación condicionales | Administrador selecciona país | Usuario selecciona país en formulario de registro | Sistema aplica misma lógica que HU004D: Si País=Colombia → habilita y requiere Departamento y Ciudad (carga listas maestras). Si País≠Colombia → deshabilita Departamento y Ciudad. Al cambiar departamento, actualiza lista de ciudades | ☐ | ☐ | ☐ |
| 5 | Agregar registro mercantil a la lista | Administrador completa todos los campos obligatorios del registro | Usuario hace clic en "Agregar" | Sistema valida campos obligatorios del registro (Matrícula, Nombre, País, Depto+Ciudad si Colombia, Dirección), guarda registro en BD asociado a la empresa, agrega fila a tabla de registros existentes mostrando: Matrícula, Nombre, País, Departamento (si aplica), Ciudad (si aplica), Dirección, botón "Eliminar". Limpia formulario para permitir agregar otro. Muestra mensaje "Registro mercantil agregado exitosamente". Registra en auditoría | ☐ | ☐ | ☐ |
| 6 | Visualización de registros mercantiles existentes | Empresa tiene registros mercantiles | Sistema carga pantalla | Sistema muestra tabla con todos los registros mercantiles de la empresa. Columnas: Matrícula Mercantil, Nombre, País, Departamento, Ciudad, Dirección, Acciones (Eliminar). Si hay más de 10, implementa paginación (10 por página) | ☐ | ☐ | ☐ |
| 7 | Eliminar registro mercantil | Administrador hace clic en "Eliminar" junto a un registro | Usuario confirma eliminación | Sistema muestra diálogo "¿Eliminar Registro Mercantil {matrícula}?". Si confirma: elimina registro de BD, quita fila de la tabla, muestra mensaje "Registro mercantil eliminado", registra en auditoría. Si cancela: cierra diálogo sin eliminar | ☐ | ☐ | ☐ |
| 8 | Validación de campos obligatorios del registro | Administrador intenta agregar registro sin completar obligatorios | Usuario hace clic en "Agregar" con campos faltantes | Sistema valida y muestra mensaje "Complete todos los campos obligatorios (*) del registro mercantil", marca campos faltantes en rojo, hace scroll al primero. No agrega el registro | ☐ | ☐ | ☐ |
| 9 | Error técnico al agregar/eliminar registro | Administrador intenta agregar o eliminar pero ocurre error backend | Sistema intenta operación pero falla | Sistema muestra mensaje "Ocurrió un error. Intente nuevamente o contacte a soporte", mantiene datos ingresados (si agregando), registra error en auditoría (ERROR). No modifica tabla de registros | ☐ | ☐ | ☐ |
| 10 | Empresa sin registros mercantiles | Empresa no tiene registros mercantiles | Sistema carga pantalla | Sistema muestra mensaje "No hay registros mercantiles. Agregue el primero completando el formulario" con ilustración/icono. Formulario visible para agregar primer registro | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 11 | Estructura obligatoria de auditoría | Eventos de agregar/eliminar registros mercantiles | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (ADMINISTRACION_EMPRESA_REGISTRO_MERCANTIL_*), Fecha/Hora (UTC ISO 8601), Usuario, Cliente (NULL), IP Local, IP Pública, Resultado, Descripción, Severidad, Datos Adicionales | ☐ | ☐ | ☐ |
| 12 | Auditoría de registro agregado | Administrador agrega registro mercantil | Sistema guarda registro | Registro: Tipo "ADMINISTRACION_EMPRESA_REGISTRO_MERCANTIL_AGREGADO", Resultado "EXITOSO", Severidad "INFO", Descripción "Administrador {usuario} agregó registro mercantil a empresa {nombre}", Datos Adicionales: {"empresa_id": "UUID", "registro_id": "UUID", "matricula": "ABC123456", "nombre_registro": "Sede Principal", "pais": "Colombia", "departamento": "Antioquia", "ciudad": "Medellín", "direccion": "Calle X"} | ☐ | ☐ | ☐ |
| 13 | Auditoría de registro eliminado | Administrador elimina registro mercantil | Sistema elimina registro | Registro: Tipo "ADMINISTRACION_EMPRESA_REGISTRO_MERCANTIL_ELIMINADO", Resultado "EXITOSO", Severidad "INFO", Descripción "Administrador {usuario} eliminó registro mercantil {matricula} de empresa {nombre}", Datos Adicionales: {"empresa_id": "UUID", "registro_id": "UUID", "matricula": "ABC123456"} | ☐ | ☐ | ☐ |
| 14 | Auditoría de error técnico | Error en backend al agregar/eliminar | Sistema falla | Registro: Tipo "ADMINISTRACION_EMPRESA_REGISTRO_MERCANTIL_ERROR", Resultado "FALLIDO", Severidad "ERROR", Descripción "Error al gestionar registro mercantil", Datos Adicionales: {"empresa_id": "UUID", "operacion": "agregar|eliminar", "error_tipo": "timeout"} | ☐ | ☐ | ☐ |
| 15 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, usuario, evento, resultado. Exportable CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

#### **HU004E-RegistrosMercantiles.html** - Gestión de registros

**Elementos:**
- Título: "Registros Mercantiles - {Nombre Empresa}" (H3)
- Botón "Agregar Registro Mercantil" (contained gradiente, expandible/colapsable)
- **Formulario de agregar registro** (expandible):
  - *Matrícula Mercantil (TextField, 5-15 caracteres, validación, normalización automática a mayúsculas)
  - *Nombre en Registro Mercantil (TextField)
  - *País (select con búsqueda)
  - *Departamento (select, condicional Colombia)
  - *Ciudad (select, condicional Colombia y según departamento)
  - *Dirección (textarea, max 200 caracteres)
  - Botón "Agregar" (outlined, ícono +)
  - Botón "Limpiar" (text)
- **Tabla de registros existentes:**
  - Columnas: Matrícula, Nombre, País, Departamento, Ciudad, Dirección, Acciones (Eliminar)
  - Paginación si > 10 registros
  - Estado vacío: "No hay registros mercantiles. Agregue el primero"
- Botón "Volver" (text, regresa a vista de detalle)

**Estados:**
- Sin registros (estado vacío)
- Con 3-4 registros (tabla poblada)
- Formulario expandido (agregando registro)
- País Colombia (Depto y Ciudad habilitados)
- País no-Colombia (Depto y Ciudad deshabilitados)
- Validación exitosa
- Errores de validación

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Registros sin límite afectan rendimiento** | Permitir cientos de registros mercantiles podría afectar rendimiento de carga | Bajo | Baja | Establecer límite razonable (ej: max 20-30 registros por empresa). Implementar paginación | Aceptado con límite |
| **Formatos de matrícula inconsistentes** | Diferentes cámaras de comercio pueden tener formatos diferentes de matrícula | Bajo | Media | Normalización a mayúsculas y eliminación de espacios. Permitir formato flexible (letras, números, guion, barra). Validar solo longitud | Aceptado |

---

## Notas Adicionales

### Fuera de Alcance

- Datos de identificación → **HU004B**
- Información complementaria → **HU004C**
- Ubicación principal → **HU004D**
- Edición de registros mercantiles existentes → HU futura (actualmente solo agregar/eliminar)
- Validación de existencia de matrícula con API de cámara de comercio → HU futura

### Dependencias

**Depende de:**
- **HU004B - Datos Identificación**: Empresa debe existir
- **Tablas Maestras**: Países, Departamentos (Colombia), Ciudades (Colombia)

**Relacionada con:**
- **HU004A - Módulo Administración**: Navegación desde vista de detalle
- **HU004D - Ubicación**: Usa misma lógica de ubicación geográfica condicional

### Valor Independiente

- ✅ Agrega domicilios fiscales complementarios
- ✅ Opcional (empresa puede tener 0 registros)
- ✅ Puede agregarse en cualquier momento
- ✅ Gestionable de forma independiente

### Referencias

- **Estándar Auditoría**: Tipos `ADMINISTRACION_EMPRESA_REGISTRO_MERCANTIL_*`
- **Glosario**: Registro Mercantil, Matrícula Mercantil, Domicilio Fiscal
- **Guía de Estilos**: Material UI, Primary/Secondary
- **Resoluciones DIAN**: Requisitos de domicilios fiscales en facturación electrónica

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**HU Original:** HU004 - Creación de Empresas (atomizada en HU004A, B, C, D, E)
