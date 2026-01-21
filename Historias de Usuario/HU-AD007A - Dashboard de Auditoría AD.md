# HU-AD007A - Dashboard de Auditoría AD

## Información General

| Campo | Descripción |
|-------|-------------|
| **Release objetivo** | Release 2 |
| **Epic** | Integración con Active Directory mediante SCIM |
| **Estado** | BORRADOR |
| **Propietario** | Por definir |
| **Arquitecto** | Por definir |
| **Desarrolladores** | Por definir |
| **Tester** | Por definir |
| **Incidentes relacionados** | N/A |
| **Arquitectura relacionada** | Portal Unificado - Auditoría, Compliance, Reporting |

---

## Contexto

Esta HU implementa un **dashboard centralizado de auditoría** para todos los eventos relacionados con la integración Active Directory, permitiendo a Administradores del Portal consultar, filtrar y exportar logs de:
- Autenticación SAML
- Sincronización SCIM
- Gestión de sesiones
- Invalidación proactiva
- Cambios en configuración AD
- Errores y eventos de seguridad

Es esencial para cumplimiento, debugging, análisis de seguridad y auditorías regulatorias.

---

**Notas:**
- Solo Administradores del Portal pueden acceder
- Logs son inmutables (no editables ni eliminables)
- Retención: mínimo 90 días (configurable)
- Soporta exportación masiva para auditorías externas

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Administrador del Portal** | Quiero consultar todos los eventos de auditoría AD en un dashboard centralizado | Para tener visibilidad completa de actividad de integración AD |
| Como **Auditor Externo** | Quiero exportar logs filtrados por período y tenant | Para cumplimiento regulatorio y auditorías periódicas |
| Como **Administrador de Seguridad** | Quiero filtrar eventos por severidad CRITICAL/ERROR | Para identificar incidentes de seguridad rápidamente |
| Como **Administrador del Portal** | Quiero buscar eventos por usuario o tenant específico | Para investigar actividad sospechosa o problemas reportados |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Acceso a dashboard de auditoría AD | Administrador navega a "Auditoría AD" | Usuario accede | Sistema valida permisos, muestra dashboard con: Título "Auditoría de Integración AD", Filtros (Tenant, Fecha Desde/Hasta, Tipo Evento, Severidad, Usuario), Métricas resumen (Total Eventos, Eventos CRITICAL, Eventos ERROR), Tabla de eventos (últimos 100), Paginación, Botón "Exportar". Si no es admin: HTTP 403 "No tiene permisos" | ☐ | ☐ | ☐ |
| 2 | Tabla de eventos de auditoría | Dashboard se carga | Sistema consulta logs | Sistema muestra tabla con columnas: Timestamp (UTC, formato ISO), Tipo Evento, Usuario (o "Sistema"), Tenant, Resultado (EXITOSO/FALLIDO), Severidad (badge con color: INFO=azul, WARNING=amarillo, ERROR=naranja, CRITICAL=rojo), Descripción (truncada 100 chars), Acciones (botón "Ver Detalle"). Query: `SELECT * FROM audit_logs WHERE tipo_evento LIKE 'INTEGRACION_AD_%' ORDER BY fecha DESC LIMIT 100`. Paginación: 100 registros por página | ☐ | ☐ | ☐ |
| 3 | Filtro por tenant | Administrador selecciona tenant específico | Usuario aplica filtro | Sistema filtra tabla: `WHERE tenant_id = {selected_tenant}`. Actualiza métricas resumen solo para ese tenant. Muestra nombre del tenant en encabezado. Botón "Limpiar Filtros" visible | ☐ | ☐ | ☐ |
| 4 | Filtro por rango de fechas | Administrador selecciona "Última semana" o rango personalizado | Usuario aplica filtro | Sistema filtra: `WHERE fecha >= {fecha_desde} AND fecha <= {fecha_hasta}`. DatePicker permite selección personalizada. Presets: "Hoy", "Ayer", "Última semana", "Último mes", "Personalizado" | ☐ | ☐ | ☐ |
| 5 | Filtro por tipo de evento | Administrador selecciona categoría | Usuario aplica filtro | Select con opciones agrupadas: **Autenticación**: SAML_LOGIN_EXITOSO, SAML_FIRMA_INVALIDA, etc.; **Sincronización**: SYNC_INICIAL_COMPLETADA, SYNC_DELTA_USUARIO_CREADO, etc.; **Sesiones**: SESION_CREADA, SESION_INVALIDADA, etc.; **Configuración**: CONFIGURACION_CREADA, TOKEN_REGENERADO, etc. Filtra: `WHERE tipo_evento = {seleccionado}` o `WHERE tipo_evento LIKE '{categoría}%'` | ☐ | ☐ | ☐ |
| 6 | Filtro por severidad | Administrador selecciona "Solo CRITICAL y ERROR" | Usuario aplica filtro | Checkbox multi-select: INFO, WARNING, ERROR, CRITICAL. Filtra: `WHERE severidad IN ({seleccionados})`. Útil para ver solo incidentes: CRITICAL + ERROR | ☐ | ☐ | ☐ |
| 7 | Búsqueda por usuario o descripción | Administrador ingresa texto en buscador | Usuario busca | Campo de búsqueda con debounce (500ms). Busca en: `WHERE usuario ILIKE '%{texto}%' OR descripcion ILIKE '%{texto}%'`. Ejemplo: buscar "juan@empresa.com" muestra todos los eventos de ese usuario | ☐ | ☐ | ☐ |
| 8 | Vista de detalle de evento | Administrador hace clic en "Ver Detalle" | Sistema muestra modal | Modal "Detalle de Evento de Auditoría" con: ID Evento (UUID, copiable), Tipo Evento, Timestamp (local + UTC), Usuario (nombre completo + email, o "Sistema"), Tenant (nombre completo), Cliente (NULL o nombre), IP Pública, Resultado, Severidad, Descripción completa, **Datos Adicionales** (JSON formateado con syntax highlighting). Botón "Copiar JSON" (copia datos_adicionales), Botón "Cerrar" | ☐ | ☐ | ☐ |
| 9 | Métricas resumen | Dashboard muestra estadísticas | Sistema consulta | Cards con: **Total Eventos** (período filtrado), **Eventos CRITICAL** (último 24h, alerta si >0), **Eventos ERROR** (último 24h), **Logins SAML (24h)**, **Sincronizaciones (24h)**. Se actualizan al aplicar filtros | ☐ | ☐ | ☐ |
| 10 | Exportación de logs filtrados | Administrador hace clic en "Exportar" | Sistema genera archivo | Sistema genera CSV con TODOS los eventos que cumplen filtros actuales (sin límite de 100): Columnas: Timestamp, Tipo Evento, Usuario, Tenant, Cliente, IP, Resultado, Severidad, Descripción, Datos Adicionales (JSON compacto). Descarga archivo `auditoria_ad_{fecha}.csv`. Registra exportación en auditoría | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 11 | Estructura obligatoria de auditoría | Eventos de acceso y exportación | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (AUDITORIA_AD_*), Fecha/Hora (UTC ISO 8601), Usuario (admin), Cliente (NULL), IP Local (NULL), IP Pública (IP admin), Resultado, Descripción, Severidad, Datos Adicionales | ☐ | ☐ | ☐ |
| 12 | Auditoría de acceso al dashboard | Admin accede a dashboard | Vista cargada | Registro: Tipo "AUDITORIA_AD_ACCESO_DASHBOARD", Resultado "EXITOSO", Severidad "INFO", Descripción "Administrador {userName} accedió al dashboard de auditoría AD", Datos Adicionales: {"admin_id": "UUID"} | ☐ | ☐ | ☐ |
| 13 | Auditoría de exportación | Admin exporta logs | CSV generado | Registro: Tipo "AUDITORIA_AD_EXPORTACION", Resultado "EXITOSO", Severidad "INFO", Descripción "Administrador {userName} exportó {n} eventos de auditoría AD", Datos Adicionales: {"admin_id": "UUID", "eventos_exportados": 1234, "filtros": {"tenant": "UUID", "fecha_desde": "2026-01-15", "severidad": ["CRITICAL", "ERROR"]}} | ☐ | ☐ | ☐ |
| 14 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, admin, evento. Exportable CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

