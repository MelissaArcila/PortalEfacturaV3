# HU-AD006A - Detección de Cambios Críticos

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
| **Arquitectura relacionada** | Portal Unificado - Seguridad, SCIM, Session Management |

---

## Contexto

Esta HU implementa el sistema de **detección de cambios críticos** en usuarios gestionados por AD que requieren **invalidación proactiva de sesiones** para mantener la seguridad.

Cuando ocurren ciertos cambios críticos en Active Directory (cambio de roles/grupos, desactivación de cuenta, eliminación), el Portal debe:
1. **Detectar** el cambio durante sincronización SCIM (HU-AD003B delta)
2. **Clasificar** si el cambio es crítico y requiere invalidación inmediata
3. **Marcar** las sesiones afectadas para invalidación
4. **Registrar** el evento para auditoría y tracking

El objetivo es que cambios de permisos en AD se reflejen en el Portal en **menos de 1 minuto** (no esperar expiración natural del JWT de 4 horas).

---

**Notas:**
- Detección ocurre durante sincronización delta SCIM (automática cada 15-30 min)
- No todos los cambios son críticos (cambio de nombre NO requiere invalidación)
- Sistema es reactivo a sincronización SCIM (no push instantáneo desde AD)
- Objetivo: <1 minuto desde cambio en AD hasta invalidación en Portal

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Portal Unificado** | Quiero detectar cambios críticos en usuarios durante sincronización SCIM | Para identificar cuándo debo invalidar sesiones por seguridad |
| Como **Portal Unificado** | Quiero clasificar cambios como críticos o no-críticos | Para invalidar sesiones solo cuando sea necesario, no por cualquier cambio |
| Como **Administrador de Seguridad** | Quiero que cambios de permisos en AD se reflejen inmediatamente | Para prevenir que usuarios conserven acceso no autorizado durante horas |
| Como **Portal Unificado** | Quiero registrar todos los cambios críticos detectados | Para auditoría y cumplimiento de políticas de seguridad |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Detección de cambio de roles (grupos) - PUT/PATCH | Sincronización delta recibe PUT o PATCH con cambio en `groups` | Sistema procesa actualización | Sistema: (1) Obtiene roles actuales del usuario desde BD (BEFORE), (2) Calcula nuevos roles desde `groups` recibido (validación catálogo HU-AD001B), (3) Compara roles BEFORE vs AFTER, (4) Si difieren: marca como cambio crítico `tipo=CAMBIO_ROLES`, almacena en tabla `cambios_criticos` con user_id, roles_anteriores, roles_nuevos, timestamp. Actualiza usuario en BD con nuevos roles | ☐ | ☐ | ☐ |
| 2 | Detección de desactivación de cuenta - PATCH active=false | Sincronización delta recibe PATCH con `active: false` | Sistema procesa | Sistema: (1) Detecta que usuario está activo en BD (`active=true`), (2) Nuevo valor es `active=false`, (3) Marca como cambio crítico `tipo=DESACTIVACION_CUENTA`, almacena en `cambios_criticos`, (4) Actualiza `users.active = false`. Usuario queda desactivado | ☐ | ☐ | ☐ |
| 3 | Detección de eliminación de usuario - DELETE | Sincronización delta recibe DELETE /Users/{id} | Sistema procesa | Sistema: (1) Usuario existe y activo, (2) Recibe DELETE, (3) Marca como cambio crítico `tipo=ELIMINACION_USUARIO`, almacena en `cambios_criticos`, (4) Realiza soft delete (`deleted_at = NOW()`). Usuario marcado como eliminado | ☐ | ☐ | ☐ |
| 4 | Cambio NO crítico - Actualización de nombre | PATCH modifica `name.givenName` o `name.familyName` | Sistema actualiza sin marcar crítico | Sistema detecta que solo cambió nombre (NO roles, NO active, NO delete). Actualiza usuario en BD pero NO crea registro en `cambios_criticos`. Sesiones NO se invalidan. Cambio no afecta permisos | ☐ | ☐ | ☐ |
| 5 | Cambio NO crítico - Actualización de email | PATCH modifica `emails` | Sistema actualiza sin marcar | Similar a escenario 4: actualiza email pero NO marca como crítico. Sesiones NO se invalidan | ☐ | ☐ | ☐ |
| 6 | Múltiples cambios críticos en una sincronización | Usuario modificado: cambio de roles Y desactivación | Sistema detecta ambos | Sistema crea UN registro en `cambios_criticos` con `tipo=MULTIPLE` que incluye: cambio_roles=true, desactivacion=true, cambios_detalle={JSON con cambios}. Evita duplicados para mismo usuario | ☐ | ☐ | ☐ |
| 7 | Tabla de cambios críticos pendientes | Sistema detecta cambio crítico | Registro creado | Sistema inserta en tabla `cambios_criticos`: id (UUID), user_id, tenant_id, tipo_cambio (CAMBIO_ROLES/DESACTIVACION/ELIMINACION/MULTIPLE), cambios_detalle (JSONB con before/after), detectado_at (NOW()), procesado (false), procesado_at (NULL). Registro queda pendiente para procesamiento por HU-AD006B | ☐ | ☐ | ☐ |
| 8 | Clasificación de cambio de roles - Adición de rol | Usuario tenía ["Contador"], ahora tiene ["Contador", "Administrador"] | Sistema clasifica severidad | Sistema detecta ADICIÓN de rol privilegiado. Marca severidad=HIGH (rol admin agregado requiere invalidación inmediata). Almacena en cambios_detalle: {accion: "ADICION", rol_agregado: "Administrador", severidad: "HIGH"} | ☐ | ☐ | ☐ |
| 9 | Clasificación de cambio de roles - Remoción de rol | Usuario tenía ["Admin", "Contador"], ahora tiene ["Contador"] | Sistema clasifica | Sistema detecta REMOCIÓN de rol privilegiado. Marca severidad=CRITICAL (perder admin es crítico, debe invalidar ya). Almacena: {accion: "REMOCION", rol_removido: "Administrador", severidad: "CRITICAL"} | ☐ | ☐ | ☐ |
| 10 | Worker de detección de cambios críticos | Job programado (cada 1 minuto) | Sistema busca cambios pendientes | Sistema ejecuta query: `SELECT * FROM cambios_criticos WHERE procesado = false ORDER BY detectado_at ASC LIMIT 100`. Para cada cambio: invoca HU-AD006B para invalidar sesiones. Permite procesamiento asíncrono y rápido (<1 min) después de detección | ☐ | ☐ | ☐ |

