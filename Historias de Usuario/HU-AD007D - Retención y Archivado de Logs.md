# HU-AD007D - Retención y Archivado de Logs

## Información General

| Campo | Descripción |
|-------|-------------|
| **Release objetivo** | Release 2 |
| **Epic** | Integración con Active Directory mediante SCIM |
| **Estado** | BORRADOR |

---

## Contexto

Política de retención de logs para cumplimiento regulatorio y optimización de almacenamiento.

---

## Escenarios Básicos

| Nº | Criterio de aceptación (Título) | Resultado esperado |
|----|--------------------------------|---------------------|
| 1 | Retención de auditoría funcional | Logs severidad INFO/WARNING: 90 días en BD activa. ERROR/CRITICAL: 1 año |
| 2 | Archivado a almacenamiento frío | Logs >90 días migran a S3/Azure Blob (comprimidos). Accesibles solo para auditorías externas |
| 3 | Retención logs técnicos SCIM | 30 días en sistema de logging (ELK/CloudWatch), luego eliminados |
| 4 | Job de limpieza automática | Job diario (3am) elimina logs expirados según políticas. Registra cantidad eliminada |
| 5 | Restauración de logs archivados | Admin puede solicitar restauración de logs archivados (proceso manual, tarda 24h) |

**Notas:**
- Cumple GDPR, SOX, ISO 27001
- Logs CRITICAL nunca se eliminan automáticamente (retención indefinida)
- Particionamiento de tabla por mes para performance

---

**Versión:** 1.0
**Módulo:** Integración AD/SCIM - Audit & Logs
