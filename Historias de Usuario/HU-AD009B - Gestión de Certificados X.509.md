# HU-AD009B - Gestión de Certificados X.509

## Información General

| Campo | Descripción |
|-------|-------------|
| **Release objetivo** | Release 2 |
| **Epic** | Integración con Active Directory mediante SCIM |
| **Estado** | BORRADOR |

---

## Contexto

Gestión del ciclo de vida de certificados X.509 del IdP usados para validar firmas SAML.

---

## Escenarios Básicos

| Nº | Criterio de aceptación (Título) | Resultado esperado |
|----|--------------------------------|---------------------|
| 1 | Validación de certificado al cargar | Admin carga certificado X.509 (PEM). Sistema valida: formato válido, no expirado, extrae: issuer, subject, validFrom, validTo, thumbprint. Rechaza si inválido |
| 2 | Alerta de expiración próxima | Si certificado expira en <30 días → Alerta WARNING diaria a admin. Email con instrucciones de renovación |
| 3 | Renovación de certificado | Admin carga nuevo certificado. Ambos válidos por período de transición (7 días). Permite migración sin downtime. Certificado antiguo se desactiva automáticamente después |
| 4 | Histórico de certificados | Sistema mantiene histórico de certificados usados (activos + expirados). Útil para auditorías |
| 5 | Validación de cadena de confianza | Sistema valida que certificado no es auto-firmado (producción). En desarrollo permite auto-firmados |
| 6 | Extracción automática de metadata | Sistema extrae automáticamente certificado desde metadata XML SAML del IdP (si cliente provee). Facilita configuración |

**Notas:**
- Certificados almacenados en BD cifrados (encryption at rest)
- Solo clave pública se almacena (Portal es SP, no necesita private key del IdP)
- Renovación sin downtime es crítica para producción

---

**Versión:** 1.0
**Módulo:** Integración AD/SCIM - Credentials Management
