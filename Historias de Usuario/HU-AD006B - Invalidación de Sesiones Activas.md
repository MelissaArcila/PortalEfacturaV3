# HU-AD006B - Invalidación de Sesiones Activas

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
| **Arquitectura relacionada** | Portal Unificado - Seguridad, Session Management |

---

## Contexto

Esta HU implementa el **proceso de invalidación proactiva** de sesiones activas cuando se detectan cambios críticos (HU-AD006A) en usuarios gestionados por AD.

El proceso:
1. **Worker** lee cambios críticos pendientes de la tabla `cambios_criticos`
2. **Invalida** todas las sesiones activas del usuario afectado
3. **Registra** la invalidación en auditoría
4. **Marca** el cambio crítico como procesado

Objetivo: Usuario pierde acceso en próxima petición (< 1 minuto desde detección del cambio).

---

**Notas:**
- Worker ejecuta cada 1 minuto para procesamiento rápido
- Invalida TODAS las sesiones del usuario (todos los dispositivos)
- Usuario debe re-autenticarse vía SAML para obtener nueva sesión con permisos actualizados
- Proceso es automático y transparente (no requiere intervención humana)

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Portal Unificado** | Quiero invalidar sesiones activas cuando detecte cambios críticos | Para garantizar que cambios de permisos se apliquen inmediatamente, no en 4 horas |
| Como **Worker de Invalidación** | Quiero procesar cambios críticos pendientes cada minuto | Para minimizar ventana de tiempo entre cambio y aplicación de seguridad |
| Como **Portal Unificado** | Quiero registrar invalidaciones proactivas en auditoría | Para cumplimiento y debugging de políticas de seguridad |
| Como **Usuario afectado** | Quiero re-autenticarme automáticamente cuando mi sesión sea invalidada | Para continuar trabajando sin fricción excesiva |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Worker de invalidación ejecuta periódicamente | Job programado (cada 1 minuto) | Timer se dispara | Worker ejecuta query: `SELECT * FROM cambios_criticos WHERE procesado = false ORDER BY detectado_at ASC LIMIT 100`. Para cada cambio crítico pendiente: invoca proceso de invalidación (escenarios 2-5). Registra en log técnico cantidad de cambios procesados | ☐ | ☐ | ☐ |
| 2 | Invalidación por cambio de roles | Cambio crítico tipo CAMBIO_ROLES | Worker procesa cambio | Worker: (1) Obtiene user_id del cambio, (2) Consulta sesiones activas: `SELECT session_id FROM sessions WHERE user_id = {id} AND invalidated_at IS NULL AND expires_at > NOW()`, (3) Para cada sesión: ejecuta `UPDATE sessions SET invalidated_at = NOW(), logout_type = 'PROACTIVO_CAMBIO_ROLES' WHERE session_id = {id}`, (4) Cuenta sesiones invalidadas, (5) Actualiza cambio crítico: `UPDATE cambios_criticos SET procesado = true, procesado_at = NOW(), sesiones_invalidadas = {count} WHERE id = {cambio_id}`, (6) Registra en auditoría | ☐ | ☐ | ☐ |
| 3 | Invalidación por desactivación de cuenta | Cambio crítico tipo DESACTIVACION | Worker procesa | Similar a escenario 2, pero `logout_type = 'PROACTIVO_DESACTIVACION'`. Usuario con `active=false` no puede re-autenticarse hasta reactivación en AD + sync | ☐ | ☐ | ☐ |
| 4 | Invalidación por eliminación de usuario | Cambio crítico tipo ELIMINACION | Worker procesa | Similar a escenarios anteriores, `logout_type = 'PROACTIVO_ELIMINACION'`. Usuario con `deleted_at` no puede re-autenticarse nunca (hasta restauración manual) | ☐ | ☐ | ☐ |
| 5 | Invalidación de múltiples sesiones | Usuario tiene 5 sesiones activas en diferentes dispositivos | Worker invalida todas | Worker invalida las 5 sesiones en una transacción. Todas las sesiones marcadas con mismo `invalidated_at` timestamp. Próxima petición desde cualquier dispositivo recibe HTTP 401 | ☐ | ☐ | ☐ |
| 6 | Usuario sin sesiones activas | Cambio crítico para usuario sin sesiones | Worker procesa | Worker ejecuta query, obtiene 0 sesiones. Marca cambio como procesado con `sesiones_invalidadas = 0`. Registra en auditoría "Cambio procesado, usuario sin sesiones activas". NO genera error | ☐ | ☐ | ☐ |
| 7 | Experiencia del usuario - Sesión invalidada | Usuario afectado hace petición después de invalidación | Sistema valida JWT | Sistema detecta que sesión está invalidada (`sessions.invalidated_at IS NOT NULL`), devuelve HTTP 401 con body: `{"error": "Session invalidated", "reason": "Security policy: permissions changed", "action": "reauthenticate"}`. Frontend detecta 401, redirige automáticamente a flujo SSO (HU-AD004B). Si usuario tiene sesión activa en IdP: silent re-authentication, obtiene nueva sesión con roles actualizados en segundos. Usuario experimenta: logout + login automático rápido | ☐ | ☐ | ☐ |
| 8 | Transacción atómica de invalidación | Worker invalida sesiones | BD transaction | Proceso de invalidación ocurre en transacción: BEGIN; UPDATE sessions...; UPDATE cambios_criticos...; INSERT audit_log...; COMMIT;. Si cualquier paso falla: ROLLBACK completo, cambio crítico queda como no-procesado, worker reintentará en próxima ejecución. Garantiza consistencia | ☐ | ☐ | ☐ |
| 9 | Manejo de error durante invalidación | BD no disponible, timeout | Worker encuentra error | Worker captura error, registra en log técnico: `{cambio_id, error, timestamp}`, NO marca cambio como procesado, actualiza `cambios_criticos.error_procesamiento = {error_message}`. Próxima ejecución del worker reintentará. Alertas si mismo cambio falla >3 veces | ☐ | ☐ | ☐ |
| 10 | Métricas de performance de invalidación | Worker ejecuta | Sistema mide tiempos | Sistema registra métricas: tiempo_deteccion_a_invalidacion (segundos), sesiones_invalidadas_por_minuto, cambios_procesados_por_ejecucion. Objetivo: <60 segundos desde detección hasta invalidación. Alertas si > 120 segundos | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 11 | Estructura obligatoria de auditoría | Eventos de invalidación proactiva | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (INTEGRACION_AD_INVALIDACION_PROACTIVA_*), Fecha/Hora (UTC ISO 8601), Usuario (user_id afectado), Cliente (tenant_id), IP Local (NULL), IP Pública (NULL), Resultado, Descripción, Severidad, Datos Adicionales | ☐ | ☐ | ☐ |
| 12 | Auditoría de invalidación por cambio de roles | Worker invalida sesiones tras cambio roles | Invalidación completada | Registro: Tipo "INTEGRACION_AD_INVALIDACION_PROACTIVA_ROLES", Resultado "EXITOSO", Severidad "WARNING", Descripción "Sesiones invalidadas para usuario {userName} por cambio de roles", Datos Adicionales: {"user_id": "UUID", "tenant_id": "UUID", "sesiones_invalidadas": 3, "cambio_id": "UUID", "roles_anteriores": ["Contador"], "roles_nuevos": ["Administrador"], "tiempo_deteccion_invalidacion_seg": 45} | ☐ | ☐ | ☐ |
| 13 | Auditoría de invalidación por desactivación | Worker invalida sesiones tras desactivación | Invalidación completada | Registro: Tipo "INTEGRACION_AD_INVALIDACION_PROACTIVA_DESACTIVACION", Resultado "EXITOSO", Severidad "CRITICAL", Descripción "Sesiones invalidadas para usuario {userName} por desactivación de cuenta", Datos Adicionales: {"user_id": "UUID", "sesiones_invalidadas": 2, "cambio_id": "UUID"} | ☐ | ☐ | ☐ |
| 14 | Auditoría de invalidación por eliminación | Worker invalida sesiones tras DELETE | Invalidación completada | Registro: Tipo "INTEGRACION_AD_INVALIDACION_PROACTIVA_ELIMINACION", Resultado "EXITOSO", Severidad "CRITICAL", Descripción "Sesiones invalidadas para usuario {userName} por eliminación", Datos Adicionales: {"user_id": "UUID", "sesiones_invalidadas": 1, "cambio_id": "UUID"} | ☐ | ☐ | ☐ |
| 15 | Auditoría de usuario sin sesiones activas | Worker procesa cambio sin sesiones | Procesado sin invalidar | Registro: Tipo "INTEGRACION_AD_INVALIDACION_PROACTIVA_SIN_SESIONES", Resultado "EXITOSO", Severidad "INFO", Descripción "Cambio crítico procesado para {userName}, sin sesiones activas", Datos Adicionales: {"user_id": "UUID", "cambio_id": "UUID", "tipo_cambio": "CAMBIO_ROLES"} | ☐ | ☐ | ☐ |
| 16 | Auditoría de error de invalidación | Worker falla al procesar | Error capturado | Registro: Tipo "INTEGRACION_AD_INVALIDACION_PROACTIVA_ERROR", Resultado "FALLIDO", Severidad "ERROR", Descripción "Error al invalidar sesiones para {userName}", Datos Adicionales: {"user_id": "UUID", "cambio_id": "UUID", "error": "Database timeout", "intentos": 1} | ☐ | ☐ | ☐ |
| 17 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, usuario, tenant, tipo. Exportable CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: N/A - Proceso backend automático**

