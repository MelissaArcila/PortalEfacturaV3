# HU-AD006C - Métricas de Invalidación Proactiva

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
| **Arquitectura relacionada** | Portal Unificado - Monitoreo, Reporting, Seguridad |

---

## Contexto

Esta HU implementa **dashboard y métricas** para monitorear la efectividad del sistema de invalidación proactiva de sesiones (HU-AD006A, B).

Permite a Administradores del Portal:
- Visualizar cambios críticos procesados
- Medir latencia de invalidación (tiempo desde detección hasta aplicación)
- Identificar patrones de cambios por tenant o tipo
- Validar cumplimiento de SLA de seguridad (<1 minuto)
- Detectar problemas en el worker de invalidación

---

**Notas:**
- Dashboard disponible solo para Administradores del Portal
- Métricas en tiempo real y históricas (últimos 30 días)
- Alertas automáticas si SLA no se cumple
- Útil para auditorías de seguridad y cumplimiento

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Administrador del Portal** | Quiero ver métricas de invalidaciones proactivas | Para validar que el sistema de seguridad está funcionando correctamente |
| Como **Administrador de Seguridad** | Quiero medir latencia de invalidación | Para garantizar cumplimiento de SLA (<1 minuto) |
| Como **Administrador del Portal** | Quiero identificar tenants con cambios frecuentes | Para detectar posibles problemas de configuración o políticas |
| Como **Sistema de Monitoreo** | Quiero generar alertas si latencia excede umbral | Para intervención rápida en caso de degradación del servicio |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Acceso a dashboard de métricas | Administrador navega a "Métricas de Invalidación AD" | Usuario accede | Sistema valida permisos, muestra dashboard con: Filtro por Tenant (select), Filtro por Fecha (últimas 24h/7d/30d), Métricas clave (Cards), Gráficas, Tabla de cambios recientes. Si no es admin: mensaje "No tiene permisos" | ☐ | ☐ | ☐ |
| 2 | Métricas clave - Cards | Dashboard se carga | Sistema consulta BD | Sistema muestra cards con: **Cambios Críticos (24h)**: `SELECT COUNT(*) FROM cambios_criticos WHERE detectado_at >= NOW() - INTERVAL '24 hours'`, **Sesiones Invalidadas (24h)**: `SELECT SUM(sesiones_invalidadas) FROM cambios_criticos WHERE ...`, **Latencia Promedio**: `SELECT AVG(EXTRACT(EPOCH FROM (procesado_at - detectado_at))) FROM cambios_criticos WHERE procesado = true AND detectado_at >= ...`, **SLA Cumplido (%)**: porcentaje de cambios procesados en <60 seg. Actualización automática cada 5 minutos | ☐ | ☐ | ☐ |
| 3 | Gráfica de cambios críticos por tipo | Dashboard muestra distribución | Sistema grafica | Gráfica de barras con: Eje X = Tipo de Cambio (CAMBIO_ROLES, DESACTIVACION, ELIMINACION), Eje Y = Cantidad. Query: `SELECT tipo_cambio, COUNT(*) FROM cambios_criticos WHERE detectado_at >= {filtro_fecha} GROUP BY tipo_cambio`. Permite identificar tipo más común (ej: cambios de roles 70%, desactivaciones 20%, eliminaciones 10%) | ☐ | ☐ | ☐ |
| 4 | Gráfica de latencia de invalidación (línea de tiempo) | Dashboard muestra tendencia | Sistema grafica | Gráfica de línea: Eje X = Tiempo (agrupado por hora), Eje Y = Latencia promedio (segundos). Línea roja horizontal en 60 seg (umbral SLA). Permite ver si latencia está aumentando. Query: `SELECT DATE_TRUNC('hour', detectado_at) AS hora, AVG(EXTRACT(EPOCH FROM (procesado_at - detectado_at))) AS latencia FROM cambios_criticos GROUP BY hora ORDER BY hora` | ☐ | ☐ | ☐ |
| 5 | Top tenants con más cambios | Dashboard muestra ranking | Sistema consulta | Tabla "Top 10 Tenants con Más Cambios Críticos": Columnas: Tenant, Cambios (24h), Sesiones Invalidadas, Latencia Promedio. Query: `SELECT t.nombre, COUNT(*) AS cambios, SUM(sesiones_invalidadas), AVG(latencia) FROM cambios_criticos JOIN tenants GROUP BY t.id ORDER BY cambios DESC LIMIT 10`. Permite identificar clientes con actividad anómala | ☐ | ☐ | ☐ |
| 6 | Tabla de cambios recientes | Dashboard lista últimos cambios | Sistema consulta | Tabla con últimos 50 cambios: Columnas: Timestamp, Tenant, Usuario, Tipo Cambio, Sesiones Invalidadas, Latencia (seg), Estado. Ordenada por timestamp DESC. Query: `SELECT cc.*, u.userName, t.nombre, EXTRACT(EPOCH FROM (procesado_at - detectado_at)) AS latencia FROM cambios_criticos JOIN users JOIN tenants ORDER BY detectado_at DESC LIMIT 50`. Paginación si >50 | ☐ | ☐ | ☐ |
| 7 | Filtro por tenant específico | Administrador selecciona tenant | Usuario aplica filtro | Sistema filtra todas las gráficas y tablas por tenant_id seleccionado. Actualiza métricas solo para ese tenant. Muestra nombre del tenant en encabezado. Botón "Limpiar Filtro" disponible | ☐ | ☐ | ☐ |
| 8 | Filtro por rango de fechas | Administrador selecciona "Últimos 7 días" | Usuario cambia filtro | Sistema actualiza queries con `WHERE detectado_at >= NOW() - INTERVAL '7 days'`. Todas las métricas y gráficas se recalculan para ese período | ☐ | ☐ | ☐ |
| 9 | Detalle de cambio crítico | Administrador hace clic en fila de tabla | Sistema muestra modal | Sistema muestra modal "Detalle de Cambio Crítico" con: ID Cambio, Usuario afectado (nombre + email), Tenant, Tipo de Cambio, Severidad, Detectado en (timestamp), Procesado en (timestamp), Latencia (segundos, resaltado si >60 seg), Sesiones Invalidadas (número), Cambios Detalle (JSON formateado): roles anteriores/nuevos, active anterior/nuevo, etc. Botón "Cerrar" | ☐ | ☐ | ☐ |
| 10 | Exportar reporte de métricas | Administrador hace clic en "Exportar Reporte" | Sistema genera CSV | Sistema genera CSV con todos los cambios críticos (aplicando filtros actuales): Columnas: Tenant, Usuario, Email, Tipo Cambio, Detectado, Procesado, Latencia (seg), Sesiones Invalidadas, Estado. Descarga archivo `invalidaciones_proactivas_{fecha}.csv` | ☐ | ☐ | ☐ |
| 11 | Alerta de SLA no cumplido | Latencia promedio >120 seg por 15 min consecutivos | Sistema detecta umbral | Sistema genera alerta: Tipo "CRITICAL", Descripción "SLA de invalidación proactiva no cumplido: latencia promedio 145 seg (objetivo <60)", Notificación email/Slack a equipo técnico (configuración externa), Dashboard muestra banner rojo "⚠️ SLA NO CUMPLIDO - Latencia Alta", Registra en logs | ☐ | ☐ | ☐ |
| 12 | Indicador de salud del worker | Dashboard muestra estado | Sistema monitorea | Card "Estado del Worker": Badge "Operativo" (verde) si última ejecución <2 min, Badge "Inactivo" (rojo) si >5 min sin ejecutar. Query: `SELECT MAX(procesado_at) FROM cambios_criticos`. Si worker inactivo: alerta CRITICAL enviada | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 13 | Estructura obligatoria de auditoría | Eventos de métricas e alertas | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (INTEGRACION_AD_METRICAS_*), Fecha/Hora (UTC ISO 8601), Usuario (admin o NULL si alerta automática), Cliente (NULL), IP Local (NULL), IP Pública (IP admin si aplica), Resultado, Descripción, Severidad, Datos Adicionales | ☐ | ☐ | ☐ |
| 14 | Auditoría de acceso a dashboard | Admin accede a métricas | Vista cargada | Registro: Tipo "INTEGRACION_AD_METRICAS_ACCESO", Resultado "EXITOSO", Severidad "INFO", Descripción "Administrador {userName} accedió a dashboard de métricas de invalidación", Datos Adicionales: {"admin_id": "UUID"} | ☐ | ☐ | ☐ |
| 15 | Auditoría de exportación de reporte | Admin exporta CSV | Archivo generado | Registro: Tipo "INTEGRACION_AD_METRICAS_REPORTE_EXPORTADO", Resultado "EXITOSO", Severidad "INFO", Descripción "Administrador {userName} exportó reporte de invalidaciones", Datos Adicionales: {"admin_id": "UUID", "cambios_exportados": 234, "filtro_tenant": "tenant-uuid", "filtro_fecha": "7d"} | ☐ | ☐ | ☐ |
| 16 | Auditoría de alerta de SLA | Sistema detecta SLA no cumplido | Alerta generada | Registro: Tipo "INTEGRACION_AD_METRICAS_ALERTA_SLA", Resultado "EXITOSO", Severidad "CRITICAL", Descripción "SLA de invalidación proactiva no cumplido", Datos Adicionales: {"latencia_promedio_seg": 145, "umbral_sla_seg": 60, "periodo": "15min", "cambios_afectados": 23} | ☐ | ☐ | ☐ |
| 17 | Auditoría de worker inactivo | Sistema detecta worker sin ejecutar | Alerta generada | Registro: Tipo "INTEGRACION_AD_METRICAS_WORKER_INACTIVO", Resultado "EXITOSO", Severidad "CRITICAL", Descripción "Worker de invalidación inactivo por >5 minutos", Datos Adicionales: {"ultima_ejecucion": "2026-01-20T14:00:00Z", "minutos_inactivo": 7} | ☐ | ☐ | ☐ |
| 18 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, admin, evento, severidad. Exportable CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