#### **HU-AD007A-DashboardAuditoria.html** - Vista principal

**Elementos:**
- Título: "Auditoría de Integración AD" (H3)
- **Filtros (panel izquierdo colapsable):**
  - Select "Tenant" (todos)
  - DateRangePicker "Período" (presets + personalizado)
  - Select "Categoría": Autenticación, Sincronización, Sesiones, Configuración, Todos
  - Multi-select "Severidad": INFO, WARNING, ERROR, CRITICAL
  - TextField "Buscar" (usuario o descripción)
  - Botón "Aplicar Filtros" (contained)
  - Link "Limpiar Filtros" (text)
- **Métricas Resumen (cards):**
  - "Total Eventos (filtrados)": **1,234**
  - "CRITICAL (24h)": **2** (rojo si >0)
  - "ERROR (24h)": **15**
  - "Logins SAML (24h)": **523**
- **Tabla de Eventos:**
  - Columnas: Timestamp, Tipo, Usuario, Tenant, Resultado, Severidad, Descripción, Acciones
  - Severidad con badge de color
  - Descripción truncada con "..."
  - Botón "Ver Detalle" (icono ojo)
  - Paginación: "Mostrando 1-100 de 1,234 eventos"
- Botón "Exportar CSV" (outlined, ícono download)
- Indicador "Última actualización: Hace 1 min" + botón "Actualizar"