**Experiencia del Usuario Afectado:**

```
Usuario hace petición → JWT validado → Sesión invalidada detectada
     ↓
HTTP 401 {"error": "Session invalidated", "reason": "Security policy: permissions changed"}
     ↓
Frontend detecta 401 → Muestra mensaje breve: "Su sesión ha expirado. Redirigiendo..."
     ↓
Redirect automático a flujo SSO (HU-AD004B)
     ↓
IdP tiene sesión activa → Silent re-authentication (sin pedir credenciales)
     ↓
Nueva sesión JWT generada con roles actualizados
     ↓
Usuario autenticado, continúa trabajando (proceso dura ~2-5 segundos)
```

**Mensaje Transitorio (opcional):**
- "Actualizando sesión..." (spinner, 2-3 seg)
- Transparente para el usuario

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Worker falla y no procesa cambios** | Si worker se detiene, cambios críticos no se invalidan | Crítico | Baja | Monitoreo del worker (heartbeat), alertas si no ejecuta en 5 min, reinicio automático, métricas de cambios pendientes | Mitigado |
| **Invalidación muy frecuente molesta a usuarios** | Si cambios en AD son muy frecuentes, usuarios re-autentican constantemente | Medio | Baja | Solo invalidar para cambios críticos (no nombres, emails), silent re-auth minimiza fricción, documentar a clientes evitar cambios frecuentes | Aceptado |
| **Transacción de invalidación falla parcialmente** | Algunas sesiones invalidan pero no todas | Alto | Baja | Transacción atómica (escenario 8), reintento automático si falla, alertas si fallas consecutivas | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance

