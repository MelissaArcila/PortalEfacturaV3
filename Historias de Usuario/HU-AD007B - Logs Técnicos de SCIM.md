# HU-AD007B - Logs Técnicos de SCIM

## Información General

| Campo | Descripción |
|-------|-------------|
| **Release objetivo** | Release 2 |
| **Epic** | Integración con Active Directory mediante SCIM |
| **Estado** | BORRADOR |
| **Propietario** | Por definir |
| **Arquitecto** | Por definir |

---

## Contexto

Logs técnicos detallados de peticiones SCIM para debugging, monitoreo de performance y troubleshooting. Separados de auditoría funcional para no saturar logs de compliance.

---

## Enunciado General de la Historia

| Roles | Característica / Funcionalidad | Razón / Resultado |
|-------|-------------------------------|-------------------|
| Como **Desarrollador** | Quiero logs detallados de peticiones SCIM | Para debugging cuando clientes reportan problemas de sincronización |
| Como **DevOps** | Quiero métricas de performance del endpoint SCIM | Para monitorear latencia y detectar degradación |

---

## Escenarios

| Nº | Criterio de aceptación (Título) | Contexto | Evento | Resultado / Comportamiento esperado | Desarrollo | QA | Product Owner |
|----|--------------------------------|----------|--------|-------------------------------------|------------|----|---------------|
| 1 | Log de petición SCIM | Endpoint SCIM recibe petición | Sistema registra | Log técnico incluye: timestamp, tenant_id, método HTTP, URL, status code HTTP, latencia_ms, ip_origen, user_agent, request_id (UUID). Nivel INFO. No incluye payloads completos (privacidad). Permite analizar patrones de uso y performance | ☐ | ☐ | ☐ |
| 2 | Log de error técnico SCIM | Operación SCIM falla (500, timeout, etc) | Sistema registra error | Log técnico nivel ERROR con: error_tipo, stack_trace, tenant_id, operación_intentada, request_id. Permite debugging de fallos internos | ☐ | ☐ | ☐ |
| 3 | Métricas de latencia | Peticiones SCIM completadas | Sistema mide tiempos | Registra métricas: latencia_p50, latencia_p95, latencia_p99 por operación (GET/POST/PUT/DELETE). Alertas si p95 >2 segundos | ☐ | ☐ | ☐ |
| 4 | Dashboard de logs SCIM | Desarrollador accede a logs | Vista de logs técnicos | Dashboard con filtros por tenant, método, status code, rango de fechas. Muestra últimos 1000 logs. Útil para troubleshooting | ☐ | ☐ | ☐ |

---

## Notas Adicionales

- Logs técnicos se almacenan en sistema de logging centralizado (ej: ELK, CloudWatch)
- Retención: 30 días (vs 90 días auditoría funcional)
- NO incluyen datos sensibles (passwords, tokens completos)

---

**Versión:** 1.0
**Última actualización:** 2026-01-20
**Módulo:** Integración AD/SCIM - Audit & Logs