#### **HU-AD007A-DetalleEvento.html** - Modal de detalle

**Elementos:**
- Título: "Detalle de Evento de Auditoría"
- **Información general:**
  - ID Evento: uuid (monospace, copiable)
  - Tipo: "INTEGRACION_AD_SAML_LOGIN_EXITOSO"
  - Timestamp Local: "20 Ene 2026, 14:30:45"
  - Timestamp UTC: "2026-01-20T19:30:45.123Z"
  - Usuario: "Juan Pérez (juan@empresa.com)" o "Sistema"
  - Tenant: "Empresa XYZ SAS"
  - IP Pública: "203.0.113.5"
  - Resultado: Badge "EXITOSO" (verde) o "FALLIDO" (rojo)
  - Severidad: Badge "INFO" (azul)
  - Descripción: Texto completo
- **Datos Adicionales** (sección expandible):
  - JSON formateado con syntax highlighting
  - Botón "Copiar JSON" (outlined)
- Botones: "Cerrar" (text)

**Estados:**
- Sin filtros (todos los eventos)
- Filtrado por tenant específico
- Filtrado por severidad CRITICAL + ERROR
- Sin resultados (mensaje "No se encontraron eventos")

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Consultas lentas con millones de logs** | Sin optimización, queries podrían tardar minutos | Alto | Alta | Índices en fecha, tipo_evento, tenant_id, severidad; particionamiento de tabla por mes; límite de 100 registros por página; cache de métricas | Mitigado |
| **Exportación masiva satura servidor** | Exportar millones de registros podría consumir memoria | Medio | Media | Límite de exportación (max 100k registros), streaming CSV (no cargar todo en memoria), job asíncrono para exportaciones grandes | Mitigado |
| **Datos sensibles expuestos en logs** | Passwords, tokens u otros datos sensibles en datos_adicionales | Alto | Baja | Sanitizar datos antes de logear (nunca logar passwords/tokens completos), revisión de código, alertas si detecta patrones sensibles | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance

- Alertas configurables basadas en eventos → HU futura
- Gráficas de tendencias de eventos → Solo métricas numéricas
- Comparación de eventos entre períodos → Solo vista de un período
- Retención automática configurable por UI → Configurable solo por devops

### Dependencias

**Depende de:**
- **Todas las HUs previas**: Generan eventos de auditoría
- **Sistema de Permisos**: Validar rol Administrador