---

## Escenarios de Auditoría

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 11 | Estructura obligatoria de auditoría | Eventos de cambios críticos | Sistema registra | Registro contiene: ID Evento (UUID), Tipo Evento (INTEGRACION_AD_CAMBIO_CRITICO_*), Fecha/Hora (UTC ISO 8601), Usuario (user_id afectado), Cliente (tenant_id), IP Local (NULL), IP Pública (NULL - cambio desde SCIM), Resultado, Descripción, Severidad, Datos Adicionales | ☐ | ☐ | ☐ |
| 12 | Auditoría de cambio de roles detectado | Sistema detecta cambio en grupos | Cambio crítico marcado | Registro: Tipo "INTEGRACION_AD_CAMBIO_CRITICO_ROLES", Resultado "EXITOSO", Severidad "WARNING", Descripción "Cambio de roles detectado para usuario {userName}", Datos Adicionales: {"user_id": "UUID", "tenant_id": "UUID", "roles_anteriores": ["Contador"], "roles_nuevos": ["Contador", "Administrador"], "accion": "ADICION", "severidad": "HIGH", "cambio_id": "UUID"} | ☐ | ☐ | ☐ |
| 13 | Auditoría de desactivación detectada | Usuario desactivado en AD | Cambio crítico marcado | Registro: Tipo "INTEGRACION_AD_CAMBIO_CRITICO_DESACTIVACION", Resultado "EXITOSO", Severidad "CRITICAL", Descripción "Cuenta desactivada para usuario {userName}", Datos Adicionales: {"user_id": "UUID", "cambio_id": "UUID"} | ☐ | ☐ | ☐ |
| 14 | Auditoría de eliminación detectada | Usuario eliminado de AD | Cambio crítico marcado | Registro: Tipo "INTEGRACION_AD_CAMBIO_CRITICO_ELIMINACION", Resultado "EXITOSO", Severidad "CRITICAL", Descripción "Usuario {userName} eliminado de AD", Datos Adicionales: {"user_id": "UUID", "deleted_at": "2026-01-20T15:00:00Z", "cambio_id": "UUID"} | ☐ | ☐ | ☐ |
| 15 | Inmutabilidad y consulta | Cualquier evento | Sistema genera registro | Registros inmutables. Filtrable por fechas, usuario, tenant, tipo cambio, severidad. Exportable CSV/Excel | ☐ | ☐ | ☐ |