- Invalidación selectiva (solo sesiones específicas) → Invalida todas las sesiones del usuario
- Notificación push al usuario antes de invalidar → Usuario descubre al hacer petición
- Whitelist de usuarios exentos de invalidación → Aplica a todos por igual
- Configuración de intervalo del worker por tenant → Global 1 minuto para todos

### Dependencias

**Depende de:**
- **HU-AD006A - Detección Cambios Críticos**: Genera cambios pendientes
- **HU-AD005A - Gestión Sesiones JWT**: Usa tabla sessions

**Es prerequisito para:**
- **HU-AD006C - Métricas de Invalidación**: Reporta estadísticas

### Valor Independiente

- ✅ Aplica políticas de seguridad proactivamente
- ✅ Objetivo <1 minuto cumple SLA de seguridad
- ✅ Minimiza ventana de acceso no autorizado
- ✅ Transparente para usuario con silent re-auth

### Especificación Técnica

**Worker Implementation (Node.js/pseudocódigo):**
```javascript
const cron = require('node-cron');

// Worker ejecuta cada 1 minuto
cron.schedule('* * * * *', async () => {
  console.log('[Worker] Procesando cambios críticos...');

  try {
    const cambios = await db.query(`
      SELECT * FROM cambios_criticos
      WHERE procesado = false
      ORDER BY detectado_at ASC
      LIMIT 100
    `);

    for (const cambio of cambios) {
      await procesarCambioCritico(cambio);
    }

    console.log(`[Worker] ${cambios.length} cambios procesados`);
  } catch (error) {
    console.error('[Worker] Error:', error);
    alertar('Worker invalidación falló', error);
  }
});

async function procesarCambioCritico(cambio) {
  const transaction = await db.beginTransaction();

  try {
    // 1. Invalidar sesiones
    const result = await transaction.query(`
      UPDATE sessions
      SET invalidated_at = NOW(),
          logout_type = $1
      WHERE user_id = $2
        AND invalidated_at IS NULL
        AND expires_at > NOW()
      RETURNING session_id
    `, [
      `PROACTIVO_${cambio.tipo_cambio}`,
      cambio.user_id
    ]);

    const sesiones_invalidadas = result.rowCount;

    // 2. Marcar cambio como procesado
    await transaction.query(`
      UPDATE cambios_criticos
      SET procesado = true,
          procesado_at = NOW(),
          sesiones_invalidadas = $1
      WHERE id = $2
    `, [sesiones_invalidadas, cambio.id]);

    // 3. Auditoría
    await transaction.query(`
      INSERT INTO audit_logs (tipo_evento, user_id, tenant_id, descripcion, datos_adicionales, ...)
      VALUES (...)
    `);

    await transaction.commit();

    console.log(`[Worker] Cambio ${cambio.id} procesado: ${sesiones_invalidadas} sesiones invalidadas`);

    // Métricas
    const tiempo_procesamiento = Date.now() - new Date(cambio.detectado_at).getTime();
    reportarMetrica('invalidacion_latencia_ms', tiempo_procesamiento);

  } catch (error) {
    await transaction.rollback();
    console.error(`[Worker] Error procesando cambio ${cambio.id}:`, error);

    // Registrar error para retry
    await db.query(`
      UPDATE cambios_criticos
      SET error_procesamiento = $1
      WHERE id = $2
    `, [error.message, cambio.id]);
  }
}
```

