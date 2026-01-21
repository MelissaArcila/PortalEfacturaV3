# HU-AD008B - Recovery y Resiliencia

## Información General

| Campo | Descripción |
|-------|-------------|
| **Release objetivo** | Release 2 |
| **Epic** | Integración con Active Directory mediante SCIM |
| **Estado** | BORRADOR |

---

## Contexto

Mecanismos de recuperación automática y resiliencia ante fallos en integración AD.

---

## Escenarios Básicos

| Nº | Criterio de aceptación (Título) | Resultado esperado |
|----|--------------------------------|---------------------|
| 1 | Retry automático de sincronización fallida | Si sync delta falla (BD timeout, red, etc.) → Worker reintenta automáticamente con backoff exponencial (1min, 2min, 4min). Máx 5 reintentos. Si persiste: alerta admin |
| 2 | Circuit breaker para IdP | Si >10 fallos SAML consecutivos del IdP → Circuit breaker abre, rechaza autenticaciones por 5 min (evita saturar IdP caído). Muestra mensaje "IdP temporalmente no disponible" |
| 3 | Degradación graceful de servicios | Si BD de auditoría falla → Sistema continúa operando (autenticación, SCIM) pero logs se pierden. Alerta CRITICAL. Permite operación con funcionalidad reducida |
| 4 | Reinicio automático de workers | Si worker invalidación falla/crash → Supervisor (PM2, Kubernetes) reinicia automáticamente. Máx 3 reinicios en 5 min, después alerta |
| 5 | Backup de configuraciones AD | Configuraciones de tenants (IdP URL, certificados, etc.) se respaldan diariamente a almacenamiento externo. Permite restauración ante pérdida de BD |
| 6 | Health check endpoints | Endpoint `/health/scim` y `/health/saml` devuelven status del sistema. Permite monitoreo externo (New Relic, Datadog) |

**Notas:**
- Diseño para alta disponibilidad (99.9% uptime)
- Fallos parciales no afectan todo el sistema
- Monitoreo proactivo detecta problemas antes de impactar usuarios

---

**Versión:** 1.0
**Módulo:** Integración AD/SCIM - Error Handling