---

## Interacción con el Usuario y Prototipo

### Mockups / Wireframes

**Estado: N/A - Proceso backend automático sin interfaz directa**

**Dashboard de Cambios Críticos (para administradores):**

Opcionalmente, un dashboard puede mostrar:
- Cambios críticos detectados (últimas 24h)
- Estado de procesamiento (pendiente/procesado)
- Tiempo desde detección hasta invalidación
- Métricas de cambios por tipo

(Detalles en HU-AD007 - Audit & Logs)

---

## Riesgos

| Riesgo | Descripción | Impacto | Probabilidad | Mitigación | Estado |
|--------|-------------|---------|--------------|------------|--------|
| **Sincronización delta no detecta cambios a tiempo** | Si sync corre cada 30 min, cambio podría tardar hasta 30 min en detectarse | Alto | Media | Documentar a clientes que configuren sync cada 15 min, considerar webhook push futuro, worker de invalidación cada 1 min minimiza latencia adicional | Aceptado |
| **Falsos positivos marcan cambios no-críticos** | Lógica incorrecta podría invalidar sesiones innecesariamente | Medio | Baja | Testing exhaustivo de clasificación, logs detallados, ajustar lógica según feedback, permitir whitelist de cambios no-críticos | Mitigado |
| **Tabla cambios_criticos crece indefinidamente** | Sin cleanup, tabla podría saturarse | Bajo | Media | Cleanup automático de registros `procesado=true` y >30 días, índices optimizados | Mitigado |

---

## Notas Adicionales

### Fuera de Alcance

- Push instantáneo desde AD (webhooks) → Solo detección durante sync SCIM pull
- Cambios críticos en usuarios NO-AD → Solo usuarios con `gestionado_por_ad=true`
- Notificaciones al usuario cuando su sesión será invalidada → Usuario descubre al recibir 401
- Configuración de qué cambios son críticos por tenant → Lógica global para todos

### Dependencias

**Depende de:**
- **HU-AD003B - Sincronización Delta**: Detección ocurre durante delta sync
- **HU-AD002D - PUT/PATCH/DELETE**: Operaciones SCIM que disparan detección
- **HU-AD001B - Catálogo Roles**: Valida roles para clasificar cambios

**Es prerequisito para:**
- **HU-AD006B - Invalidación de Sesiones**: Procesa cambios críticos detectados

### Valor Independiente

- ✅ Identifica cambios que requieren acción de seguridad
- ✅ Provee base de datos de cambios para auditoría
- ✅ Puede implementarse sin invalidación (solo logging inicialmente)
- ✅ Crítico para cumplimiento de políticas de seguridad

### Especificación Técnica

