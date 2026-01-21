# HU-AD005B - Monitor de Actividad de Sesiones

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
| **Arquitectura relacionada** | Portal Unificado - Administración, Session Management |

---

## Contexto

Esta HU implementa un **dashboard de monitoreo** para que Administradores del Portal puedan visualizar y gestionar sesiones activas de usuarios con integración AD a nivel de tenant.

Permite:
- Ver sesiones activas en tiempo real por tenant
- Identificar usuarios con múltiples sesiones
- Cerrar sesiones específicas o masivas por motivos de seguridad
- Monitorear actividad de logins y sesiones sospechosas
- Obtener métricas de uso de SSO

---

**Notas:**
- Solo Administradores del Portal pueden acceder
- Vista filtrable por tenant (multitenant)
- Útil para incidentes de seguridad (cuenta comprometida)
- Provee visibilidad de adopción de integración AD

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Administrador del Portal** | Quiero ver todas las sesiones activas de usuarios AD | Para monitorear actividad y detectar sesiones anómalas |
| Como **Administrador del Portal** | Quiero cerrar sesiones específicas o masivas | Para responder a incidentes de seguridad (ej: cuenta comprometida) |
| Como **Administrador del Portal** | Quiero ver métricas de logins SSO | Para medir adopción de integración AD y detectar problemas |
| Como **Administrador del Portal** | Quiero filtrar sesiones por tenant | Para enfocarse en un cliente específico |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Acceso a dashboard de sesiones | Administrador del Portal en menú principal | Usuario navega a "Monitoreo de Sesiones AD" | Sistema valida permisos de administrador, muestra dashboard con: Filtro por Tenant (select), Métricas generales (Sesiones Activas Totales, Logins Hoy, Logins Última Hora), Tabla de sesiones activas (ordenada por última actividad desc). Si no es administrador: mensaje "No tiene permisos" | ☐ | ☐ | ☐ |
| 2 | Métricas generales de sesiones | Dashboard se carga | Sistema consulta BD | Sistema muestra tarjetas con métricas: **Sesiones Activas**: `SELECT COUNT(*) FROM sessions WHERE invalidated_at IS NULL AND expires_at > NOW() AND origen_saml = true`, **Logins Hoy**: `SELECT COUNT(*) FROM audit_logs WHERE tipo_evento = 'INTEGRACION_AD_SAML_LOGIN_EXITOSO' AND fecha >= TODAY()`, **Logins Última Hora**: similar con fecha >= NOW() - 1 hour. Actualización automática cada 30 segundos | ☐ | ☐ | ☐ |
| 3 | Tabla de sesiones activas | Administrador visualiza dashboard | Sistema lista sesiones | Sistema consulta: `SELECT s.session_id, s.user_id, u.userName, t.nombre AS tenant_nombre, s.created_at, s.last_activity, s.ip_usuario, s.user_agent FROM sessions s JOIN users u ON s.user_id = u.id JOIN tenants t ON s.tenant_id = t.id WHERE s.invalidated_at IS NULL AND s.expires_at > NOW() AND s.origen_saml = true ORDER BY s.last_activity DESC LIMIT 100`. Muestra tabla con columnas: Usuario (userName), Tenant, Inicio Sesión, Última Actividad, IP, Dispositivo (user_agent parseado), Acciones (botón "Cerrar") | ☐ | ☐ | ☐ |
| 4 | Filtro por tenant específico | Administrador selecciona tenant en dropdown | Usuario aplica filtro | Sistema filtra tabla: `WHERE s.tenant_id = {selected_tenant}`. Actualiza métricas solo para ese tenant. Muestra nombre del tenant seleccionado en encabezado. Botón "Limpiar Filtro" visible | ☐ | ☐ | ☐ |
| 5 | Búsqueda por userName | Administrador ingresa email en campo de búsqueda | Usuario busca | Sistema filtra tabla por userName (LIKE '%{búsqueda}%'). Muestra resultados en tiempo real (debounce 500ms). Si no hay resultados: mensaje "No se encontraron sesiones para '{búsqueda}'" | ☐ | ☐ | ☐ |
| 6 | Detalle de sesión específica | Administrador hace clic en fila de sesión | Sistema muestra modal | Sistema muestra modal "Detalle de Sesión" con: Session ID, Usuario (nombre completo + email), Tenant, Creada: fecha/hora, Expira: fecha/hora (resaltado si <1h), Última Actividad: "Hace 5 minutos", IP: con geolocalización estimada, User Agent: parseado completo (SO, navegador, versión), Origen: "SAML 2.0", Botones: "Cerrar Sesión" (contained), "Cancelar" (text) | ☐ | ☐ | ☐ |
| 7 | Cerrar sesión individual | Administrador hace clic en "Cerrar Sesión" | Sistema invalida sesión | Sistema muestra confirmación: "¿Cerrar sesión de {userName}? El usuario deberá autenticarse nuevamente.". Si confirma: ejecuta `UPDATE sessions SET invalidated_at = NOW(), logout_type = 'ADMIN_MANUAL' WHERE session_id = {id}`, registra en auditoría quién cerró la sesión, muestra mensaje "Sesión cerrada exitosamente", actualiza tabla (sesión desaparece). Usuario afectado recibe 401 en próxima petición | ☐ | ☐ | ☐ |
| 8 | Cerrar todas las sesiones de un usuario | Administrador en detalle de sesión hace clic en "Cerrar Todas las Sesiones del Usuario" | Sistema invalida múltiples sesiones | Sistema muestra confirmación: "¿Cerrar TODAS las sesiones de {userName} ({n} sesiones)? Útil si cuenta comprometida.". Si confirma: `UPDATE sessions SET invalidated_at = NOW(), logout_type = 'ADMIN_SEGURIDAD' WHERE user_id = {id} AND invalidated_at IS NULL`, registra en auditoría (CRÍTICO), muestra mensaje "{n} sesiones cerradas", actualiza tabla | ☐ | ☐ | ☐ |
| 9 | Identificar usuarios con sesiones múltiples | Dashboard muestra estadística | Sistema consulta | Sección "Usuarios con Más Sesiones" muestra top 10: `SELECT u.userName, t.nombre, COUNT(*) AS sesiones FROM sessions s JOIN users u JOIN tenants t WHERE s.invalidated_at IS NULL GROUP BY u.id ORDER BY sesiones DESC LIMIT 10`. Permite identificar usuarios con muchas sesiones (posible compromiso o uso excesivo de dispositivos) | ☐ | ☐ | ☐ |
| 10 | Exportar reporte de sesiones | Administrador hace clic en "Exportar Reporte" | Sistema genera CSV | Sistema genera CSV con todas las sesiones activas (aplicando filtros actuales): Columnas: Tenant, Usuario, Email, Creada, Última Actividad, IP, Dispositivo, Session ID. Descarga archivo `sesiones_activas_{fecha}.csv`. Útil para análisis o cumplimiento | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 11 | Estructura obligatoria de auditoría | Eventos de administración de sesiones | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (INTEGRACION_AD_ADMIN_SESION_*), Fecha/Hora (UTC ISO 8601), Usuario (Administrador que ejecuta acción), Cliente (tenant afectado o NULL), IP Local (NULL), IP Pública (IP del admin), Resultado, Descripción, Severidad, Datos Adicionales | ☐ | ☐ | ☐ |
| 12 | Auditoría de sesión cerrada por admin | Admin cierra sesión individual | Sesión invalidada | Registro: Tipo "INTEGRACION_AD_ADMIN_SESION_CERRADA", Resultado "EXITOSO", Severidad "WARNING", Descripción "Administrador {admin_userName} cerró sesión de {usuario_afectado}", Datos Adicionales: {"admin_id": "UUID", "session_id": "UUID", "user_afectado_id": "UUID", "tenant_id": "UUID", "razon": "Manual por administrador"} | ☐ | ☐ | ☐ |
| 13 | Auditoría de cierre masivo de sesiones | Admin cierra todas las sesiones de un usuario | Múltiples sesiones invalidadas | Registro: Tipo "INTEGRACION_AD_ADMIN_SESIONES_CERRADAS_MASIVO", Resultado "EXITOSO", Severidad "CRITICAL", Descripción "Administrador {admin} cerró {n} sesiones de usuario {afectado} por seguridad", Datos Adicionales: {"admin_id": "UUID", "user_afectado_id": "UUID", "sesiones_cerradas": 5, "razon": "Posible compromiso"} | ☐ | ☐ | ☐ |
| 14 | Auditoría de acceso al dashboard | Admin accede al monitor | Vista cargada | Registro: Tipo "INTEGRACION_AD_ADMIN_MONITOR_ACCESO", Resultado "EXITOSO", Severidad "INFO", Descripción "Administrador {userName} accedió al monitor de sesiones AD", Datos Adicionales: {"admin_id": "UUID", "sesiones_activas_vistas": 45} | ☐ | ☐ | ☐ |
| 15 | Auditoría de exportación de reporte | Admin exporta CSV | Archivo generado | Registro: Tipo "INTEGRACION_AD_ADMIN_REPORTE_EXPORTADO", Resultado "EXITOSO", Severidad "INFO", Descripción "Administrador {userName} exportó reporte de sesiones AD", Datos Adicionales: {"admin_id": "UUID", "sesiones_exportadas": 120, "filtro_tenant": "tenant-uuid"} | ☐ | ☐ | ☐ |
| 16 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, admin, tenant, evento. Exportable CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: Pendiente**