**Relacionada con:**
- **HU-AD007B-D**: Otros dashboards específicos de auditoría

### Valor Independiente

- ✅ Centraliza auditoría de integración AD
- ✅ Esencial para cumplimiento regulatorio
- ✅ Facilita debugging de problemas
- ✅ Permite análisis de seguridad

### Especificación Técnica

**Índices Críticos:**
```sql
-- Tabla de auditoría ya existe (creada en HUs previas)
-- Agregar índices optimizados para dashboard

CREATE INDEX idx_audit_logs_ad_dashboard ON audit_logs(fecha DESC)
WHERE tipo_evento LIKE 'INTEGRACION_AD_%';

CREATE INDEX idx_audit_logs_ad_tenant ON audit_logs(tenant_id, fecha DESC)
WHERE tipo_evento LIKE 'INTEGRACION_AD_%';

CREATE INDEX idx_audit_logs_ad_severidad ON audit_logs(severidad, fecha DESC)
WHERE tipo_evento LIKE 'INTEGRACION_AD_%';

CREATE INDEX idx_audit_logs_ad_tipo ON audit_logs(tipo_evento, fecha DESC);
```

**Query Optimizado (con filtros):**
```sql
SELECT
  id,
  tipo_evento,
  fecha,
  usuario,
  t.nombre AS tenant_nombre,
  resultado,
  severidad,
  descripcion,
  LEFT(descripcion, 100) AS descripcion_corta
FROM audit_logs a
LEFT JOIN tenants t ON a.tenant_id = t.id
WHERE tipo_evento LIKE 'INTEGRACION_AD_%'
  AND ($1::UUID IS NULL OR tenant_id = $1)  -- Filtro tenant opcional
  AND fecha >= $2  -- Fecha desde
  AND fecha <= $3  -- Fecha hasta
  AND ($4::VARCHAR IS NULL OR tipo_evento = $4)  -- Tipo evento opcional
  AND severidad = ANY($5::VARCHAR[])  -- Severidades (array)
  AND (
    $6::VARCHAR IS NULL
    OR usuario ILIKE '%' || $6 || '%'
    OR descripcion ILIKE '%' || $6 || '%'
  )  -- Búsqueda texto opcional
ORDER BY fecha DESC
LIMIT 100 OFFSET $7;
```

**Particionamiento por Mes (para escalabilidad):**
```sql
-- Partición por mes para performance con millones de registros
CREATE TABLE audit_logs_2026_01 PARTITION OF audit_logs
FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');

CREATE TABLE audit_logs_2026_02 PARTITION OF audit_logs
FOR VALUES FROM ('2026-02-01') TO ('2026-03-01');

-- Job automático crea particiones futuras
```

**Exportación Streaming (evitar memoria):**
```javascript
async function exportarAuditoria(filtros) {
  const stream = db.stream(buildQuery(filtros));
  const csvStream = csv.stringify({header: true});

  res.setHeader('Content-Type', 'text/csv');
  res.setHeader('Content-Disposition', `attachment; filename="auditoria_ad_${Date.now()}.csv"`);

  stream.pipe(csvStream).pipe(res);

  // No carga todos los registros en memoria
}
```

**Retención de Logs:**
```sql
-- Job diario elimina logs antiguos (>90 días)
DELETE FROM audit_logs
WHERE fecha < NOW() - INTERVAL '90 days'
  AND tipo_evento LIKE 'INTEGRACION_AD_%'
  AND severidad IN ('INFO', 'WARNING');  -- No eliminar CRITICAL/ERROR

-- CRITICAL/ERROR se retienen 1 año mínimo
```

### Referencias

- **Estándar Auditoría**: Todos los tipos `INTEGRACION_AD_*`
- **Glosario**: Auditoría, Inmutabilidad, Severidad, Compliance
- **Guía de Estilos**: Material UI, Tables, Filters, Badges
- **Regulaciones**: GDPR, SOX, ISO 27001 (retención de logs)

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**Módulo:** Integración AD/SCIM - Audit & Logs