**Tabla cambios_criticos:**
```sql
CREATE TABLE cambios_criticos (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id),
  tenant_id UUID REFERENCES tenants(id),
  tipo_cambio VARCHAR(50) NOT NULL, -- CAMBIO_ROLES, DESACTIVACION, ELIMINACION, MULTIPLE
  severidad VARCHAR(20), -- LOW, MEDIUM, HIGH, CRITICAL
  cambios_detalle JSONB NOT NULL, -- Detalles específicos del cambio
  detectado_at TIMESTAMP DEFAULT NOW(),
  procesado BOOLEAN DEFAULT false,
  procesado_at TIMESTAMP NULL,
  sesiones_invalidadas INT DEFAULT 0,
  error_procesamiento TEXT NULL
);

CREATE INDEX idx_cambios_criticos_pendientes ON cambios_criticos(procesado, detectado_at) WHERE procesado = false;
CREATE INDEX idx_cambios_criticos_user ON cambios_criticos(user_id, detectado_at DESC);
```

**Ejemplo de cambios_detalle (JSONB):**

Cambio de Roles:
```json
{
  "tipo": "CAMBIO_ROLES",
  "roles_anteriores": ["Contador"],
  "roles_nuevos": ["Contador", "Administrador del Portal"],
  "accion": "ADICION",
  "rol_agregado": "Administrador del Portal",
  "severidad": "HIGH"
}
```

Desactivación:
```json
{
  "tipo": "DESACTIVACION",
  "active_anterior": true,
  "active_nuevo": false
}
```

Eliminación:
```json
{
  "tipo": "ELIMINACION",
  "deleted_at": "2026-01-20T15:00:00Z"
}
```

**Clasificación de Severidad:**

| Cambio | Severidad | Justificación |
|--------|-----------|---------------|
| Adición de rol Administrador | HIGH | Nuevo acceso privilegiado |
| Remoción de rol Administrador | CRITICAL | Pérdida de privilegios, debe invalidar inmediatamente |
| Adición de rol regular | MEDIUM | Nuevos permisos pero no críticos |
| Remoción de rol regular | HIGH | Pérdida de permisos |
| Desactivación account | CRITICAL | Usuario no debe tener acceso |
| Eliminación usuario | CRITICAL | Usuario no debe existir |

**Lógica de Detección (pseudocódigo):**
```javascript
async function detectarCambiosCriticos(operacion, usuario_id, datos_nuevos) {
  const usuario_actual = await db.getUserById(usuario_id);

  let cambio_critico = null;

  if (operacion === 'PUT' || operacion === 'PATCH') {
    // Detectar cambio de roles
    if (datos_nuevos.groups !== undefined) {
      const roles_anteriores = usuario_actual.roles;
      const roles_nuevos = mapearGruposARoles(datos_nuevos.groups);

      if (!arraysIguales(roles_anteriores, roles_nuevos)) {
        cambio_critico = {
          tipo_cambio: 'CAMBIO_ROLES',
          severidad: clasificarSeveridadRoles(roles_anteriores, roles_nuevos),
          cambios_detalle: {
            roles_anteriores,
            roles_nuevos,
            accion: determinarAccion(roles_anteriores, roles_nuevos)
          }
        };
      }
    }

    // Detectar desactivación
    if (datos_nuevos.active === false && usuario_actual.active === true) {
      cambio_critico = {
        tipo_cambio: 'DESACTIVACION',
        severidad: 'CRITICAL',
        cambios_detalle: {active_anterior: true, active_nuevo: false}
      };
    }
  } else if (operacion === 'DELETE') {
    cambio_critico = {
      tipo_cambio: 'ELIMINACION',
      severidad: 'CRITICAL',
      cambios_detalle: {deleted_at: new Date().toISOString()}
    };
  }

  if (cambio_critico) {
    await db.insertCambioCritico({
      user_id: usuario_id,
      tenant_id: usuario_actual.tenant_id,
      ...cambio_critico,
      procesado: false
    });
  }
}
```

### Referencias

- **HU-AD003B - Sincronización Delta**: Contexto de detección
- **HU-AD002D - PUT/PATCH/DELETE**: Operaciones monitoreadas
- **Estándar Auditoría**: Tipos `INTEGRACION_AD_CAMBIO_CRITICO_*`
- **Glosario**: Cambio Crítico, Severidad, Invalidación Proactiva

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**Módulo:** Integración AD/SCIM - Proactive Invalidation