#### **HU-AD005B-MonitorSesiones.html** - Dashboard principal

**Elementos:**
- Título: "Monitor de Sesiones AD" (H3)
- **Filtros:**
  - Select "Tenant" (todos los tenants con AD activo, placeholder "Todos")
  - TextField "Buscar por usuario" (con ícono lupa)
  - Botón "Limpiar Filtros" (text)
- **Métricas (Cards):**
  - "Sesiones Activas": **247** (número grande, actualización automática)
  - "Logins Hoy": **1,523**
  - "Logins Última Hora": **45**
- **Sección "Usuarios con Más Sesiones":**
  - Lista top 10: "juan.perez@empresa.com (5 sesiones)", "maria.gomez@empresa.com (4 sesiones)"...
  - Click en usuario filtra tabla a ese usuario
- **Tabla de Sesiones Activas:**
  - Columnas: Usuario, Tenant, Inicio, Última Actividad, IP, Dispositivo, Acciones
  - Indicador visual si última actividad >1h (amarillo), >2h (rojo)
  - Paginación: 50 por página
  - Botón "Exportar Reporte" (outlined, ícono download)
- Auto-refresh: Indicador "Actualizado hace 15 seg", botón "Actualizar Ahora" (ícono refresh)

#### **HU-AD005B-DetalleSesion.html** - Modal de detalle

