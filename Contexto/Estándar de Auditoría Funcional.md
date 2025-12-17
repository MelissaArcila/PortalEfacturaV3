# Estándar de Auditoría Funcional

## Propósito

Este documento define los lineamientos funcionales para el registro de auditoría en todas las funcionalidades del Portal Unificado. El objetivo es garantizar trazabilidad completa, capacidad de investigación de incidentes de seguridad, detección de patrones de comportamiento anómalo y cumplimiento con los requisitos de seguridad definidos en APL03.

---

## Principios Generales

1. **Inmutabilidad**: Los registros de auditoría no pueden ser modificados ni eliminados una vez creados.
2. **Completitud**: Todos los eventos de seguridad, acceso a información confidencial y operaciones críticas deben ser auditados.
3. **Trazabilidad**: Debe ser posible reconstruir la secuencia completa de eventos relacionados con cualquier acción u operación.
4. **No Repudio**: Los registros deben permitir identificar de forma inequívoca quién realizó cada acción y cuándo.
5. **Disponibilidad**: Los registros deben estar disponibles para consulta por usuarios autorizados durante el período definido por la regulación.

---

## Estructura Obligatoria de un Registro de Auditoría

Cada registro de auditoría debe contener **obligatoriamente** los siguientes campos:

| Campo | Descripción | Obligatorio | Ejemplo |
|-------|-------------|-------------|---------|
| **ID Único del Evento** | Identificador único e irrepetible del registro de auditoría | Sí | `AUD-2025-001234567` |
| **Tipo de Evento** | Clasificación del evento auditado en formato estándar | Sí | `AUTENTICACION_EXITOSA`, `DOCUMENTO_CONSULTADO`, `CONFIGURACION_MODIFICADA` |
| **Fecha y Hora** | Momento exacto de ocurrencia con milisegundos y zona horaria | Sí | `2025-12-17 14:35:22.456 UTC-5` |
| **Usuario** | Identificación del usuario que realiza la acción (nombre de usuario, email o identificador único) | Sí | `juan.perez@empresa.com` |
| **Cliente (NIT)** | NIT del cliente bajo cuyo contexto se realizó la acción | Condicional* | `900123456-7` |
| **Cliente (Nombre)** | Nombre del cliente bajo cuyo contexto se realizó la acción | Condicional* | `Empresa ABC S.A.S.` |
| **IP Local** | Dirección IP local desde donde se originó la acción | Sí | `192.168.1.45` |
| **IP Pública** | Dirección IP pública desde donde se originó la acción | Sí | `181.48.235.12` |
| **Resultado** | Resultado de la operación | Sí | `EXITOSO`, `FALLIDO` |
| **Descripción** | Descripción clara y concisa del evento auditado | Sí | `Usuario autenticado exitosamente e ingresó al sistema` |
| **Nivel de Severidad** | Clasificación de la severidad del evento | Sí | `INFO`, `WARNING`, `ERROR` |
| **Datos Adicionales** | Información contextual específica del tipo de evento | No | `{"intentos_fallidos": 3, "modulo": "Facturación"}` |

**\*Condicional:** El Cliente (NIT y Nombre) es obligatorio cuando la acción se realiza bajo el contexto de un cliente específico. No aplica para eventos previos a la selección de cliente (ej.: validación de credenciales).

---

## Niveles de Severidad

Los niveles de severidad permiten clasificar la criticidad de los eventos auditados:

| Nivel | Descripción | Cuándo usar | Ejemplos |
|-------|-------------|-------------|----------|
| **INFO** | Eventos normales y exitosos del sistema | Operaciones completadas exitosamente, acciones rutinarias | Login exitoso, Consulta de documento, Generación de reporte |
| **WARNING** | Eventos que requieren atención pero no representan un error crítico | Intentos fallidos, accesos denegados, situaciones anómalas que no comprometen el sistema | Credenciales incorrectas, Intento de acceso a recurso no autorizado, Usuario inactivo intenta acceder |
| **ERROR** | Eventos críticos de seguridad o fallas que comprometen operaciones | Bloqueos de cuenta, errores críticos, eventos que requieren intervención inmediata | Cuenta bloqueada, Falla en validación DIAN, Error crítico en proceso de negocio |

---

## Tipos de Eventos que Deben ser Auditados

Todas las Historias de Usuario deben considerar la auditoría de los siguientes tipos de eventos:

### 1. Eventos de Seguridad (Obligatorios)
- Autenticación exitosa y fallida
- Bloqueos y desbloqueos de cuentas
- Cambios de contraseña
- Intentos de acceso no autorizado
- Modificación de permisos y roles
- Cierre de sesión (explícito o por timeout)

### 2. Acceso a Información Confidencial (Obligatorios)
- Consulta de documentos electrónicos
- Descarga de archivos (XML, PDF, AttachedDocument)
- Visualización de información de clientes
- Consulta de datos financieros o sensibles
- Acceso a información de terceros

### 3. Operaciones Críticas de Negocio (Obligatorios)
- Emisión de documentos electrónicos
- Transmisión a la DIAN
- Generación de eventos RADIAN
- Modificaciones de configuraciones críticas
- Activación/desactivación de clientes o usuarios
- Reenvío de correos electrónicos
- Anulación o corrección de documentos

### 4. Eventos Excepcionales (Obligatorios)
- Errores en procesos críticos
- Timeouts o fallas de conexión con sistemas externos (DIAN, servicios internos)
- Validaciones DIAN rechazadas
- Intentos de operaciones no permitidas

### 5. Eventos de Administración (Obligatorios)
- Creación, modificación o eliminación de usuarios
- Cambios en configuraciones del sistema
- Parametrizaciones y ajustes operativos
- Gestión de resoluciones de facturación