#### **HU-AD006C-MetricasInvalidacion.html** - Dashboard principal

**Elementos:**
- Título: "Métricas de Invalidación Proactiva" (H3)
- **Filtros:**
  - Select "Tenant" (todos, placeholder "Todos los Tenants")
  - Radio buttons "Período": "24 horas", "7 días" (default), "30 días"
  - Botón "Actualizar" (outlined, ícono refresh)
- **Métricas Clave (Cards grandes):**
  - "Cambios Críticos (7d)": **342**
  - "Sesiones Invalidadas (7d)": **1,247**
  - "Latencia Promedio": **42 seg** (verde si <60, rojo si >60)
  - "SLA Cumplido": **98.5%** (meta: >95%)
- **Estado del Worker:**
  - Badge "Operativo ✓" (verde) o "Inactivo ⚠️" (rojo)
  - "Última ejecución: Hace 45 segundos"
- **Gráficas:**
  - "Distribución por Tipo de Cambio" (barras horizontales)
  - "Latencia de Invalidación" (línea temporal con umbral SLA)
- **Tablas:**
  - "Top 10 Tenants con Más Cambios" (ranking)
  - "Cambios Recientes" (últimos 50, paginación)
- Botón "Exportar Reporte" (outlined, ícono download)
- Auto-refresh: Indicador "Actualizado hace 2 min"

