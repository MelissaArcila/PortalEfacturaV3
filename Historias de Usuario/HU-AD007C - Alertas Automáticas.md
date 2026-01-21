# HU-AD007C - Alertas Automáticas

## Información General

| Campo | Descripción |
|-------|-------------|
| **Release objetivo** | Release 2 |
| **Epic** | Integración con Active Directory mediante SCIM |
| **Estado** | BORRADOR |

---

## Contexto

Sistema de alertas automáticas para eventos críticos de integración AD que requieren atención inmediata del equipo técnico.

---

## Escenarios Básicos

| Nº | Criterio de aceptación (Título) | Resultado esperado |
|----|--------------------------------|---------------------|
| 1 | Alerta de autenticaciones fallidas masivas | Si >50 SAML_LOGIN_FALLIDO en 15 min para un tenant → Alerta CRITICAL email + Slack |
| 2 | Alerta de worker invalidación inactivo | Si worker no ejecuta >5 min → Alerta CRITICAL |
| 3 | Alerta de sincronización fallida | Si sync inicial/delta falla >3 veces consecutivas → Alerta ERROR |
| 4 | Alerta de certificado próximo a expirar | Si certificado X.509 expira en <30 días → Alerta WARNING diaria |
| 5 | Alerta de rate limiting excedido frecuente | Si tenant excede rate limit >10 veces en 1 hora → Alerta WARNING |

**Notas:**
- Alertas enviadas a canales configurables (email, Slack, PagerDuty)
- Evita fatiga con throttling (1 alerta del mismo tipo cada 1 hora máx)
- Dashboard de alertas activas para monitoreo

---

**Versión:** 1.0
**Módulo:** Integración AD/SCIM - Audit & Logs