---

## Requisitos Funcionales de Consulta

El sistema debe proporcionar capacidad de consulta de registros de auditoría con los siguientes filtros:

### Filtros Básicos (Obligatorios)
- **Rango de fechas**: Desde fecha/hora hasta fecha/hora
- **Usuario**: Por nombre de usuario exacto o coincidencia parcial
- **Tipo de evento**: Por tipo específico o categoría
- **Resultado**: Exitoso, Fallido o Ambos
- **Nivel de severidad**: INFO, WARNING, ERROR o combinaciones

### Filtros Avanzados (Recomendados)
- **Cliente**: Por NIT o nombre del cliente
- **IP origen**: Por dirección IP local o pública exacta o rango
- **Módulo o funcionalidad**: Por área funcional específica
- **Texto libre**: Búsqueda en descripción del evento

### Capacidades de Visualización
- **Orden cronológico**: Más reciente primero o más antiguo primero
- **Agrupación**: Por usuario, por tipo de evento, por cliente
- **Exportación**: Posibilidad de exportar resultados de consulta para análisis externo
- **Detalle completo**: Visualización de todos los campos del registro incluyendo datos adicionales

---

## Requisitos de Retención

Los registros de auditoría deben cumplir con los siguientes requisitos de retención:

1. **Período mínimo**: Los registros deben conservarse por el período mínimo definido por la regulación vigente aplicable al ecosistema de facturación electrónica DIAN.

2. **Accesibilidad**: Durante todo el período de retención, los registros deben estar accesibles para consulta por usuarios autorizados.

3. **Integridad**: El sistema debe garantizar que los registros mantengan su integridad durante todo el período de retención.

4. **Respaldo**: Los registros de auditoría deben incluirse en los procesos de respaldo y recuperación del sistema.

---

## Requisitos de Seguridad

1. **Control de acceso**: Solo usuarios con permisos específicos de auditoría pueden consultar los registros.

2. **Auditoría de la auditoría**: Las consultas a los registros de auditoría también deben ser auditadas, registrando quién consultó qué información y cuándo.

3. **Protección de información sensible**: Los registros no deben contener contraseñas, tokens completos u otra información altamente sensible que pueda ser utilizada para comprometer la seguridad.

4. **Segregación de responsabilidades**: Los usuarios que realizan operaciones no deben tener la capacidad de modificar o eliminar sus propios registros de auditoría.

---

## Formato Estándar de Tipo de Evento

Para mantener consistencia, los tipos de evento deben seguir esta convención de nomenclatura:

**Formato:** `<MODULO>_<ENTIDAD>_<ACCION>_<RESULTADO_OPCIONAL>`

**Ejemplos:**
- `AUTENTICACION_USUARIO_EXITOSA`
- `AUTENTICACION_CREDENCIALES_FALLIDAS`
- `DOCUMENTO_FACTURA_CONSULTADO`
- `DOCUMENTO_FACTURA_EMITIDO`
- `CONFIGURACION_USUARIO_MODIFICADA`
- `CUENTA_USUARIO_BLOQUEADA`
- `RADIAN_EVENTO_TRANSMITIDO`
- `DIAN_VALIDACION_RECHAZADA`

---

## Uso en Historias de Usuario

Al crear una Historia de Usuario que involucre cualquiera de los eventos listados anteriormente, se deben:

1. **Incluir criterios de aceptación específicos** que definan qué eventos deben ser auditados y con qué estructura.

2. **Definir el tipo de evento** específico usando la nomenclatura estándar.

3. **Especificar el nivel de severidad** apropiado para cada tipo de evento.

4. **Identificar datos adicionales** específicos del contexto que deben ser registrados.

5. **Garantizar cumplimiento** con todos los campos obligatorios de la estructura estándar.

### Plantilla de Criterio de Aceptación para Auditoría

```markdown
| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| X | Auditoría de [nombre del evento] | Ocurre [condición o escenario] | [Acción del usuario o sistema] | El sistema crea un registro de auditoría con: Tipo de evento="[TIPO_EVENTO]", Resultado="[EXITOSO/FALLIDO]", Descripción="[descripción clara]", Nivel de severidad="[INFO/WARNING/ERROR]", Datos adicionales: [lista de campos específicos del contexto] | ☐ | ☐ | ☐ |
```

---

## Lineamientos de Seguridad Aplicables (APL03)

Este estándar de auditoría funcional está alineado con los siguientes lineamientos del documento APL03:

- **AP-0022**: Registro de eventos excepcionales, de seguridad y acceso a información confidencial
- **AP-0023**: Registro de nivel de severidad para cada evento
- **AP-0024**: Registro de momento exacto con fecha, hora, milisegundos, zona horaria y datos de contexto
- **AP-0025**: Los logs no deben ser modificables o alterables
- **AP-0026**: Almacenamiento por el tiempo que disponga la regulación vigente
- **AP-0027**: Gestión de logs por sistema operativo o sistema externo (no parte de la HU funcional)
- **AP-0028**: Permitir a usuarios consultar su historial de acciones

---

## Notas Finales

- Este documento define requisitos **funcionales** de auditoría. Las decisiones sobre almacenamiento, indexación, herramientas y tecnologías específicas son responsabilidad del equipo de arquitectura.

- Todas las Historias de Usuario que involucren eventos auditables deben hacer referencia explícita a este estándar.

- Este documento debe evolucionar conforme se identifiquen nuevos tipos de eventos o requisitos regulatorios adicionales.

---

**Última actualización:** 2025-12-17
**Versión:** 1.0