#### **HU-AD006C-DetalleCambio.html** - Modal de detalle

**Elementos:**
- Título: "Detalle de Cambio Crítico"
- **Información:**
  - ID: uuid (monospace, copiable)
  - Usuario: "Juan Pérez" (juan@empresa.com)
  - Tenant: "Empresa XYZ SAS"
  - Tipo: Badge "CAMBIO_ROLES" (color según tipo)
  - Severidad: Badge "HIGH" (amarillo) o "CRITICAL" (rojo)
  - Detectado: "20 Ene 2026, 14:30:15"
  - Procesado: "20 Ene 2026, 14:30:58"
  - **Latencia: 43 segundos** (verde si <60, rojo si >60)
  - Sesiones Invalidadas: **3**
- **Cambios Detalle** (JSON formateado):
```json
{
  "roles_anteriores": ["Contador"],
  "roles_nuevos": ["Contador", "Administrador del Portal"],
  "accion": "ADICION",
  "rol_agregado": "Administrador del Portal"
}
```
- Botón "Cerrar" (text)

**Estados:**
- Latencia normal (<60 seg, verde)
- Latencia alta (>60 seg, rojo, resaltado)
- Worker operativo (verde)
- Worker inactivo (banner rojo alerta)
- SLA cumplido (verde)
- SLA no cumplido (rojo)

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Consultas de métricas lentas** | Dashboard con miles de cambios podría tardar en cargar | Medio | Media | Índices optimizados, agregación por hora/día, cache de métricas (5 min TTL), paginación en tablas | Mitigado |
| **Alertas de SLA generan fatiga** | Muchas alertas falsas o frecuentes reducen efectividad | Medio | Baja | Umbral conservador (120 seg + 15 min consecutivos), agregación de alertas (1 alerta/hora máx), silencio manual por mantenimiento | Mitigado |
| **Métricas incorrectas por queries erróneos** | Bugs en queries SQL podrían mostrar datos incorrectos | Medio | Baja | Testing exhaustivo de queries, validación cruzada con auditoría, revisión de código | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance

- Predicción de cambios futuros → Solo históricos y en tiempo real
- Gráficas interactivas complejas (drill-down) → Gráficas estáticas básicas
- Alertas configurables por usuario → Alertas fijas para todo el equipo
- Dashboard personalizable → Layout fijo para todos los admins

### Dependencias

**Depende de:**
- **HU-AD006A, B - Invalidación Proactiva**: Genera datos a medir
- **Sistema de Permisos**: Validar rol Administrador

**Relacionada con:**
- **HU-AD007A-D - Audit & Logs**: Dashboard complementario de auditoría

### Valor Independiente

- ✅ Valida efectividad del sistema de seguridad
- ✅ Provee visibilidad para cumplimiento
- ✅ Detecta degradación del servicio temprano
- ✅ Útil para auditorías y reporting

### Especificación Técnica

**Queries Optimizados:**

Métricas Clave:
```sql
-- Índice crítico
CREATE INDEX idx_cambios_criticos_metricas ON cambios_criticos(detectado_at, procesado_at, tenant_id);

-- Cambios últimos 7 días
SELECT COUNT(*) AS cambios,
       SUM(sesiones_invalidadas) AS sesiones,
       AVG(EXTRACT(EPOCH FROM (procesado_at - detectado_at))) AS latencia_promedio
FROM cambios_criticos
WHERE detectado_at >= NOW() - INTERVAL '7 days'
  AND procesado = true;

-- SLA cumplido (%)
SELECT
  COUNT(*) FILTER (WHERE EXTRACT(EPOCH FROM (procesado_at - detectado_at)) <= 60) * 100.0 / COUNT(*) AS sla_porcentaje
FROM cambios_criticos
WHERE detectado_at >= NOW() - INTERVAL '7 days'
  AND procesado = true;
```

Distribución por Tipo:
```sql
SELECT tipo_cambio,
       COUNT(*) AS cantidad,
       AVG(EXTRACT(EPOCH FROM (procesado_at - detectado_at))) AS latencia_promedio
FROM cambios_criticos
WHERE detectado_at >= NOW() - INTERVAL '7 days'
GROUP BY tipo_cambio
ORDER BY cantidad DESC;
```

Top Tenants:
```sql
SELECT t.nombre AS tenant,
       COUNT(cc.id) AS cambios,
       SUM(cc.sesiones_invalidadas) AS sesiones_invalidadas,
       AVG(EXTRACT(EPOCH FROM (cc.procesado_at - cc.detectado_at))) AS latencia_promedio
FROM cambios_criticos cc
JOIN tenants t ON cc.tenant_id = t.id
WHERE cc.detectado_at >= NOW() - INTERVAL '7 days'
GROUP BY t.id, t.nombre
ORDER BY cambios DESC
LIMIT 10;
```

**Cache de Métricas (Redis):**
```javascript
const CACHE_TTL = 300; // 5 minutos

async function getMetricasKey(tenant_id, periodo) {
  const cacheKey = `metricas:invalidacion:${tenant_id || 'all'}:${periodo}`;

  // Intentar cache
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);

  // Calcular métricas
  const metricas = await calcularMetricas(tenant_id, periodo);

  // Almacenar en cache
  await redis.setex(cacheKey, CACHE_TTL, JSON.stringify(metricas));

  return metricas;
}
```

**Alerta Automática:**
```javascript
async function verificarSLA() {
  const latencia_promedio = await db.query(`
    SELECT AVG(EXTRACT(EPOCH FROM (procesado_at - detectado_at))) AS latencia
    FROM cambios_criticos
    WHERE procesado_at >= NOW() - INTERVAL '15 minutes'
      AND procesado = true
  `);

  if (latencia_promedio > 120) {
    await generarAlerta({
      tipo: 'CRITICAL',
      descripcion: `SLA no cumplido: latencia ${latencia_promedio} seg`,
      notificar: ['equipo-devops@portal.com']
    });
  }
}

// Ejecutar cada 5 minutos
setInterval(verificarSLA, 5 * 60 * 1000);
```

### Referencias

- **HU-AD006A, B - Invalidación Proactiva**: Fuente de datos
- **Estándar Auditoría**: Tipos `INTEGRACION_AD_METRICAS_*`
- **Glosario**: SLA, Latencia, Worker Heartbeat
- **Guía de Estilos**: Material UI, Dashboard, Charts (Chart.js o similar)

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**Módulo:** Integración AD/SCIM - Proactive Invalidation