**Elementos:**
- Título: "Detalle de Sesión"
- **Información:**
  - Usuario: "Juan Pérez" (juan.perez@empresa.com)
  - Tenant: "Empresa XYZ SAS"
  - Session ID: uuid (monospace, copiable)
  - Creada: "20 Ene 2026, 10:00 AM"
  - Expira: "20 Ene 2026, 2:00 PM" (badge "Expira en 45 min" si <1h)
  - Última Actividad: "Hace 5 minutos"
  - IP: "203.0.113.5" (Medellín, Colombia)
  - Dispositivo: "Chrome 120 en Windows 10"
  - Origen: Badge "SAML 2.0"
- **Acciones:**
  - Botón "Cerrar Esta Sesión" (contained, warning color)
  - Botón "Cerrar Todas las Sesiones del Usuario" (outlined, critical color, confirmación extra)
  - Botón "Cancelar" (text)

**Estados:**
- Sesión normal (última actividad <1h)
- Sesión inactiva (última actividad >2h, resaltado amarillo)
- Sesión próxima a expirar (<1h, badge naranja)

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Admin cierra sesiones incorrectas por error** | Confundir usuario y cerrar sesiones de persona incorrecta | Medio | Baja | Confirmación con nombre de usuario visible, auditoría completa de acciones, reversible (usuario re-autentica), capacitación a admins | Mitigado |
| **Dashboard no escala con miles de sesiones** | Consultas lentas si >10k sesiones activas | Medio | Media | Paginación (50 por página), índices optimizados en BD, filtro por tenant reduce resultados, considerar cache para métricas | Mitigado |
| **Exposición de información sensible** | IPs y user-agents podrían considerarse datos sensibles | Bajo | Baja | Solo administradores con permisos, auditoría de accesos al dashboard, cumplir GDPR/políticas de privacidad | Aceptado |