**Query Optimizado para Sesiones Activas:**
```sql
-- Índice crítico para performance
CREATE INDEX idx_sessions_user_active_invalidacion ON sessions(user_id, invalidated_at, expires_at)
WHERE invalidated_at IS NULL;

-- Query rápida
UPDATE sessions
SET invalidated_at = NOW(),
    logout_type = 'PROACTIVO_CAMBIO_ROLES'
WHERE user_id = 'user-uuid'
  AND invalidated_at IS NULL
  AND expires_at > NOW();

-- Con índice, esta query es O(log n) muy rápida
```

**Métricas Clave:**
- `invalidacion_latencia_ms`: Tiempo desde detección hasta invalidación
- `sesiones_invalidadas_por_minuto`: Throughput del worker
- `cambios_pendientes`: Cola de cambios sin procesar
- `errores_invalidacion`: Fallos en worker

**Objetivo SLA:**
- Latencia promedio: <60 segundos
- Latencia p95: <120 segundos
- Disponibilidad worker: >99.9%

### Referencias

- **HU-AD006A - Detección Cambios Críticos**: Genera cambios pendientes
- **HU-AD005A - Gestión Sesiones JWT**: Modelo sessions
- **HU-AD004B - Flujo SSO**: Silent re-authentication
- **Estándar Auditoría**: Tipos `INTEGRACION_AD_INVALIDACION_PROACTIVA_*`
- **Glosario**: Invalidación Proactiva, Silent Re-authentication, SLA Seguridad

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**Módulo:** Integración AD/SCIM - Proactive Invalidation