---

## Notas Adicionales

### Fuera de Alcance

- Cerrar sesiones de usuarios NO-AD (locales) → Solo usuarios con origen_saml=true
- Notificaciones push a usuarios cuando admin cierra su sesión → Usuario descubre al recibir 401
- Gráficas históricas de sesiones → Solo vista actual
- Exportación de auditoría desde este dashboard → Dashboard de auditoría separado (HU-AD007)

### Dependencias

**Depende de:**
- **HU-AD005A - Gestión Sesiones JWT**: Usa tabla sessions
- **Sistema de Permisos**: Validar rol Administrador del Portal

**Relacionada con:**
- **HU-AD007A-D - Audit & Logs**: Registra acciones de admin

### Valor Independiente

- ✅ Permite gestión proactiva de sesiones por admins
- ✅ Útil para respuesta a incidentes de seguridad
- ✅ Provee visibilidad de uso de integración AD
- ✅ Puede entregarse después de HU-AD005A

### Especificación Técnica

**Permisos Requeridos:**
```javascript
// Middleware de autorización
function requireAdmin(req, res, next) {
  const user = req.user; // De JWT validado

  if (!user.roles.includes('Administrador del Portal')) {
    return res.status(403).json({
      error: 'No tiene permisos para acceder a esta sección'
    });
  }

  next();
}

// Ruta protegida
app.get('/api/admin/sessions', requireAdmin, getSessions);
```

**Parseo de User-Agent:**
```javascript
const UAParser = require('ua-parser-js');

function parseUserAgent(uaString) {
  const parser = new UAParser(uaString);
  const result = parser.getResult();

  return {
    browser: `${result.browser.name} ${result.browser.version}`,
    os: `${result.os.name} ${result.os.version}`,
    device: result.device.type || 'Desktop'
  };
}

// Ejemplo: "Chrome 120 en Windows 10"
```

**Geolocalización por IP (básica):**
```javascript
const geoip = require('geoip-lite');

function getLocation(ip) {
  const geo = geoip.lookup(ip);

  if (!geo) return 'Ubicación desconocida';

  return `${geo.city || 'Ciudad desconocida'}, ${geo.country}`;
}

// Ejemplo: "Medellín, CO"
```

**Consulta Optimizada (índices):**
```sql
-- Índice compuesto para query principal
CREATE INDEX idx_sessions_monitor ON sessions(
  invalidated_at,
  expires_at,
  origen_saml,
  last_activity DESC
) WHERE invalidated_at IS NULL AND origen_saml = true;

-- Query rápida
SELECT s.session_id, u.userName, t.nombre, s.created_at, s.last_activity, s.ip_usuario
FROM sessions s
INNER JOIN users u ON s.user_id = u.id
INNER JOIN tenants t ON s.tenant_id = t.id
WHERE s.invalidated_at IS NULL
  AND s.expires_at > NOW()
  AND s.origen_saml = true
ORDER BY s.last_activity DESC
LIMIT 50 OFFSET 0;
```

### Referencias

- **HU-AD005A - Gestión Sesiones JWT**: Modelo de datos sessions
- **Estándar Auditoría**: Tipos `INTEGRACION_AD_ADMIN_SESION_*`
- **Glosario**: Session Monitor, User-Agent, Geolocalización
- **Guía de Estilos**: Material UI, Dashboard, Tables, Cards

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**Módulo:** Integración AD/SCIM - Session Management
